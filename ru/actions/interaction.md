# Взаимодействие

`interaction` — это `game.interaction()`: удары, использование, махи рукой и ломание; всё, кроме `breakingBlock()`, работает только на клиентском потоке. `combat` — это `client.combat()`, он даёт точку прицеливания на хитбоксе цели.

```kotlin
on<ClientTickEvent> {
    val target = raycast.entityAtCrosshair(player.entityReachBlocks()) ?: return@on
    if (player.attackCooldown() < 1f) return@on
    val point = combat.attackPoint(target, AttackPoint.MULTI_POINT, 3.0, false)
    rotations.apply(rotations.lookAt(point))
    interaction.attack(target)
}
```

## Атака

| Метод | Тип | Описание |
|---|---|---|
| `interaction.attack(target)` | `void` | бьёт сущность и машет основной рукой (только главный поток) (бросает `ScriptStateException`, если цель не жива) |
| `interaction.attackCrosshair()` | `void` | ванильный удар по тому, что под прицелом (только главный поток) |
| `interaction.swing(hand)` | `void` | проигрывает мах и отправляет пакет маха (только главный поток) |

Константы `Hand` — на странице [Инвентарь и предметы](../game/inventory.md#руки).

## Куда бить

| Метод | Тип | Описание |
|---|---|---|
| `combat.attackPoint(target)` | [`Vec`](../game/math.md#vec) | точка `NEAREST`, макс. дистанция = досягаемость по сущностям, сквозь стены |
| `combat.attackPoint(target, mode, maxDistanceBlocks, throughWalls)` | [`Vec`](../game/math.md#vec) | точка на хитбоксе; дистанцию и стены читают только `MULTI_POINT` и `TRIANGLE` (бросает `ScriptException`, если `maxDistanceBlocks` не больше 0) |
| `combat.markTarget(target, timeoutTicks)` | `void` | помечает цель TargetESP клиента на столько тиков (только главный поток) (бросает `ScriptException`, если `timeoutTicks` не больше 0) |
| `combat.target()` | [`Entity`](../game/entities.md)`?` | текущая цель TargetESP, null если её нет |

Null в `mode` считается за `NEAREST`; обе перегрузки `attackPoint` бросают `ScriptStateException`, когда сущность покинула мир.
`markTarget` бросает `ScriptStateException`, если цель не живая сущность.

### AttackPoint

| Константа | Описание |
|---|---|
| `CENTER` | геометрический центр коробки |
| `NEAREST` | ближайшая к глазам точка поверхности, хитбокс сжат на 0.01 блока |
| `MULTI_POINT` | лучшая из сетки 11×11 по видимым граням, каждая точка проверяется лучом |
| `TRIANGLE` | по одной точке на видимую грань, сортировка по досягаемости и дельте мыши |

## Урон от взрыва

| Метод | Тип | Описание |
|---|---|---|
| `combat.explosionExposure(target, source)` | `float` | доля хитбокса 0..1, которую блоки не закрывают (API 3) (только главный поток) |
| `combat.explosionExposure(target, source, cover)` | `float` | то же с учётом коробок, которых в мире ещё нет (только главный поток) |
| `combat.explosionDamage(target, source, power)` | `float` | урон до брони и эффектов, 0 за радиусом (API 3) (только главный поток) (бросает `ScriptException`, если `power` не больше 0) |
| `combat.explosionDamage(target, source, power, cover)` | `float` | то же с учётом коробок, которых в мире ещё нет (только главный поток) (бросает `ScriptException`, если `power` не больше 0) |
| `combat.explosionDamageTaken(target, source, power)` | `float` | то же после брони, сопротивления и защиты (API 3) (только главный поток) (бросает `ScriptException`, если `power` не больше 0) |
| `combat.damageAfterArmor(target, damage)` | `float` | остаток урона после единиц брони и прочности, пробитие оружием не учтено (API 3) |

## Использовать предмет

| Метод | Тип | Описание |
|---|---|---|
| `interaction.useItem(hand)` | `void` | начинает использование предмета в этой руке (только главный поток) |
| `interaction.useCrosshair()` | `void` | ванильное использование того, что под прицелом (только главный поток) |
| `interaction.stopUsingItem()` | `void` | отпускает использование: выстрел из лука, бросок трезубца (только главный поток) |
| `interaction.swapHands()` | `void` | отправляет пакет обмена рук (только главный поток) (бросает `ScriptStateException`, если нет подключения) |

Игра сама зовёт `stopUsingItem` на следующем тике, пока клавиша использования не зажата физически, поэтому начатое из скрипта использование сразу обрывается, если не отменять `StopUsingItemEvent`.

## Блоки

| Метод | Тип | Описание |
|---|---|---|
| `interaction.useBlock(x, y, z, side, hand)` | `void` | взаимодействует с центром этой грани блока (только главный поток) |
| `interaction.placeBlock(x, y, z, side, hand)` | `void` | то же самое, что `useBlock`, делегирует ему (только главный поток) |
| `interaction.useBlock(x, y, z, side, hand, hit)` | `void` | взаимодействует в этой точке грани, а не в её центре (только главный поток) |
| `interaction.placeBlock(x, y, z, side, hand, hit)` | `void` | то же самое с точкой попадания, делегирует `useBlock` (только главный поток) |
| `interaction.startBreaking(x, y, z, side)` | `void` | начинает ломать этот блок (только главный поток) |
| `interaction.continueBreaking(x, y, z, side)` | `boolean` | продвигает ломание, true когда блок сломался (только главный поток) |
| `interaction.pendingAck()` | `boolean` | твоё использование блока или удар ещё ждёт подтверждения сервера |
| `interaction.stopBreaking()` | `void` | отменяет текущий прогресс ломания (только главный поток) |
| `interaction.breakingBlock()` | `boolean` | true пока блок ломается, false вне мира |

Константы `Side` и `opposite()` — на странице [Лучи и прицел](../game/raycast.md#стороны).
