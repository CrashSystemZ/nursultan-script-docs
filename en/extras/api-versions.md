# API versions

Every client ships one version of the script API. A script says which one it needs on the first line:

```kotlin
requireApi(1)
```

On a client older than that, the script refuses to load with a clear message instead of behaving strangely.
New methods say which version they arrived in. If a page does not mention a version, the thing has been there since 1.

## What `requireApi` cannot do

It cannot save you from a **missing method**. Your script is Kotlin source that the client compiles when it loads, and compilation happens before the first line runs — so a script calling something the old client has never heard of fails before `requireApi` gets its turn:

```
auto-jump.kts:14  Unresolved reference: arc
```

That message is what your users will actually see. `requireApi` still earns its place, for two reasons: it catches the cases where the method exists but behaves differently than the version you wrote against, and the client mentions your requested version in the error when a reference does not resolve.

So do both — put the line at the top of the file, and say the version in the script's description, so a user on an old client knows what the error means:

```kotlin
name("AutoJump")
description("Jumps for you. Needs Nursultan script API 2 or newer")

requireApi(2)
```

## What a bump costs you

Updating the SDK replaces `.sdk/nursultan-script-api-vN.jar` with a new file under a new name, and rewrites `build.gradle.kts` to point at it. IDEA notices and re-syncs the project once. That is the whole cost — your scripts do not need editing.

## Things to keep in mind

* Ask for the lowest version that actually works. `requireApi(5)` in a script that only uses version-1 features locks out users for no reason.
* Packets are the exception to all of this — they follow the Minecraft version, not the API version. See [Packets](../actions/packets.md).
