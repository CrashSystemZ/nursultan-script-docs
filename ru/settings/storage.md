# Сохранение данных

`storage` — конфиг скрипта по умолчанию, `config("name")` открывает любой другой. Значение всегда строка, файл лежит в `%APPDATA%/Nursultan/configs/<uid>/scripts/<scriptId>/<name>.dat`.

```kotlin
var runs = 0

onEnable {
    runs = storage.getInt("runs", 0) + 1
}

onDisable {
    storage.put("runs", runs)
    storage.save()
}
```

## Открыть конфиг

| Метод | Тип | Описание |
|---|---|---|
| `config(name)` | `Config` | открывает именованный конфиг скрипта, кэшируется по имени |
| `configs.open(name)` | `Config` | то же, что `config(name)` |
| `configs.names()` | `List<String>` | файлы конфигов этого скрипта на диске, по алфавиту |
| `configs.exists(name)` | `boolean` | файл конфига есть на диске |
| `configs.delete(name)` | `void` | удаляет файл и очищает открытый экземпляр |

Имя обрезается по краям, приводится к нижнему регистру и должно совпасть с `^[a-z0-9][a-z0-9_-]{0,31}$`, иначе `ScriptException`.
`storage` — это конфиг с именем `storage`.

## Чтение

| Метод | Тип | Описание |
|---|---|---|
| `storage.name()` | `String` | имя конфига в нижнем регистре |
| `storage.has(key)` | `boolean` | ключ есть в памяти |
| `storage.get(key, fallback)` | `String` | значение, либо fallback, если ключа нет |
| `storage.getInt(key, fallback)` | `int` | разбор в int, fallback если ключа нет или мусор |
| `storage.getLong(key, fallback)` | `long` | разбор в long, fallback если ключа нет или мусор |
| `storage.getDouble(key, fallback)` | `double` | разбор в double, fallback если ключа нет или мусор |
| `storage.getBoolean(key, fallback)` | `boolean` | true только у `true` без учёта регистра, fallback если ключа нет |
| `storage.getList(key)` | `List<String>` | сохранённый список, пустой если ключа нет (API 2) |
| `storage.getList(key, fallback)` | `List<String>` | сохранённый список, копия fallback если ключа нет (API 2) |
| `storage.getIntList(key)` | `List<Integer>` | список как int, неразобранные элементы выброшены (API 2) |
| `storage.getDoubleList(key)` | `List<Double>` | список как double, неразобранные элементы выброшены (API 2) |

Пустой ключ бросает `ScriptException` в любом методе, который его принимает.
Значение, записанное через `put`, читается через `getList` как список из одного элемента.

## Запись

| Метод | Тип | Описание |
|---|---|---|
| `storage.put(key, value: String)` | `void` | пишет строку (бросает при длине больше 65536 символов) |
| `storage.put(key, value: Int)` | `void` | пишет десятичную запись числа |
| `storage.put(key, value: Long)` | `void` | пишет десятичную запись числа |
| `storage.put(key, value: Double)` | `void` | пишет десятичную запись числа |
| `storage.put(key, value: Boolean)` | `void` | пишет `true` или `false` |
| `storage.putList(key, values)` | `void` | пишет `List<String>` (API 2) |
| `storage.putIntList(key, values)` | `void` | пишет `List<Integer>` текстом (API 2) |
| `storage.putDoubleList(key, values)` | `void` | пишет `List<Double>` текстом (API 2) |
| `storage.remove(key)` | `void` | убирает ключ |
| `storage.clear()` | `void` | убирает все ключи из памяти |

В конфиге 4096 ключей; попытка добавить ещё один бросает `ScriptException`.
`put` бросает `NullPointerException` на null-значении, списочные методы — `ScriptException` на null-элементе.

## Файл

| Метод | Тип | Описание |
|---|---|---|
| `storage.keys()` | `Set<String>` | неизменяемая копия текущего набора ключей |
| `storage.dirty()` | `boolean` | значения отличаются от последнего save или load |
| `storage.save()` | `void` | пишет файл и сбрасывает `dirty()` |
| `storage.load()` | `void` | перечитывает файл, теряя несохранённое |
| `storage.delete()` | `void` | удаляет файл и очищает значения в памяти |

Каждый открытый конфиг с `dirty()` сохраняется при выгрузке или перезагрузке скрипта.
Файл — сжатый Zstd обфусцированный MessagePack; ошибки записи только логируются, наружу не летят.
