# Версии API

`ApiVersion.CURRENT` равен 7. `requireApi(n)` не даёт скрипту загрузиться на клиенте постарше указанного, но от ошибки компиляции он не спасает: имя, которого в старом SDK нет, не скомпилируется вообще.

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
| `ApiVersion.CURRENT` | `int` | версия скриптового API этого клиента, сейчас 7 |
| `ApiVersion.require(minimum)` | `void` | статический (бросает `ScriptApiException`, когда `CURRENT` < `minimum`) |
| `requireApi(minimum)` | `Unit` | форма `ApiVersion.require` из DSL (бросает `ScriptApiException`, когда `CURRENT` < `minimum`) |

У `ApiVersion` приватный конструктор: экземпляра нет, только два статических члена, а к каждой ошибке `Unresolved reference` клиент дописывает `this client provides v7`.
Записи пакетов следуют за версией Minecraft, а не за этим номером — см. [Пакеты](../actions/packets.md).

## Что добавила каждая версия

### API 3

| Добавлено в 3 | Где описано |
|---|---|
| `combat.explosionExposure(target, source)` | [Взаимодействие](../actions/interaction.md) |
| `combat.explosionDamage(target, source, power)` | [Взаимодействие](../actions/interaction.md) |
| `combat.explosionDamageTaken(target, source, power)` | [Взаимодействие](../actions/interaction.md) |
| `combat.damageAfterArmor(target, damage)` | [Взаимодействие](../actions/interaction.md) |
| `living.visibleEffects()` | [Сущности и фильтры](../game/entities.md) |
| `block.collisionBoxes()` | [Мир и блоки](../game/world.md) |
| `block.outlineBoxes()` | [Мир и блоки](../game/world.md) |
| `world.blockCollisionsIn(box)` | [Мир и блоки](../game/world.md) |
| `world.isBlockSpaceFree(box)` | [Мир и блоки](../game/world.md) |
| `RenderItemEvent.translate(x, y, z)` | [Список событий](../events/reference.md) |
| `RenderItemEvent.rotate(degrees, axisX, axisY, axisZ)` | [Список событий](../events/reference.md) |
| `RenderItemEvent.rotateX(degrees)` | [Список событий](../events/reference.md) |
| `RenderItemEvent.rotateY(degrees)` | [Список событий](../events/reference.md) |
| `RenderItemEvent.rotateZ(degrees)` | [Список событий](../events/reference.md) |
| `RenderItemEvent.scale(x, y, z)` | [Список событий](../events/reference.md) |
| `RenderItemEvent.Matrix` | [Список событий](../events/reference.md) |

### API 4

| Добавлено в 4 | Где описано |
|---|---|
| `client.party()` | [Как устроен скрипт](../start/lifecycle.md) |
| `party` | [Сообщения в группе](party.md) |
| `Party` | [Сообщения в группе](party.md) |
| `PartyChannel` | [Сообщения в группе](party.md) |
| `PartyMember` | [Сообщения в группе](party.md) |
| `PartyMessage` | [Сообщения в группе](party.md) |
| `PartyShapedMessage` | [Сообщения в группе](party.md) |
| `PartyMessageKind` | [Сообщения в группе](party.md) |
| `PartyFields` | [Сообщения в группе](party.md) |
| `PartyShape` | [Сообщения в группе](party.md) |
| `PartyShapeBuilder` | [Сообщения в группе](party.md) |
| `PartyField` | [Сообщения в группе](party.md) |
| `PartyFieldType` | [Сообщения в группе](party.md) |
| `PartyWire` | [Сообщения в группе](party.md) |
| `PartyStruct` | [Сообщения в группе](party.md) |
| `PartyShapedWriter` | [Сообщения в группе](party.md) |
| `PartyPayloadWriter` | [Сообщения в группе](party.md) |
| `PartyPayloadReader` | [Сообщения в группе](party.md) |
| `PartyTarget` | [Сообщения в группе](party.md) |
| `PartyTargetKind` | [Сообщения в группе](party.md) |
| `SenderRule` | [Сообщения в группе](party.md) |
| `PartySendResult` | [Сообщения в группе](party.md) |
| 43 хелпера группы в `nursultan.dsl` — `shape`, объявления полей, `send`, `publish`, типизированное чтение полей | [Сообщения в группе](party.md) |
| `nursultan.party.*` в импортах по умолчанию | [Сообщения в группе](party.md) |
| `entity.isItem()` | [Сущности и фильтры](../game/entities.md) |
| `entity.asItemEntity()` | [Сущности и фильтры](../game/entities.md) |
| `ItemEntity` | [Сущности и фильтры](../game/entities.md) |

### API 5

| Добавлено в 5 | Где описано |
|---|---|
| `party.code()` | [Сообщения в группе](party.md) |
| `PartyMember.color()` | [Сообщения в группе](party.md) |
| `PartyMember.position()` | [Сообщения в группе](party.md) |
| `PartyMember.positionAge()` | [Сообщения в группе](party.md) |
| `player.serverSprinting()` | [Свой игрок](../game/player.md) |
| `player.velocity(value)` | [Свой игрок](../game/player.md) |
| `render.pushScissor(x, y, width, height)` | [Рендер 2D](../ui/render-2d.md) |
| `render.popScissor()` | [Рендер 2D](../ui/render-2d.md) |

### API 6

