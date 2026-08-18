# Подписка на события

`on<E> { }` регистрирует обработчик — он живёт, пока скрипт включён. Все обработчики скриптов идут на одном фиксированном приоритете: после модулей клиента из `BEFORE_ALL` и `BEFORE`, до модулей из `NOW`.

```kotlin
on<ClientTickEvent> { }

on<UseItemEvent> { e ->
    if (inventory.held().isA("ender_pearl")) {
        e.cancel()
    }
}

val sub = on<AttackEvent>(ignoreCancelled = true) { e -> chat.print(e.target().name()) }
sub.unsubscribe()
```

## Регистрация обработчика

| Метод | Тип | Описание |
|---|---|---|
| `on<E>(ignoreCancelled) { }` | `Subscription` | подписка по типу в угловых скобках, `ignoreCancelled` по умолчанию false (бросает `ScriptException`, если `E` — не поддерживаемое событие) |
| `on(type, ignoreCancelled) { }` | `Subscription` | подписка по классу события (бросает `ScriptException`, если класс не поддерживается) |
| `on<E>(priority, ignoreCancelled) { }` | `Subscription` | (устарело, убери аргумент) |
| `on(type, priority, ignoreCancelled) { }` | `Subscription` | (устарело, убери аргумент) |

`ignoreCancelled = true` пропускает обработчик, если событие уже отменено.

### Events

| Метод | Тип | Описание |
|---|---|---|
| `Events.on(type, handler)` | `Subscription` | подписка с `EventOptions.DEFAULT` |
| `Events.on(type, options, handler)` | `Subscription` | подписка с опциями; null заменяется на `DEFAULT` |
| `Events.supportedEvents()` | `List<String>` | простые имена всех поддерживаемых событий, отсортированы |

`Events` — интерфейс, которому делегируют перегрузки `on(...)` выше. Обработчик, зарегистрированный при выключенном скрипте, активируется при включении.

## Отмена

| Метод | Тип | Описание |
|---|---|---|
| `Cancellable.cancel()` | `void` | помечает эту отправку события отменённой |
| `Cancellable.cancelled()` | `boolean` | true после `cancel()` на этой отправке |
| `CancellableEvent.cancel()` | `void` | ставит флаг, сам его не сбрасывает |
| `CancellableEvent.cancelled()` | `boolean` | значение флага, в начале отправки false |

`CancellableEvent` — базовый класс всех отменяемых событий; флаг сбрасывается перед каждой отправкой, так что отмена не переносится на следующую.
Какие события отменяемы и что именно пропускает отмена каждого — в [списке событий](reference.md).

## Опции

| Метод | Тип | Описание |
|---|---|---|
| `EventOptions(ignoreCancelled)` | `EventOptions` | опции с `Priority.NORMAL` (API 2) |
| `EventOptions(priority, ignoreCancelled)` | `EventOptions` | канонический конструктор (бросает `NullPointerException`, когда priority null) |
| `EventOptions.DEFAULT` | `EventOptions` | `Priority.NORMAL`, `ignoreCancelled = false` |
| `EventOptions.priority()` | `Priority` | (устарело) (ничего не делает: значение нигде не читается) |
| `EventOptions.ignoreCancelled()` | `boolean` | true пропускает обработчик на уже отменённом событии |
| `EventOptions.ignoreCancelled(value)` | `EventOptions` | копия с заменённым флагом |
| `EventOptions.priority(priority)` | `EventOptions` | копия `DEFAULT` с этим приоритетом (устарело) (ничего не делает: значение нигде не читается) |

Перегрузки `on(...)` собирают `EventOptions(ignoreCancelled)` сами; свои опции можно передать только в `Events.on(type, options, handler)`.

## Приоритет ничего не делает

Каждый обработчик скрипта регистрируется на одном фиксированном слоте, `EventPriority.SCRIPT` — после модулей клиента из `BEFORE_ALL` и `BEFORE`, до модулей из `NOW`, `AFTER` и `AFTER_ALL`; между скриптами порядок — порядок регистрации.
Весь enum помечен `@NoEffect` начиная с API 2.

| Константа | Описание |
|---|---|
| `FIRST` | (ничего не делает: порядок вызова не меняется) |
| `EARLY` | (ничего не делает: порядок вызова не меняется) |
| `NORMAL` | значение по умолчанию в `EventOptions` (ничего не делает: порядок вызова не меняется) |
| `LATE` | (ничего не делает: порядок вызова не меняется) |
| `LAST` | (ничего не делает: порядок вызова не меняется) |

Члены, которые всё ещё принимают или возвращают `Priority` — две устаревшие перегрузки `on(priority, ...)` и `EventOptions.priority(...)` — компилируются и выбрасывают значение.

## Отписка

| Метод | Тип | Описание |
|---|---|---|
| `Subscription.unsubscribe()` | `void` | снимает обработчик; повторный вызов ничего не делает |
| `Subscription.active()` | `boolean` | true, пока подписка жива и скрипт загружен |
| `Subscription.close()` | `void` | вызывает `unsubscribe()`; `Subscription` — `AutoCloseable` |

Выключение скрипта снимает все обработчики, включение регистрирует их заново.

## Потоки и бюджет

Все события приходят на клиентском потоке Minecraft, кроме `PacketReceiveEvent` (IO-поток netty) и `PacketSendEvent` (поток, который отправляет пакет) — см. [Пакеты](../actions/packets.md).
На один вызов обработчика даётся 250 мс; превышение выключает скрипт, как и 5 исключений подряд — [Песочница и лимиты](../extras/limits.md#бюджеты).
`Render2DEvent`, `Render3DEvent`, `PacketReceiveEvent` и `PacketSendEvent` полностью игнорируют `EventOptions` — и `priority`, и `ignoreCancelled`.
