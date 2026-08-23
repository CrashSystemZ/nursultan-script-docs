# The project in your IDE

The script development folder from the site (`nursultan-scripts-v5.zip`) unpacks into `nursultan-scripts/`, a ready IntelliJ IDEA project: the API jars, the build files and an example script. Inside the client's scripts folder the client refreshes `.sdk/` and both build files on every launch.

## What to download

| Path | What it is |
|---|---|
| `example.kts` | example script; every `.kts` in the folder root is one script |
| `assets/` | fonts, images and data the scripts read — see [the assets folder](../extras/assets.md) |
| `.sdk/nursultan-script-api-v<N>.jar` | the API and the `.kts` script definition; `<N>` is the client's script API version, currently 5 |
| `.sdk/nursultan-script-packets-<mcVersion>.jar` | Minecraft packet classes, needed only by scripts that read or send packets; the suffix is the client's Minecraft version |
| `build.gradle.kts`, `settings.gradle.kts` | wire the `.sdk` jars and Kotlin scripting in |
| `README.md` | the same steps, offline |

`.sdk/` is generated: the client overwrites its jars and deletes every `nursultan-script-*.jar` left from another version. Files are rewritten only when their bytes changed, so an unchanged SDK causes no Gradle re-sync.

## Open it in IntelliJ IDEA

1. `File → Open`, pick the folder, "Open as Project", trust the project.
2. Gradle syncs once and pulls the Kotlin plugin, `kotlin-scripting-jvm` and `kotlin-scripting-common` from Maven Central.
3. The project sets `jvmToolchain(21)`, so Gradle needs a JDK 21.

## The script definition

**Settings → Languages & Frameworks → Kotlin → Kotlin Scripting** → *Custom Definitions* → **Search Definitions** → **Nursultan Script** (`.kts`) appears in the list → keep it enabled and **above** *Default Kotlin Script* → **Apply**.

IntelliJ does not discover custom script definitions on its own: until this is applied every `.kts` is red. Resolution is computed per file, so open a script after applying.

## Moving it into the game

1. Copy the `.kts` into `%APPDATA%\Nursultan\scripts`; the file name without `.kts` is the script id.
2. Copy the fonts, images and data it reads into `%APPDATA%\Nursultan\scripts\assets`, keeping the subfolders used in the project.
3. Switch the script on in the Scripts tab.

`.sdk/` and the build files stay behind — the client does not read them. Saving an already loaded file recompiles and reloads the script in the running game.

## Developing in the game folder

`%APPDATA%\Nursultan\scripts` counts as the project when it contains `build.gradle.kts` or a `.sdk/` folder; put the whole development folder there and open that path in IDEA. On every launch the client then rebuilds the `.sdk` jars out of itself and rewrites `build.gradle.kts` and `settings.gradle.kts`, so the SDK never lags behind the client. With neither `build.gradle.kts` nor `.sdk/` present the client leaves the folder alone.
