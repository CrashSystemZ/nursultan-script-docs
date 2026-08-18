# Сообщения

`chat` печатает локально и отправляет на сервер, `log` пишет в консоль скриптов, `notify` кладёт карточку в стек уведомлений клиента, `clipboard` читает и пишет системный буфер обмена. Все четыре — корни скрипта; `chat` — это `client.chat()`, `log` — `client.log()`, `clipboard` — `client.clipboard()`.

```kotlin
on<ClientTickEvent> {
    if (player.health() > 6f) return@on
    chat.print(text.literal("low health").color(Colors.RED))
    notify("low health", NotifyKind.WARN)
    log.error("health " + player.health())
}
```

## Чат

| Метод | Тип | Описание |
|---|---|---|
| `chat.print(message)` | `void` | печатает `[<scriptId>] message` только в твой чат |
| `chat.printPrefixed(prefix, message)` | `void` | то же, но с `[prefix] ` вместо id скрипта |
| `chat.sendToServer(message)` | `void` | отправляет сообщение в чат, ведущий `/` отправляет команду сервера (бросает `NullPointerException` без подключения) |
| `chat.runCommand(command)` | `void` | пересылает текст с клиентским префиксом `.`, поэтому выполняется команда клиента (бросает `NullPointerException` без подключения) |

Аргумент `Text` у `print`/`printPrefixed` сохраняет оформление, всё остальное проходит через `toString`, null печатается как `null` — смотри [Оформленный текст](text.md).
`sendToServer` и `runCommand` игнорируют null и пустые строки; `runCommand` сначала снимает один ведущий символ префикса, а потом добавляет его сам.

## Уведомления

| Метод | Тип | Описание |
|---|---|---|
| `notify(text, kind)` | `void` | кладёт карточку уведомления, `kind` по умолчанию `NotifyKind.OK` |
| `notifications.show(text)` | `void` | карточка с видом OK, null-текст становится `""` |
| `notifications.show(text, kind)` | `void` | карточка с этим стилем, null-вид откатывается к OK |

`notifications` — это `client.notifications()`; `notify(...)` — тот же вызов из корня скрипта.

### NotifyKind

| Константа | Описание |
|---|---|
| `OK` | стиль успеха, по умолчанию |
| `WARN` | стиль предупреждения |
| `FAIL` | стиль ошибки |
| `ACCENT` | стиль акцентного цвета клиента |

## Лог

| Метод | Тип | Описание |
|---|---|---|
| `log.info(message)` | `void` | строка INFO в консоли скриптов и в логе клиента |
| `log.warn(message)` | `void` | строка WARN в консоли скриптов и в логе клиента |
| `log.error(message)` | `void` | строка ERROR в консоли скриптов и в логе клиента |
| `log.error(message, cause)` | `void` | строка ERROR плюс `<file>.kts:<line>: <текст причины>`, полный стектрейс только в логе клиента |

Консоль — меню → **Скрипты** → кнопка с терминалом; в каждой строке время, уровень и id скрипта, хранится последняя 1000 строк.
Загрузку, перезагрузку, включение и ошибки компиляции пишет туда сам клиент, как и всё, что упало внутри скрипта.

## Буфер обмена

| Метод | Тип | Описание |
|---|---|---|
| `clipboard.get()` | `String` | текст из буфера, `""` когда он пуст или в нём лежит не текст (API 2) |
| `clipboard.set(text)` | `void` | заменяет содержимое буфера, null игнорируется (API 2) (бросает `ScriptException` при длине больше 65536 символов) |
| `clipboard.clear()` | `void` | пишет в буфер пустую строку (API 2) |
| `clipboard.empty()` | `boolean` | true, когда `get()` вернул `""`, в том числе по таймауту чтения (API 2) |

Вне клиентского потока `get()`/`empty()` переходят на него и ждут до 1000 мс, отдавая `""`/`true` по таймауту, а `set`/`clear` встают в очередь и не блокируют — смотри [переход на клиентский поток](../extras/tasks.md#перейти-на-клиентский-поток).
