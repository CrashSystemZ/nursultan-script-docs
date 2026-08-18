# The assets folder

`assets` is `client.assets()` and reads files under `%APPDATA%\Nursultan\scripts\assets`. `base64(...)` is the alternative: it turns string literals in the `.kts` itself into bytes a font or an image can be built from. Everything on this page is API 2.

```kotlin
val greeting = assets.text("messages/hello.txt") ?: "no file"

val FONT = base64(
    "AAEAAAAOAIAAAwBgT1MvMg…",
    "…AAAAAAAAAAAAAAAAAAAA=",
)
font("myfont", FONT)

on<Render2DEvent> { e ->
    e.render().text(greeting, 10f, 10f, 12f, Colors.WHITE, "myfont")
}
```

## Reading a file

| Method | Type | Description |
|---|---|---|
| `assets.exists(path)` | `boolean` | true when the path resolves inside `scripts/assets` and is a regular file (API 2) |
| `assets.bytes(path)` | `byte[]?` | whole file, null when rejected, missing, over 8 MiB or unreadable (API 2) |
| `assets.text(path)` | `String?` | the same bytes decoded as UTF-8, null under the same conditions (API 2) |
| `assets.list()` | `List<String>` | sorted file names directly in the assets root, capped at 1024 (API 2) |
| `assets.list(directory)` | `List<String>` | same for one subfolder, empty when it does not resolve or is not a directory; null, blank or `"."` means the root (API 2) |

Paths are relative to `scripts/assets`, forward or backslash separated. Nothing here throws: a rejected, missing or oversized file gives `null` or an empty list plus one warning in the script console.
`list` returns file names only — no paths, no directories, no recursion.

## What is rejected

| Rejected | What you get |
|---|---|
| `..` in any path segment | `null` / empty list |
| absolute path, drive letter or UNC path | `null` / empty list |
| empty, blank or malformed path | `null` / empty list |
| a path whose real target leaves `scripts/assets` | `null` / empty list |
| a file larger than 8 MiB | `null` from `bytes` and `text` |
| more than 1024 files in one folder | the first 1024 names, the rest skipped |

Each case warns once per path in the script console. There is no way to open a file by absolute path and nothing in the API writes files — for a script's own data see [Saving data](../settings/storage.md).

## Assets inside the script

| Method | Type | Description |
|---|---|---|
| `base64(vararg parts)` | `ByteArray` | joins the parts and MIME-decodes them, empty vararg gives an empty array (API 2) (throws `ScriptApiException` when the payload is not valid base64) |
| `base64Encode(bytes)` | `String` | standard base64 on one line, the other direction (API 2) |
| `font(name, ttf)` | `void` | registers a TTF family from bytes, ignored when the name is taken, the array is empty or over 8 MiB (API 2) |
| `image(name, png)` | `Texture?` | PNG from bytes cached under `name`, null for a blank name, empty bytes or over 8 MiB (API 2) |

The joined text may be a `data:` URL: everything up to and including `;base64,` is dropped, and a `data:` prefix without that marker throws `ScriptApiException`. Whitespace and line breaks inside a part are ignored.
A single string literal in a class file cannot exceed 64 KB, and Kotlin folds `"a" + "b"` back into one — that is why `base64` takes a vararg. `name` in `image(name, png)` is a per-script cache key, not a path; both byte overloads are also listed on [2D render](../ui/render-2d.md).

## Files from older scripts

`font(name, file)`, `image(file)`, `client.fonts().register(name, file)`, `client.textures().image(file)` and `Render.image(file, …)` fall back to the scripts root when the file is not in `scripts/assets`, and warn once in the console.
`exists`, `bytes`, `text` and `list` have no fallback — they only ever read `scripts/assets`. `texture(identifier)` takes a Minecraft resource id, not a file, and touches neither folder.
