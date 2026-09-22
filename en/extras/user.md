# Your account

`user` is `client.user()` — the Nursultan account the launcher authenticated, not the Minecraft profile. Every member is read-only, and both types were added in API 2.

```kotlin
requireApi(2)

on<Render2DEvent> { e ->
    val days = user.subscribeTimeMinutes() / 1440L
    e.render().text("${user.username()}: $days d", 46f, 16f, 9f, Colors.WHITE)
    val avatar = user.avatar() ?: return@on
    e.render().texture(avatar, 10f, 10f, 32f, 32f)
}
```

## Who you are

| Method | Type | Description |
|---|---|---|
| `user.username()` | `String` | Nursultan login from the launcher, not the Minecraft nickname (API 2) |
| `user.uid()` | `int` | numeric Nursultan account id (API 2) |
| `user.subscribeTimeMinutes()` | `long` | subscription time left in minutes (API 2) |

All three are read from the launcher arguments at startup and stay fixed for the session; `subscribeTimeMinutes()` does not count down.

## Avatar

| Method | Type | Description |
|---|---|---|
| `user.avatar()` | `Texture?` | avatar texture handle, null when `avatarTextureId()` is 0 (API 2) |
| `user.avatarTextureId()` | `int` | OpenGL texture id of the avatar, 0 when there is none (API 2) |

On the handle, `glId()` returns 0 off the client thread and `width()`/`height()` are the size the launcher supplied, whatever it was. `Texture` itself is documented on [2D render](../ui/render-2d.md#textures).

## Discord

| Method | Type | Description |
|---|---|---|
| `user.discord()` | `DiscordUser?` | Rich Presence account, null when Discord is not connected (API 2) |
| `discord.id()` | `String` | Discord snowflake id as a string, `"0"` when absent (API 2) |
| `discord.username()` | `String` | Discord username, `"Unknown"` when absent (API 2) |
| `discord.discriminator()` | `String` | legacy 4-digit discriminator, `"0"` on migrated accounts (API 2) |
| `discord.globalName()` | `String?` | Discord display (global) name, null when unset (API 2) |
| `discord.displayName()` | `String` | global name when non-blank, otherwise the username (API 2) |
| `discord.tag()` | `String` | `username` when the discriminator is `"0"`, else `username#discriminator` (API 2) |
| `discord.avatarHash()` | `String?` | raw avatar hash, null without a custom avatar (API 2) |
| `discord.avatarUrl()` | `String?` | avatar CDN url, null without a custom avatar (API 2) |
| `discord.effectiveAvatarUrl()` | `String` | `avatarUrl()` or the default Discord avatar url, never null (API 2) |
| `discord.bot()` | `boolean` | true when the account is a bot (API 2) |

`user.discord()` is null until the Rich Presence handshake finishes, so it can turn non-null mid-session. `avatarUrl()` ends in `.gif` when `avatarHash()` starts with `a_`, otherwise `.png`.
