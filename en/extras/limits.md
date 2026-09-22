# Sandbox and limits

A script runs against a whitelist: the Kotlin standard library, `java.time`, `java.math`, `kotlin.random.Random` and the whole `nursultan.*` API. Every handler invocation carries a 250 ms budget.

```kotlin
on<ClientTickEvent> {
    try {
        if (world.block(player.position()).liquid()) control.jump()
    } catch (error: ScriptStateException) {
        log.warn("no world: ${error.message}")
    }
}
```

## What you get

| Package | Description |
|---|---|
| `nursultan.**` | the whole script API |
| `kotlin.*` | every class directly in the `kotlin` package, subpackages excluded |
| `kotlin.collections.**` | lists, maps, sets and their operators |
| `kotlin.sequences.**` | lazy sequences |
| `kotlin.ranges.**` | ranges and progressions |
| `kotlin.text.**` | strings, regex, `StringBuilder` |
| `kotlin.comparisons.**` | comparator builders |
| `kotlin.math.**` | math functions and constants |
| `kotlin.random.**` | `Random` |
| `kotlin.enums.**` | `entries` on enums |
| `kotlin.annotation.**` | annotation targets and retention |
| `kotlin.jvm.internal.**`, `kotlin.jvm.functions.**` | intrinsics and lambda types the compiler emits |
| `java.util.stream.**` | streams and collectors |
| `java.util.function.**` | functional interfaces |
| `java.time.**` | instants, durations, dates |
| `java.math.**` | `BigInteger`, `BigDecimal` |
| `java.lang` (single classes) | `String`, `StringBuilder`, `Math`, number wrappers, common exceptions |
| `java.util` (single classes) | collections, `Optional`, `UUID`, `Random`, `Locale`, `BitSet`, `Objects` |
| `java.util.concurrent.ThreadLocalRandom` | the only class allowed out of `java.util.concurrent` |
| `kotlin.reflect.KProperty*` | as a type only, for `by` delegates |

`nursultan.**` and `kotlin.random.Random` are default imports; the `kotlin.*` rows are Kotlin's own defaults.

## What is blocked

| Blocked | Description |
|---|---|
| `java.io`, `java.nio` | no file access; only [assets](assets.md) and [configs](../settings/storage.md) touch the disk |
| `java.net` | no sockets, no http |
| `java.lang.Thread`, `java.util.concurrent` | no threads, pools or timers, `ThreadLocalRandom` aside |
| `java.lang.Class`, `java.lang.reflect`, `kotlin.reflect` | no reflection; only the `KProperty` types pass |
| `java.lang.ClassLoader`, `java.lang.System` | no class loading, no process or environment access |
| `net.minecraft.**`, `fun.nursultan.**` | no direct game or client internals |
| `nursultan.internal.BudgetControl` | the one denied class inside `nursultan` |
| native methods | a `native` method in a script class is rejected |
| `invokedynamic` | only `LambdaMetafactory` and `StringConcatFactory` bootstraps pass |

The check runs over the compiled classes before the script executes: a violation fails the load with `script class <name> references blocked symbol: <symbol>` and nothing runs.

## Budgets

| What | Limit |
|---|---|
| one handler, command or callback invocation | 250 ms |
| the top level of the file at load | 5 s |
| throws in a row before the script switches off | 5 |
| error reports per script | one per 3000 ms, the rest counted as `(+N suppressed)` |

Overrunning throws `ScriptTimeoutError`, reports it and switches the script off; the deadline is checked every 512th instrumented jump.
One successful invocation resets the throw counter.

## Exceptions

| Exception | Extends | Thrown when |
|---|---|---|
| `ScriptException` | `RuntimeException` | bad argument, unknown module/setting/entry reference, invalid id or url |
| `ScriptStateException` | `ScriptException` | no world or player, entity left the world, script already unloaded |
| `ScriptThreadException` | `ScriptException` | an API call that needs the client thread ran off it |
| `ScriptApiException` | `ScriptException` | `requireApi(n)` with `n` above `ApiVersion.CURRENT` |
| `ScriptTimeoutError` | `Error` | the handler budget ran out |

Each takes a `String message`; `ScriptException` also has a `(message, cause)` constructor.
`ScriptTimeoutError` is an `Error`, not a `ScriptException` — catching it changes nothing, the script is reported and switched off anyway.

## Resource limits

| Resource | Limit |
|---|---|
| keys per config | 4096, `ScriptException` on the next new key |
| chars per config value | 65536, `ScriptException` above it |
| asset file | 8 MiB, `bytes()` and `text()` return null above it |
| entries per `assets.list()` | 1024, extra names dropped with one console warning |
| embedded font or PNG | 8 MiB, ignored with one console warning above it |
| live gpu resources per script | 64 meshes, pipelines and render types together, `IllegalStateException` |
| attributes per vertex format | 16, `IllegalArgumentException` |
| vertices per mesh | 1 048 576, `IllegalStateException` |
| indices per mesh | 4 194 304, `IllegalStateException` |

Mesh buffers are freed while the script is switched off; the resources still count against the 64 until the script unloads.
