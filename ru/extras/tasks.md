# Таймеры и задачи

`timer()` на каждый вызов отдаёт новый независимый секундомер. Запланированные действия выполняются на клиентском потоке и только пока скрипт включён.

```kotlin
val sinceAttack = timer()

on<ClientTickEvent> {
    if (sinceAttack.passedAndReset(10)) interaction.attackCrosshair()
}

everyTicks(100) { chat.print("still here") }
```

## Таймеры

| Метод | Тип | Описание |
|---|---|---|
| `timer()` | `Timer` | новый независимый секундомер, уже запущен |
| `elapsedTicks()` | `int` | клиентских тиков с последнего сброса, 0..2147483647 |
| `elapsedMillis()` | `long` | миллисекунд с последнего сброса, никогда не отрицательное |
| `passed(ticks)` | `boolean` | true, когда `elapsedTicks() >= ticks` |
| `passedMillis(ms)` | `boolean` | true, когда `elapsedMillis() >= ms` |
| `passedAndReset(ticks)` | `boolean` | `passed(ticks)` и сброс, если вернул true |
| `passedMillisAndReset(ms)` | `boolean` | `passedMillis(ms)` и сброс, если вернул true |
| `reset()` | `void` | заново запускает отсчёт тиков и миллисекунд |

Поля таймера volatile, поэтому его можно читать и сбрасывать вне клиентского потока.

## Планирование

| Метод | Тип | Описание |
|---|---|---|
| `nextTick(action)` | `Task` | выполняется один раз через 1 клиентский тик |
| `afterTicks(ticks, action)` | `Task` | выполняется один раз через столько клиентских тиков (бросает `ScriptException` при ticks < 0) |
| `everyTicks(ticks, action)` | `Task` | повторяется каждые `ticks` клиентских тиков, первый запуск через 2×`ticks` (бросает `ScriptException` при ticks < 1) |
| `client.tasks().nextTick(action)` | `Task` | форма `nextTick` у объекта `Tasks` |
| `client.tasks().afterTicks(ticks, action)` | `Task` | форма `afterTicks` у объекта `Tasks` |
| `client.tasks().everyTicks(ticks, action)` | `Task` | форма `everyTicks` у объекта `Tasks` |

`afterTicks(0, ...)` принимается, но задача снимается раньше, чем успевает выполниться; null вместо действия бросает `NullPointerException`.
Запланированное действие пропускается, пока скрипт выключен или выгружен, исключение уходит в консоль скриптов и не всплывает, а при выгрузке все задачи снимаются.

## Ручка задачи

| Метод | Тип | Описание |
|---|---|---|
| `cancel()` | `void` | прекращает выполнение и убирает задачу из списка скрипта |
| `cancelled()` | `boolean` | true после отмены |
| `close()` | `void` | вызывает `cancel()`, `Task` — `AutoCloseable` |

## Перейти на клиентский поток

| Метод | Тип | Описание |
|---|---|---|
| `onClientThread(action)` | `Unit` | отдаёт действие клиентскому потоку, сразу же, если ты уже на нём |
| `client.tasks().onClientThread(action)` | `void` | форма `onClientThread` у объекта `Tasks` |

`PacketReceiveEvent` (IO-поток netty) и `PacketSendEvent` (поток отправки) — единственные события не на клиентском потоке, см. [Пакеты](../actions/packets.md).
Вызовы мира, игрока, инвентаря, контейнера, взаимодействия и поворотов вне клиентского потока бросают `ScriptThreadException`.

## Время

| Метод | Тип | Описание |
|---|---|---|
| `client.tick()` | `long` | клиентских тиков с запуска, за сессию не сбрасывается |
| `client.millis()` | `long` | системное время в миллисекундах, во `float` теряет точность |
| `client.nanos()` | `long` | монотонные наносекунды, `System.nanoTime()` |
| `client.fps()` | `int` | кадров в секунду по данным клиента |
| `client.tickDelta()` | `float` | прогресс кадра между двумя тиками, 0..1 |
| `client.onClientThread()` | `boolean` | true, когда вызов идёт на клиентском потоке |
