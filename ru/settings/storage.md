# Сохранение данных

Значения настроек клиент сохраняет сам — их писать никуда не надо. Конфиги нужны для всего остального: счётчиков, списков, домов, статистики.

## Быстрый старт

`storage` — конфиг по умолчанию, он есть у каждого скрипта:

```kotlin
val runs = storage.getInt("runs", 0) + 1
storage.put("runs", runs)
storage.save()

chat.print("запуск номер $runs")
```

Значения лежат в памяти, `save()` пишет их на диск. Забыл сохранить — не страшно: при выгрузке скрипта несохранённое допишется само.

## Несколько конфигов

Если данных много и они разные, разложи по файлам:

```kotlin
val stats = config("stats")
val homes = config("homes")

stats.put("kills", stats.getInt("kills", 0) + 1)
stats.save()
```

Имя конфига — от 1 до 32 символов из `a-z`, `0-9`, `_` и `-`.

```kotlin
configs.names()            // какие конфиги уже есть на диске
configs.exists("stats")
configs.delete("stats")
```

## Что умеет конфиг

```kotlin
storage.get("name", "по умолчанию")
storage.getInt("count", 0)
storage.getLong("time", 0L)
storage.getDouble("ratio", 1.0)
storage.getBoolean("on", false)

storage.put("name", "текст")
storage.put("count", 5)
storage.put("on", true)

storage.has("count")
storage.remove("count")
storage.clear()
storage.keys()             // все ключи
storage.dirty()            // есть ли несохранённые изменения

storage.save()
storage.load()             // перечитать с диска, потеряв несохранённое
storage.delete()           // удалить файл
```

Второй аргумент у `get*` — что вернуть, если ключа нет. Если в ключе лежит мусор, который не разбирается в число, тоже вернётся он.

## Хранить не только строки

Конфиг хранит пары «строка → строка». Всё остальное раскладывай по ключам сам:

```kotlin
// список
storage.put("homes", homes.joinToString(","))
val homes = storage.get("homes", "").split(",").filter { it.isNotEmpty() }

// объект
storage.put("home.base.x", x)
storage.put("home.base.y", y)
storage.put("home.base.z", z)

// найти все дома
val names = storage.keys()
    .filter { it.startsWith("home.") && it.endsWith(".x") }
    .map { it.removePrefix("home.").removeSuffix(".x") }
```

## Где это лежит

Файлы хранятся отдельно по каждому скрипту, сжатые и зашифрованные, в папке конфигов клиента. Руками туда лезть не нужно и нечего — формат внутренний.

Ограничения: до 4096 ключей на конфиг и до 64 КиБ на значение. Упрёшься — получишь понятную ошибку.

## Когда сохранять

`save()` пишет файл на диске сразу, поэтому не вызывай его каждый тик. Нормальные места:

* в обработчике команды, после того как игрок что-то изменил;
* в `onDisable` и `onUnload`;
* раз в N секунд через [`everyTicks`](../extras/tasks.md), если данные копятся постоянно.
