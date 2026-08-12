# The assets folder

Fonts, images and any other file a script reads live in one place: `%APPDATA%\Nursultan\scripts\assets`. The client creates the folder next to the scripts themselves, and every path the API takes is relative to it.

```kotlin
font("myfont", "fonts/my-font.ttf")          // scripts/assets/fonts/my-font.ttf
val logo = image("images/logo.png")          // scripts/assets/images/logo.png

val greeting = assets.text("messages.txt")
val palette = assets.bytes("palette.bin")

if (assets.exists("images/logo.png")) {
    // ...
}

for (name in assets.list("images")) {
    // "logo.png", "mark.png", …
}
```

Subfolders are yours to invent — the client only ever creates `assets` itself.

## Reading a file

| Call | What you get |
|---|---|
| `assets.exists("data.json")` | `true` if the file is there |
| `assets.text("data.json")` | the file as a `String` (UTF-8), or `null` |
| `assets.bytes("logo.png")` | the file as a `ByteArray`, or `null` |
| `assets.list()` | file names in `assets` |
| `assets.list("images")` | file names in `assets/images` |

Nothing throws. A missing file, a folder that is not there, a path the client refuses — you get `null` or an empty list, plus one line in the script console naming the file.

`list()` gives you names, not paths, and only of files: no subfolders, nothing from deeper down. Hand a name back together with the folder you asked about:

```kotlin
val icons = assets.list("images").filter { it.endsWith(".png") }
val first = image("images/" + icons.first())
```

Reading is not free — the file goes off the disk on every call. Read it once at the top of the script or on enable, keep it in a variable, and stay away from reading inside a tick or render handler.

## What the client will not read

The path is relative to `assets` and it stays inside `assets`:

* `..` in any shape, an absolute path, a drive letter, a network path — refused;
* a link pointing out of `assets` — refused, even though the path itself looks innocent;
* a file over 8 MiB — refused by `text()` and `bytes()`;
* a folder with more than 1024 files — `list()` returns the first 1024 names and says so.

The rest of the disk does not exist for a script: there is no way to open a file by full path, and nothing in the API writes files. For your own data use [configs](../settings/storage.md).

## Assets inside the script

A script that ships as a single `.kts` can carry its files with it, encoded as base64 — the user downloads nothing extra:

```kotlin
val LOGO = base64(
    "0YXRg9C5INGF0YPQuSDRhdGD0Lkg0YXRg9C5INGF0YPQuSDRhdGD0Lk=",
    "0LbQvtC/0LAg0LbQvtC/0LAg0LbQvtC/0LAg0LbQvtC/0LA=",
)

val logo = image("logo", LOGO)               // decoded once, then cached
font("myfont", base64(FONT_PART1, FONT_PART2))

on<Render2DEvent> { e ->
    e.render().texture(logo ?: return@on, 10f, 10f, 59f, 105f)
}
```

The text comes from any encoder — [base64encode.org](https://www.base64encode.org/), for instance: pick the file, copy the result.

| Call | What you get |
|---|---|
| `base64(vararg parts)` | `ByteArray`; if the text is not base64 the script fails to load, with the file and line |
| `base64Encode(bytes)` | `String` — the other direction, handy for putting your own data into a [config](../settings/storage.md) |
| `image(name, png)` | `Texture?`, decoded once and cached under `name` |
| `font(name, ttf)` | a font family registered from the bytes, same as `font(name, file)` |

**Split a long blob into several strings.** A single string literal in a compiled class cannot exceed 64 KB, and Kotlin folds `"a" + "b"` back into one literal — so a font or a large PNG has to arrive as separate arguments, and `base64(part1, part2, part3)` joins them at runtime. Inside a part, whitespace and line breaks are ignored and a `data:image/png;base64,` prefix is stripped, so you can paste whatever the encoder gave you.

The `name` in `image(name, png)` is a cache key, not a path: the same name gives back the same texture without decoding again, names are private to your script, and the texture is freed when the script unloads. Decode at the top level of the script or in `onEnable` — never in a render or tick handler.

The 8 MiB per-asset cap applies here too. And remember that everything you embed lives in the source: a 200 KB font is 270 KB of text in the `.kts`.

## Files from older scripts

Before `assets` existed, fonts and images sat in the scripts folder itself. That still works: if the file is not in `assets`, the client looks next to the script and asks you once, in the console, to move it. New scripts should ship their files in `assets` — `exists()`, `text()`, `bytes()` and `list()` never look anywhere else.

## Worth remembering

* The client creates `assets` on start, empty; the subfolders inside it are up to you.
* Somebody else has to find these files: if your script ships more than a file or two, put them in a subfolder named after the script.
* `null` from `text()` or `bytes()` is a normal answer, not a crash — handle it and tell the user what is missing.
