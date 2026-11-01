# Singleton Pattern

> **Interview weight: high** — not because you'll implement one, but because *"when would you use
> a Singleton?"* is a trap question. The strong answer is usually **"I wouldn't write one; I'd
> register a singleton lifetime in the DI container"** — and then explain why.

The **Singleton** pattern ensures a class has **exactly one instance** and provides a **global
point of access** to it.

Note that those are **two separate promises**, and the second is the problem. Guaranteeing one
instance is often legitimate. Providing global access is what makes Singleton the most criticised
pattern in the GoF catalogue.

---

## 1. Classic implementations (and why most are wrong)

### ❌ Not thread-safe

```csharp
public sealed class Config
{
    private static Config _instance;
    private Config() { }

    public static Config Instance => _instance ??= new Config();   // ⚠️ race condition
}
```

Two threads can both see `_instance == null` and both construct. Works in every single-threaded
test and fails in production.

### ⚠️ Double-checked locking — correct but dated

```csharp
public sealed class Config
{
    private static volatile Config _instance;      // volatile is REQUIRED here
    private static readonly object _lock = new();
    private Config() { }

    public static Config Instance
    {
        get
        {
            if (_instance is null)                 // avoid locking on the hot path
            {
                lock (_lock)
                {
                    if (_instance is null)         // re-check inside the lock
                        _instance = new Config();
                }
            }
            return _instance;
        }
    }
}
```

Correct, but verbose and easy to get subtly wrong. **Omitting `volatile` is a real bug** — without
it the memory model permits a thread to observe a non-null reference to a partially constructed
object.

### ✅ Static initialiser — simplest correct version

```csharp
public sealed class Config
{
    // The CLR guarantees type initialisers run once, thread-safely.
    private static readonly Config _instance = new();
    static Config() { }        // forces laziness; suppresses beforefieldinit
    private Config() { }

    public static Config Instance => _instance;
}
```

The explicit static constructor matters: without it the compiler marks the type `beforefieldinit`
and the runtime may initialise it **earlier than first access**. Usually harmless, occasionally not.

### ✅ `Lazy<T>` — the idiomatic .NET version

```csharp
public sealed class Config
{
    private static readonly Lazy<Config> _instance = new(() => new Config());
    private Config() { }

    public static Config Instance => _instance.Value;
}
```

`Lazy<T>` is thread-safe by default (`LazyThreadSafetyMode.ExecutionAndPublication`) and expresses
intent directly. **If you must hand-roll a singleton, write this one.**

---

## 2. Why modern .NET rarely writes any of the above

**The DI container already does this**, and does it better:

```csharp
builder.Services.AddSingleton<IConfigProvider, ConfigProvider>();
```

| | Hand-rolled Singleton | DI singleton lifetime |
|---|---|---|
| One instance guaranteed | ✅ | ✅ |
| Thread-safe construction | Your job | ✅ container's job |
| **Testable / swappable** | ❌ callers hard-code `Config.Instance` | ✅ inject `IConfigProvider`, substitute a fake |
| **Dependencies visible** | ❌ hidden — nothing in the signature says you use it | ✅ declared in the constructor |
| Lifetime control | Process-wide, forever | Scoped to the container; per-tenant/per-host possible |
| Disposal | Manual and awkward | ✅ container disposes it |

**This is the answer that scores.** The Singleton *pattern* couples every caller to a concrete type
via a static accessor. The DI *lifetime* gives you the same single instance while callers depend on
an abstraction they declare — so you keep substitutability and honest constructor signatures.

> **Interview line:** *"Singleton conflates two things: 'one instance' and 'globally reachable'.
> I want the first and almost never the second. A DI container gives me single-instance semantics
> while dependencies stay explicit and swappable — so I register `AddSingleton` rather than writing
> the pattern."*

---

## 3. Why Singleton is widely considered an anti-pattern

| Problem | Detail |
|---|---|
| **Hidden dependency** | `Config.Instance` inside a method means the class's real dependencies aren't in its constructor. You can't tell what a class needs by reading its signature. |
| **Untestable** | You cannot substitute it. Tests share mutable state and become order-dependent — the classic "passes alone, fails in the suite". |
| **Global mutable state** | If it holds mutable state, it is shared mutable state across the whole process, with all the concurrency hazards that implies. |
| **Violates SRP** | The class manages both its own responsibility *and* its lifecycle. |
| **Violates DIP** | Callers depend on a concrete type, not an abstraction. |
| **Hard to evolve** | "One per process" becomes wrong the moment you need one per tenant, per request, or per test. |

### The .NET-specific trap: the **captive dependency**

```csharp
builder.Services.AddSingleton<OrderProcessor>();   // singleton
builder.Services.AddScoped<AppDbContext>();        // scoped (per request)

public class OrderProcessor(AppDbContext db) { }   // ⚠️ singleton capturing a scoped service
```

The singleton is constructed **once** and holds that **first request's** `DbContext` **forever** —
so you get a disposed or cross-request context, `DbContext` used concurrently from multiple threads
(it is not thread-safe), and connection leaks. Symptoms appear under load, not in dev.