| Добавлено в 6 | Где описано |
|---|---|
| `rotations.quantized(rotation)` | [Повороты](../actions/rotations.md) |
| `BackRotation` | [Повороты](../actions/rotations.md) |
| `BackRotation.step(from, to, tick)` | [Повороты](../actions/rotations.md) |
| `BackRotation.maxTicks()` | [Повороты](../actions/rotations.md) |
| `BackRotations` | [Повороты](../actions/rotations.md) |
| `BackRotations.SNAP` | [Повороты](../actions/rotations.md) |
| `BackRotations.INSTANT` | [Повороты](../actions/rotations.md) |
| `BackRotations.HUMANIZED` | [Повороты](../actions/rotations.md) |
| `backRotation(maxTicks) { }` | [Повороты](../actions/rotations.md) |

### API 7

| Добавлено в 7 | Где описано |
|---|---|
| `entity.yaw(value)` | [Сущности и фильтры](../game/entities.md) |
| `entity.pitch(value)` | [Сущности и фильтры](../game/entities.md) |
| `entity.velocity(value)` | [Сущности и фильтры](../game/entities.md) |
| `entity.fallDistanceBlocks(value)` | [Сущности и фильтры](../game/entities.md) |
| `entity.noClip()` | [Сущности и фильтры](../game/entities.md) |
| `entity.noClip(value)` | [Сущности и фильтры](../game/entities.md) |

Ничего не закрыто пометкой по отдельным членам: всё перечисленное выше есть в клиенте своей версии безусловно, а `requireApi(n)` — единственная проверка.
API 1 — это та часть, у которой пометки нет вообще; члены API 2 помечены `(API 2)` в таблицах той страницы, которая их описывает.

## Что больше ничего не делает

`@NoEffect` помечает элемент, который остался на месте и компилируется, но его значение клиент нигде не читает.

| Метод | Тип | Описание |
|---|---|---|
| `NoEffect.value()` | `String` | предложение о том, почему элемент не читается |

`@Retention(CLASS)` — виден в IDE и в jar SDK, но не через рефлексию.
Цели: тип, метод, поле, параметр, компонент записи.

### Что помечено

| Элемент | Тип | Описание |
|---|---|---|
| `Priority` | `enum` | помечен весь тип, пять констант оставлены ради старых скриптов (ничего не делает: порядок вызова не меняется) |
| `EventOptions.priority()` | `Priority` | компонент записи и его аксессор (устарело) (ничего не делает: значение нигде не читается) |
| `EventOptions.priority(priority)` | `EventOptions` | копия `DEFAULT` с этим значением (устарело) (ничего не делает: значение нигде не читается) |
| `ScriptScope.on<E>(priority, ignoreCancelled) { }` | `Subscription` | аргумент выбрасывается (устарело, убери аргумент) |
| `ScriptScope.on(type, priority, ignoreCancelled) { }` | `Subscription` | аргумент выбрасывается (устарело, убери аргумент) |
| `EntryScope.on<E>(priority, ignoreCancelled) { }` | `Subscription` | аргумент выбрасывается (устарело, убери аргумент) |
| `EntryScope.on(type, priority, ignoreCancelled) { }` | `Subscription` | аргумент выбрасывается (устарело, убери аргумент) |
| `RotationOptions.clientSide()` | `boolean` | компонент записи и его аксессор (устарело) (ничего не делает: поворот всегда уходит на сервер) |
| `RotationOptions.clientSide(value)` | `RotationOptions` | копия с этим значением (устарело) (ничего не делает: поворот всегда уходит на сервер) |
| `RotationOptions.normalizeMouseMovement()` | `boolean` | компонент записи и его аксессор (устарело, используй `rotations.quantized`) (ничего не делает: значение нигде не читается) |
| `RotationOptions.normalizeMouseMovement(value)` | `RotationOptions` | копия с этим значением (устарело, используй `rotations.quantized`) (ничего не делает: значение нигде не читается) |

Все одиннадцать компилируются и хранят то, что ты передал; сохранённое значение никто не читает.
Порядок, который пришёл на смену `Priority`, — на странице [Подписка на события](../events/basics.md#приоритет-ничего-не-делает), два флага поворота — на странице [Повороты](../actions/rotations.md#настройки-поворота).
Каждый из них даёт предупреждение компиляции в консоли скрипта, со строкой, на которой он стоит.

### Устарело, но работает

| Член | Тип | Описание |
|---|---|---|
| `RotationOptions.smoothBackRotation()` | `boolean` | компонент записи и его аксессор (устарело, используй `backRotation`) |
| `RotationOptions.smoothBackRotation(value)` | `RotationOptions` | true превращает возврат `SNAP` в `HUMANIZED` (устарело, используй `backRotation`) |
| `BackRotation.FAST` | `BackRotation` | тот же возврат, что у `BackRotations.SNAP` (устарело, используй `BackRotations.SNAP`) |
| `BackRotation.SMOOTH` | `BackRotation` | тот же возврат, что у `BackRotations.HUMANIZED` (устарело, используй `BackRotations.HUMANIZED`) |

Эти четыре — псевдонимы, а не заглушки: при переписывании возврата головы в API 6 они сохранили прежнюю форму, поэтому скрипт, написанный до него, ведёт себя так же.

## Во что обходится повышение

Jar API из SDK называется `nursultan-script-api-v<N>.jar`, поэтому обновление кладёт новый файл и переписывает `build.gradle.kts` на него; IDEA один раз пересинхронизирует проект.
`ApiVersion.CURRENT` входит в ключ кеша компиляции, поэтому повышение обнуляет весь кеш скомпилированных скриптов и заставляет пересобрать их при первом запуске после обновления.
