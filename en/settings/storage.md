# Saving data

Setting values are saved for you — you never write those anywhere. Configs are for everything else: counters, lists, homes, stats.

## Quick start

`storage` is the default config every script gets:

```kotlin
val runs = storage.getInt("runs", 0) + 1
storage.put("runs", runs)
storage.save()

chat.print("run number $runs")
```

Values live in memory and `save()` writes them to disk. Forgetting to save is not fatal: anything unsaved is flushed when the script unloads.

## Several configs

If you have a lot of unrelated data, split it into files:

```kotlin
val stats = config("stats")
val homes = config("homes")

stats.put("kills", stats.getInt("kills", 0) + 1)
stats.save()
```

A config name is 1 to 32 characters of `a-z`, `0-9`, `_` and `-`.

```kotlin
configs.names()            // which configs already exist on disk
configs.exists("stats")
configs.delete("stats")
```

## What a config can do

```kotlin
storage.get("name", "default")
storage.getInt("count", 0)
storage.getLong("time", 0L)
storage.getDouble("ratio", 1.0)
storage.getBoolean("on", false)

storage.put("name", "text")
storage.put("count", 5)
storage.put("on", true)

storage.has("count")
storage.remove("count")
storage.clear()
storage.keys()             // every key
storage.dirty()            // are there unsaved changes

storage.save()
storage.load()             // re-read from disk, dropping unsaved changes
storage.delete()           // delete the file
```

The second argument of `get*` is what to return when the key is missing. If the key holds junk that will not parse as a number, you get it back too.

## Storing more than strings

A config stores string-to-string pairs. Anything else you lay out across keys yourself:

```kotlin
// a list
storage.put("homes", homes.joinToString(","))
val homes = storage.get("homes", "").split(",").filter { it.isNotEmpty() }

// an object
storage.put("home.base.x", x)
storage.put("home.base.y", y)
storage.put("home.base.z", z)

// find every home
val names = storage.keys()
    .filter { it.startsWith("home.") && it.endsWith(".x") }
    .map { it.removePrefix("home.").removeSuffix(".x") }
```

## Where it lives

Files are stored per script, compressed and encrypted, in the client's config folder. There is nothing to edit there by hand — the format is internal.

Limits: up to 4096 keys per config and up to 64 KiB per value. Hit one and you get a clear error.

## When to save

`save()` writes the file right away, so do not call it every tick. Sensible places:

* in a command handler, after the player changed something;
* in `onDisable` and `onUnload`;
* every few seconds via [`everyTicks`](../extras/tasks.md) if data keeps piling up.
