# Saving data

`storage` is the script's default config; `config("name")` opens another one. Every value is a string, and the file lives at `%APPDATA%/Nursultan/configs/<uid>/scripts/<scriptId>/<name>.dat`.

```kotlin
var runs = 0

onEnable {
    runs = storage.getInt("runs", 0) + 1
}

onDisable {
    storage.put("runs", runs)
    storage.save()
}
```

## Opening one

| Method | Type | Description |
|---|---|---|
| `config(name)` | `Config` | opens this script's named config, cached per name |
| `configs.open(name)` | `Config` | same as `config(name)` |
| `configs.names()` | `List<String>` | this script's config files on disk, alphabetical |
| `configs.exists(name)` | `boolean` | config file exists on disk |
| `configs.delete(name)` | `void` | deletes the file and empties the open instance |

A name is trimmed, lowercased and must match `^[a-z0-9][a-z0-9_-]{0,31}$`, otherwise `ScriptException`.
`storage` is the config named `storage`.

## Reading

| Method | Type | Description |
|---|---|---|
| `storage.name()` | `String` | the config's lowercased name |
| `storage.has(key)` | `boolean` | key present in memory |
| `storage.get(key, fallback)` | `String` | stored value, fallback when absent |
| `storage.getInt(key, fallback)` | `int` | parsed as int, fallback when absent or unparsable |
| `storage.getLong(key, fallback)` | `long` | parsed as long, fallback when absent or unparsable |
| `storage.getDouble(key, fallback)` | `double` | parsed as double, fallback when absent or unparsable |
| `storage.getBoolean(key, fallback)` | `boolean` | true only for `true` ignoring case, fallback when absent |
| `storage.getList(key)` | `List<String>` | stored list, empty when absent (API 2) |
| `storage.getList(key, fallback)` | `List<String>` | stored list, copy of fallback when absent (API 2) |
| `storage.getIntList(key)` | `List<Integer>` | list parsed as ints, unparsable entries dropped (API 2) |
| `storage.getDoubleList(key)` | `List<Double>` | list parsed as doubles, unparsable entries dropped (API 2) |

A blank key throws `ScriptException` on every method that takes one.
A value written with `put` reads back through `getList` as a single-element list.

## Writing

| Method | Type | Description |
|---|---|---|
| `storage.put(key, value: String)` | `void` | stores the string (throws over 65536 chars) |
| `storage.put(key, value: Int)` | `void` | stores the decimal text |
| `storage.put(key, value: Long)` | `void` | stores the decimal text |
| `storage.put(key, value: Double)` | `void` | stores the decimal text |
| `storage.put(key, value: Boolean)` | `void` | stores `true` or `false` |
| `storage.putList(key, values)` | `void` | stores a `List<String>` (API 2) |
| `storage.putIntList(key, values)` | `void` | stores a `List<Integer>` as text (API 2) |
| `storage.putDoubleList(key, values)` | `void` | stores a `List<Double>` as text (API 2) |
| `storage.remove(key)` | `void` | drops the key |
| `storage.clear()` | `void` | drops every key in memory |

A config holds 4096 keys; adding one more throws `ScriptException`.
`put` throws `NullPointerException` on a null value, the list writers throw `ScriptException` on a null element.

## The file

| Method | Type | Description |
|---|---|---|
| `storage.keys()` | `Set<String>` | immutable copy of the current key set |
| `storage.dirty()` | `boolean` | values differ from the last save or load |
| `storage.save()` | `void` | writes the file and clears `dirty()` |
| `storage.load()` | `void` | re-reads the file, dropping unsaved changes |
| `storage.delete()` | `void` | deletes the file and empties the values in memory |

Every open config with `dirty()` is saved when the script is unloaded or reloaded.
The file is Zstd-compressed obfuscated MessagePack; io failures are logged, never thrown.
