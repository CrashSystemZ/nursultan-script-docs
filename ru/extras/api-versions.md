# Версии API

`ApiVersion.CURRENT` равен 2. `requireApi(n)` не даёт скрипту загрузиться на клиенте постарше указанного, но от ошибки компиляции он не спасает: имя, которого в старом SDK нет, не скомпилируется вообще.

```kotlin
requireApi(2)

name("Mesh demo")

// gpu появился в API 2; на клиенте v1 этот файл не загрузится
val format = gpu.format(VertexAttribute.floats(3), VertexAttribute.color())
val mesh = gpu.indexedMesh(format)
```

## Требование версии

| Метод | Тип | Описание |
|---|---|---|
| `ApiVersion.CURRENT` | `int` | версия скриптового API этого клиента, сейчас 2 |
| `ApiVersion.require(minimum)` | `void` | статический (бросает `ScriptApiException`, когда `CURRENT` < `minimum`) |
| `requireApi(minimum)` | `Unit` | форма `ApiVersion.require` из DSL (бросает `ScriptApiException`, когда `CURRENT` < `minimum`) |

У `ApiVersion` приватный конструктор: экземпляра нет, только два статических члена.
К каждой ошибке `Unresolved reference` клиент дописывает `this client provides v2`, а компиляция идёт раньше первой строки.

## Что нового во 2

| Появилось во 2 | Где описано |
|---|---|
| `Client.user()`, `assets()`, `clipboard()`, `gpu()`; `Game.recipes()`, `settings()` | [Как устроен скрипт](../start/lifecycle.md) |
| `Assets` — корень `assets` | [Папка assets](assets.md) |
| `base64(...)`, `base64Encode(...)`, `font(name, bytes)`, `image(name, bytes)` | [Папка assets](assets.md) |
| `Clipboard` — корень `clipboard` | [Сообщения](../ui/messages.md) |
| `User`, `DiscordUser` — корень `user` | [Твой аккаунт](user.md) |
| `Config.getList`, `getIntList`, `getDoubleList`, `putList`, `putIntList`, `putDoubleList` | [Сохранение данных](../settings/storage.md) |
| `Hunger` — `player.hunger()` | [Свой игрок](../game/player.md) |
| `SelfPlayer.attackCooldown(tickDelta)`, `cooldownPeriod()`, `ticksSinceLastAttack()` | [Свой игрок](../game/player.md) |
| `LivingEntity.swinging()`, `swingTicks()` | [Сущности и фильтры](../game/entities.md) |
| `GameSettings`, `Perspective` — корень `gameSettings` | [Движение](../actions/control.md) |
| `Recipes`, `RecipeEntry`, `RecipeKind` — корень `recipes` | [Контейнеры](../game/containers.md) |
| `Container.Batch.onFinish`, `Inventory.Batch.onFinish` | [Контейнеры](../game/containers.md), [Инвентарь и предметы](../game/inventory.md) |
| `Item.buildable()` | [Инвентарь и предметы](../game/inventory.md) |
| `Item.cooldownProgress()`, `cooldownProgress(tickDelta)`, `setCooldown(ticks)`, `removeCooldown()` | [Инвентарь и предметы](../game/inventory.md) |
| `TabEntry.pingMs(value)` | [Сервер, табло, таблист](../game/server.md) |
| `input(name, value, placeholder)` | [Виды настроек](../settings/types.md) |
| `Gpu` и 11 типов GPU | [Своя геометрия](../ui/gpu.md) |
| `Weight` и перегрузки `Render` с насыщенностью | [Рендер 2D](../ui/render-2d.md) |
| `blur(x, y, width, height, radius, argb, tl, tr, bl, br)` | [Рендер 2D](../ui/render-2d.md) |
| `Shader.setMat4`, `Shader.set(uniform, texture, filter, wrap)` | [Шейдеры](../ui/shaders.md) |
| `EventOptions(ignoreCancelled)` | [Подписка на события](../events/basics.md) |
| `FireworkEntitySpeedEvent` | [Список событий](../events/reference.md) |
| `PerspectiveEvent` | [Список событий](../events/reference.md) |
| `RenderCrosshairEvent` | [Список событий](../events/reference.md) |
| `Render3DEvent.viewMatrix()`, `projectionMatrix()` | [Список событий](../events/reference.md) |

Ничего не закрыто пометкой по отдельным членам: всё перечисленное есть в клиенте v2 безусловно, а `requireApi(2)` — единственная проверка.
Записи пакетов следуют за версией Minecraft, а не за этим номером — см. [Пакеты](../actions/packets.md).

## Что больше ничего не делает

`@NoEffect` помечает элемент, который остался на месте и компилируется, но его значение клиент нигде не читает.

| Метод | Тип | Описание |
|---|---|---|
| `NoEffect.value()` | `String` | предложение о том, почему элемент не читается |

`@Retention(CLASS)` — виден в IDE и в jar SDK, но не через рефлексию.
Цели: тип, метод, поле, параметр, компонент записи.

| Элемент | Тип | Описание |
|---|---|---|
| `Priority` | `enum` | помечен весь тип, пять констант оставлены ради старых скриптов (ничего не делает: все обработчики скриптов срабатывают в один момент) |
| `EventOptions.priority` | `Priority` | компонент записи и его аксессор (устарело) (ничего не делает: все обработчики скриптов срабатывают в один момент) |
| `EventOptions.priority(priority)` | `EventOptions` | копия `DEFAULT` с этим значением (устарело) (ничего не делает: все обработчики скриптов срабатывают в один момент) |
| `ScriptScope.on<E>(priority, ignoreCancelled) { }` | `Subscription` | аргумент выбрасывается (устарело, убери аргумент) |
| `ScriptScope.on(type, priority, ignoreCancelled) { }` | `Subscription` | аргумент выбрасывается (устарело, убери аргумент) |
| `EntryScope.on<E>(priority, ignoreCancelled) { }` | `Subscription` | аргумент выбрасывается (устарело, убери аргумент) |
| `EntryScope.on(type, priority, ignoreCancelled) { }` | `Subscription` | аргумент выбрасывается (устарело, убери аргумент) |
| `RotationOptions.clientSide` | `boolean` | компонент записи и его аксессор (устарело) (ничего не делает: поворот всегда уходит на сервер) |
| `RotationOptions.clientSide(value)` | `RotationOptions` | копия с этим значением (устарело) (ничего не делает: поворот всегда уходит на сервер) |
| `RotationOptions.normalizeMouseMovement` | `boolean` | компонент записи и его аксессор (устарело) (ничего не делает: угол всегда снапится к шагу мыши) |
| `RotationOptions.normalizeMouseMovement(value)` | `RotationOptions` | копия с этим значением (устарело) (ничего не делает: угол всегда снапится к шагу мыши) |

Все одиннадцать компилируются и хранят то, что ты передал; сохранённое значение никто не читает.
Порядок, который пришёл на смену `Priority`, — на странице [Подписка на события](../events/basics.md#приоритет-ничего-не-делает), два флага поворота — на странице [Повороты](../actions/rotations.md#настройки-поворота).

## Во что обходится повышение

Jar API из SDK называется `nursultan-script-api-v<N>.jar`, поэтому обновление кладёт новый файл и переписывает `build.gradle.kts` на него; IDEA один раз пересинхронизирует проект.
`ApiVersion.CURRENT` входит в ключ кеша компиляции, поэтому повышение обнуляет весь кеш скомпилированных скриптов и заставляет пересобрать их при первом запуске после обновления.
