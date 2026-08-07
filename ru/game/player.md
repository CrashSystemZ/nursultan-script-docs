# Свой игрок

`player` — это ты. Всё, что есть у любой [сущности](entities.md), у него тоже есть, плюс то, что видно только про себя.

Данные читаются в момент вызова: `player.health()` даёт здоровье прямо сейчас, кэшировать значения между тиками не стоит.

Почти всё имеет смысл только в игре, так что начинай с проверки:

```kotlin
on<ClientTickEvent> {
    if (!inGame) return@on
    chat.print("здоровье " + player.health())
}
```

или короче:

```kotlin
on<ClientTickEvent> {
    whenInGame {
        chat.print("здоровье " + player.health())
    }
}
```

## Где я и куда смотрю

```kotlin
player.position()            // Vec ног
player.eyePosition()         // Vec глаз
player.x()  player.y()  player.z()
player.previousPosition()    // где был в прошлом тике
player.renderPosition()      // где рисуется в этом кадре

player.rotation()            // Rotation
player.yaw()  player.pitch()
player.velocity()            // Vec скорости
```

`renderPosition()` — то, что нужно для рисования: позиция уже сглажена между тиками. `position()` — «настоящая» позиция для логики.

## Состояние

```kotlin
player.health()              // 0..20 обычно
player.maxHealth()
player.absorption()          // золотые сердца
player.armorPoints()
player.food()
player.saturation()

player.alive()
player.onGround()
player.sneaking()
player.sprinting()
player.wasSprinting()        // бежал ли в прошлом тике
player.inWater()
player.onFire()
player.flying()              // летит в креативе
player.gliding()             // элитры
player.climbing()            // на лестнице
player.riding()              // сидит на лошади/лодке
player.creative()
player.gameMode()
player.pingMs()
player.fallDistanceBlocks()
player.hasMovementInput()    // жмёт ли клавиши движения
```

## Бой

```kotlin
player.attackCooldown()             // 0..1, где 1 это полный замах
player.belowMinimumAttackCharge()   // бить сейчас смысла нет
player.entityReachBlocks()          // до кого дотянешься
player.blockReachBlocks()           // до какого блока дотянешься
player.usingItem()                  // держит ПКМ
player.usingHand()                  // какой рукой
player.itemUseTicks()               // сколько тиков уже держит
player.blocking()                   // блокирует щитом
player.hurtTicks()                  // сколько тиков назад получил урон
```

Типовая проверка «можно бить»:

```kotlin
if (player.attackCooldown() >= 0.9f && !player.belowMinimumAttackCharge()) {
    interaction.attack(target)
}
```

## Эффекты

```kotlin
player.hasEffect("speed")
player.effect("speed")?.amplifier()
player.effects()             // все эффекты
```

Каждый эффект несёт `id()`, `name()`, `amplifier()`, `durationTicks()`, `infinite()`, `beneficial()`.

## Предметы на себе

```kotlin
player.mainHandItem()
player.offHandItem()
player.armorItem(ArmorSlot.HELMET)
player.armorItems()
player.isNaked()             // брони нет вообще
```

Инвентарь целиком — в [Инвентаре](inventory.md).

## Разное

```kotlin
player.respawn()             // нажать «Возродиться»
player.xpLevel()
player.xpProgress()
player.airTicks()
```

## Векторы

Позиции — это `Vec` с обычной арифметикой:

```kotlin
val target = player.position().add(0.0, 1.0, 0.0)
val distance = player.position().distanceTo(target)
val direction = target.subtract(player.position()).normalize()
```

`squaredDistanceTo` дешевле, чем `distanceTo` — если надо только сравнить расстояния, бери его.
