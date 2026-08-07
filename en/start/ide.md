# The project in your IDE

You can write scripts in Notepad, but an IDE gives you autocompletion across the whole API, type checking and go-to-definition. It is a one-time setup.

## What to download

The script development folder is on the site. It already contains everything:

| Path | What it is |
|---|---|
| `*.kts` | your scripts — `example.kts` is there to start from |
| `.sdk/nursultan-script-api-vN.jar` | the API itself, and the script template that tells the IDE what a `.kts` in this folder is |
| `.sdk/nursultan-script-packets-<version>.jar` | Minecraft packet classes, only needed if your script reads or sends them |
| `build.gradle.kts`, `settings.gradle.kts` | wire those jars in |
| `README.md` | the same setup steps as this page, offline |

`.sdk/` is generated — do not edit anything inside it. When you update the SDK, replace the whole folder.

## Open it in IntelliJ IDEA

`File → Open`, pick the folder, "Open as Project", trust it. Gradle syncs once and downloads only the Kotlin plugin.

## The required step: the script definition

IntelliJ does not discover custom script definitions on its own, so you have to point it at ours once per project:

> **Settings → Languages & Frameworks → Kotlin → Kotlin Scripting**
> → under *Custom Definitions* press **Search Definitions**
> → **Nursultan Script** (`.kts`) shows up in the list
> → keep it enabled and **above** *Default Kotlin Script* → **Apply**

Without this every `.kts` is red. Resolution is computed per file, so open a script afterwards to let the IDE work it out. If references still do not resolve, `File → Invalidate Caches → Invalidate and Restart`.

## Moving it into the game

A finished script is a single `.kts`. Copy it into the client's scripts folder (`%APPDATA%\Nursultan\scripts`) and switch it on in the Scripts tab. Nothing else needs to travel — not `.sdk`, not the build files.

Saving a file that is already loaded reloads the script live, so you never restart the game to test an edit.

## Developing straight in the game folder

If you would rather not copy anything, you can work directly in the client's scripts folder: put `build.gradle.kts`, `settings.gradle.kts` and `.sdk/` there. The client will see it is a project and keep `.sdk` up to date on every launch. Without those files it leaves the folder alone.
