# Предсказание

Предсказание отвечает на вопрос «что будет, если ничего не менять»: где я окажусь через несколько тиков и куда упадёт снаряд.

## Где я буду

```kotlin
val soon = prediction.after(10)      // через 10 тиков

soon.position()
soon.velocity()
soon.box()
soon.onGround()
soon.inWater()
soon.sneaking()
soon.sprinting()
soon.jumping()
soon.fallDistanceBlocks()
```

Считается по текущей скорости, вводу и физике мира. Ввод игрока в будущем неизвестен, так что чем дальше в тики, тем менее точно. Практический потолок — десяток-другой тиков.

Пример: не прыгать, если через пять тиков всё равно окажешься в воде.

```kotlin
if (prediction.after(5).inWater()) return@on
```

## А если бы я нажал другое

Вариант выше держит зажатыми твои текущие клавиши весь прогон. Передай `MoveInput` — и прогон пойдёт на тех клавишах, которые назовёшь. Так и спрашивают «что было бы, если бы я отсюда прыгнул», не прыгая:

```kotlin
prediction.after(3, moveInput(forward = true, sprint = true, jump = true))
```

У `moveInput(...)` есть `forward`, `backward`, `left`, `right`, `jump`, `sneak`, `sprint` — всё `false`, пока не скажешь иначе. Чтобы отталкиваться не от пустоты, а от реально зажатых клавиш, возьми их из события и поменяй нужную:

```kotlin
on<MoveInputEvent> { e ->
    val jumped = e.toInput().copy(jump = true)
    if (prediction.after(2, jumped).onGround()) {
        e.jump(true)
    }
}
```

`copy` работает как у настроек — что не указал, остаётся как было.

Всё остальное про тебя по-прежнему берётся из настоящего: позиция, скорость, эффекты, атрибуты, предмет в руках. Меняются только клавиши.

## Каждый тик, а не только последний

`after(n)` отдаёт тик `n` и ничего не говорит про тики по дороге. Когда вопрос звучит как «в какой-то момент за n тиков», спрашивай весь прогон:

```kotlin
prediction.path(10)                 // десять результатов, с первого тика по десятый
prediction.path(10, someInput)      // то же самое, на названных клавишах
```

Одна симуляция, один результат на тик, по порядку. Это близнец `projectile().points()`, только про движение.

Прыжок по лестнице — вся идея в одну строку. На ровном полу через два тика после прыжка ты ещё в воздухе, а на ступеньке уже стоишь на ней, так что «окажусь ли я снова на земле почти сразу» — ровно тот вопрос, который их различает:

```kotlin
val landsFast = prediction.path(2, e.toInput().copy(jump = true)).any { it.onGround() }
```

Не собирай это из `after` в цикле: `path(n)` прогоняет симуляцию один раз, а `(1..n).map { after(it) }` — n раз подряд.

## Остановиться раньше

Попросить сто тиков не значит заплатить за сто. Добавь условие — и прогон остановится на первом тике, который ему удовлетворяет: этот тик войдёт в список, дальше ничего не считается.

```kotlin
val fall = prediction.path(100) { it.onGround() }

fall.size                    // сколько тиков до приземления
fall.last().position()       // где приземлишься
```

Если условие так и не сработало, вернутся все `ticks` результатов — так что приземлился ты или просто кончились тики, показывает `fall.last().onGround()`.

Условие видит каждый тик по мере счёта, в этом и смысл: `path(100) { it.inWater() }`, `path(60) { it.fallDistanceBlocks() > 3.0 }`, `path(40, someInput) { !it.sprinting() }`.

Одно правило: **не вызывай предсказание изнутри условия.** Симуляция идёт на одной общей сущности, и вложенный вызов тихо испортит тот прогон, внутри которого он сидит. За такое прилетит `ScriptStateException`, а не кривые числа.

## Куда полетит снаряд

```kotlin
val path = prediction.projectile(ProjectileKind.ARROW)

path.points()             // траектория точками
path.lands()              // долетит ли вообще
path.landingPosition()    // куда упадёт
path.hitEntity()          // в кого попадёт, или null
path.hitsBlock()
path.flightTicks()        // сколько тиков лететь
```

Виды снарядов: `ARROW`, `TRIDENT`, `ENDER_PEARL`, `SPLASH_POTION`, `SNOWBALL`, `WIND_CHARGE`.

По умолчанию считается для текущего угла и полного заряда. Можно задать своё:

```kotlin
prediction.projectile(ProjectileKind.ARROW, aim)
prediction.projectile(ProjectileKind.ARROW, aim, 0.7f)
```

## Стрелять только по цели

Самое частое применение — не тратить снаряд впустую:

```kotlin
fun willHitEnemy(kind: ProjectileKind): Boolean {
    val victim = prediction.projectile(kind).hitEntity() ?: return false
    return victim.alive() && !victim.isSelf() && !victim.isAlly()
}

on<ClientTickEvent> {
    if (!player.usingItem()) return@on
    if (!inventory.held().isA("trident")) return@on
    if (player.itemUseTicks() < 10) return@on
    if (!willHitEnemy(ProjectileKind.TRIDENT)) return@on
    interaction.stopUsingItem()
}
```

## Нарисовать траекторию

`points()` — готовый список точек, его можно скормить [рендеру 3D](../ui/render-3d.md):

```kotlin
on<Render3DEvent> { e ->
    val path = prediction.projectile(ProjectileKind.ENDER_PEARL)
    val points = path.points()
    for (i in 0 until points.size - 1) {
        e.render().line(points[i], points[i + 1], Colors.CYAN, true)
    }
    path.landingPosition()?.let {
        e.render().box(Box.around(it, 0.2), Colors.RED, true)
    }
}
```

## Про стоимость

Каждый запрос — это симуляция, а не чтение поля. Один-два раза за тик нормально, в цикле по всем сущностям — нет. Если нужен результат несколько раз за тик, посчитай один раз и положи в переменную.