**Rule:** a service may only depend on services with a lifetime **equal to or longer than its own.**
Singleton → singleton only. To use a scoped service from a singleton, inject
`IServiceScopeFactory` and create a scope per unit of work:

```csharp
public class OrderProcessor(IServiceScopeFactory scopeFactory)
{
    public async Task ProcessAsync(int id, CancellationToken ct)
    {
        using var scope = scopeFactory.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        // ... use db within this scope only
    }
}
```

ASP.NET Core's default container **validates scopes in Development** and will throw on this. Do not
disable that check — it is catching a real bug.

---

## 4. When a singleton *is* legitimate

Single-instance is correct when the object is **stateless or immutable**, or when it wraps a
resource that is genuinely expensive and designed to be shared:

- `HttpClient` / `IHttpClientFactory` handlers — socket-exhaustion and DNS-staleness reasons
- Loggers, `IMemoryCache`, `IConfiguration`
- Connection multiplexers (e.g. `StackExchange.Redis` `ConnectionMultiplexer` — explicitly designed
  to be shared)
- Caches, object pools, `JsonSerializerOptions` instances
- Compiled expensive artefacts: regexes, compiled queries, mapping configuration

**In every one of these cases, register it with `AddSingleton` — don't write the pattern.**

**Thread safety is on you.** A DI singleton is shared across all concurrent requests, so any
mutable state inside it must be thread-safe (`ConcurrentDictionary`, `Interlocked`, immutability).
This is the most common source of heisenbugs in ASP.NET Core apps.

---

## 5. Real-world .NET example

```csharp
// ConnectionMultiplexer is expensive and designed to be shared — a genuine singleton case.
builder.Services.AddSingleton<IConnectionMultiplexer>(_ =>
    ConnectionMultiplexer.Connect(builder.Configuration.GetConnectionString("Redis")!));

// Consumers depend on the abstraction and declare it honestly.
public class ProductCache(IConnectionMultiplexer redis)
{
    private readonly IDatabase _db = redis.GetDatabase();
    public Task<RedisValue> GetAsync(string key) => _db.StringGetAsync(key);
}
```

Single instance, explicit dependency, substitutable in tests, disposed by the container. All the
benefit of Singleton, none of the coupling.

---

## 6. Interview questions

### Q1. Implement a thread-safe Singleton.
**Evaluating:** whether you know the memory-model details — then whether you question the premise.
**Strong answer:** show `Lazy<T>` (or the static-readonly form with an explicit static
constructor), mention that double-checked locking requires `volatile` and why, **then** add: "In an
ASP.NET Core codebase I'd register `AddSingleton` instead, so the instance is shared but callers
still depend on an interface."

### Q2. Why is Singleton considered an anti-pattern?
**Strong answer:** It hides dependencies (nothing in the constructor reveals them), prevents
substitution in tests, introduces global mutable state, and violates DIP by coupling callers to a
concrete type. The single-instance *guarantee* is usually fine; the *global static accessor* is the
problem.

### Q3. A singleton service depends on `DbContext`. What happens?
**Strong answer:** A **captive dependency**. The singleton is built once and captures the first
scope's `DbContext` permanently — leading to a disposed context, concurrent use of a non-thread-safe
type, and leaked connections. Fix: inject `IServiceScopeFactory` and create a scope per operation.
ASP.NET Core's scope validation catches this in Development.

### Q4. Singleton vs static class?
**Strong answer:** A static class can't implement an interface, be injected, be substituted in
tests, or carry per-container state — so it's even less flexible. A singleton *instance* behind an
interface, registered in DI, gets you shared state with none of those limitations. Static classes
are fine for genuinely stateless pure helpers (`Math`-style); anything with a dependency or state
should be an injected instance.

---

## Flashcards

**Q:** What two things does Singleton guarantee, and which is the problem?
**A:** One instance (usually fine) and a global access point (the problem — hidden dependencies, no substitution).

**Q:** Idiomatic thread-safe singleton in C#?
**A:** `Lazy<T>` — thread-safe by default. Or `static readonly` with an explicit static constructor.

**Q:** Why does double-checked locking need `volatile`?
**A:** Without it, a thread may observe a non-null reference to a partially constructed object.

**Q:** Preferred modern .NET approach?
**A:** `services.AddSingleton<IFoo, Foo>()` — same single instance, but dependencies stay explicit and substitutable.

**Q:** What is a captive dependency?
**A:** A longer-lived service capturing a shorter-lived one (singleton holding a scoped `DbContext`), freezing it past its intended lifetime.

**Q:** DI lifetime rule?
**A:** A service may only depend on lifetimes **equal to or longer than its own**.

**Q:** Legitimate singleton candidates?
**A:** Stateless/immutable or expensive shared resources — `ConnectionMultiplexer`, `IMemoryCache`, loggers, `JsonSerializerOptions`, object pools.

---

*Related: [Factory Method](1-factory-method.md) ·
[SOLID](../../05-design-principles/SOLID/SOLID.md).*
