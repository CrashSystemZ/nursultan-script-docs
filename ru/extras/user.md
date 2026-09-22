# Твой аккаунт

`user` — это `client.user()`: аккаунт Nursultan, под которым лаунчер авторизовал клиент, а не профиль Minecraft. Всё только на чтение, оба типа появились в API 2.

```kotlin
requireApi(2)

on<Render2DEvent> { e ->
    val days = user.subscribeTimeMinutes() / 1440L
    e.render().text("${user.username()}: $days d", 46f, 16f, 9f, Colors.WHITE)
    val avatar = user.avatar() ?: return@on
    e.render().texture(avatar, 10f, 10f, 32f, 32f)
}
```

## Кто ты

| Метод | Тип | Описание |
|---|---|---|
| `user.username()` | `String` | логин Nursultan из лаунчера, а не ник в Minecraft (API 2) |
| `user.uid()` | `int` | числовой id аккаунта Nursultan (API 2) |
| `user.subscribeTimeMinutes()` | `long` | сколько минут подписки осталось (API 2) |

Все три читаются из аргументов лаунчера на старте и за сессию не меняются; `subscribeTimeMinutes()` не убывает.

## Аватар

| Метод | Тип | Описание |
|---|---|---|
| `user.avatar()` | `Texture?` | текстура аватара, null когда `avatarTextureId()` равен 0 (API 2) |
| `user.avatarTextureId()` | `int` | OpenGL id текстуры аватара, 0 когда аватара нет (API 2) |

У текстуры `glId()` вне клиентского потока возвращает 0, а `width()`/`height()` — размер той картинки, которую передал лаунчер, какой бы она ни была. Сам `Texture` описан в [Рендере 2D](../ui/render-2d.md#текстуры).

## Discord

| Метод | Тип | Описание |
|---|---|---|
| `user.discord()` | `DiscordUser?` | аккаунт Rich Presence, null когда Discord не подключён (API 2) |
| `discord.id()` | `String` | snowflake id Discord строкой, `"0"` когда его нет (API 2) |
| `discord.username()` | `String` | имя пользователя Discord, `"Unknown"` когда его нет (API 2) |
| `discord.discriminator()` | `String` | старый 4-значный дискриминатор, `"0"` у новых аккаунтов (API 2) |
| `discord.globalName()` | `String?` | глобальное отображаемое имя, null когда не задано (API 2) |
| `discord.displayName()` | `String` | глобальное имя, а если оно пустое — username (API 2) |
| `discord.tag()` | `String` | `username` при дискриминаторе `"0"`, иначе `username#discriminator` (API 2) |
| `discord.avatarHash()` | `String?` | сырой хеш аватара, null без своего аватара (API 2) |
| `discord.avatarUrl()` | `String?` | ссылка на аватар в CDN, null без своего аватара (API 2) |
| `discord.effectiveAvatarUrl()` | `String` | `avatarUrl()` или ссылка на дефолтный аватар, никогда не null (API 2) |
| `discord.bot()` | `boolean` | true, когда аккаунт — бот (API 2) |

`user.discord()` остаётся null, пока не пройдёт рукопожатие Rich Presence, так что он может стать ненулевым посреди сессии. `avatarUrl()` заканчивается на `.gif`, когда `avatarHash()` начинается с `a_`, иначе на `.png`.
