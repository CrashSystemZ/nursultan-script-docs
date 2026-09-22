# Песочница и лимиты

Скрипт работает по белому списку: стандартная библиотека Kotlin, `java.time`, `java.math`, `kotlin.random.Random` и всё API `nursultan.*`. На каждый вызов обработчика даётся 250 мс.

```kotlin
on<ClientTickEvent> {
    try {
        if (world.block(player.position()).liquid()) control.jump()
    } catch (error: ScriptStateException) {
        log.warn("no world: ${error.message}")
    }
}
```

## Что доступно

| Пакет | Описание |
|---|---|
| `nursultan.**` | всё API скриптов |
| `kotlin.*` | все классы прямо в пакете `kotlin`, подпакеты не входят |
| `kotlin.collections.**` | списки, мапы, сеты и операции над ними |
| `kotlin.sequences.**` | ленивые последовательности |
| `kotlin.ranges.**` | диапазоны и прогрессии |
| `kotlin.text.**` | строки, регулярки, `StringBuilder` |
| `kotlin.comparisons.**` | сборка компараторов |
| `kotlin.math.**` | математические функции и константы |
| `kotlin.random.**` | `Random` |
| `kotlin.enums.**` | `entries` у енумов |
| `kotlin.annotation.**` | цели и retention аннотаций |
| `kotlin.jvm.internal.**`, `kotlin.jvm.functions.**` | интринсики и типы лямбд, которые генерит компилятор |
| `java.util.stream.**` | стримы и коллекторы |
| `java.util.function.**` | функциональные интерфейсы |
| `java.time.**` | моменты, длительности, даты |
| `java.math.**` | `BigInteger`, `BigDecimal` |
| `java.lang` (отдельные классы) | `String`, `StringBuilder`, `Math`, обёртки чисел, обычные исключения |
| `java.util` (отдельные классы) | коллекции, `Optional`, `UUID`, `Random`, `Locale`, `BitSet`, `Objects` |
| `java.util.concurrent.ThreadLocalRandom` | единственный разрешённый класс из `java.util.concurrent` |
| `kotlin.reflect.KProperty*` | только как тип, для делегатов `by` |

`nursultan.**` и `kotlin.random.Random` — импорты по умолчанию; строки `kotlin.*` закрывают дефолтные импорты самого Kotlin.

## Чего нет

| Запрещено | Описание |
|---|---|
| `java.io`, `java.nio` | файлов нет; на диск ходят только [ассеты](assets.md) и [конфиги](../settings/storage.md) |
| `java.net` | ни сокетов, ни http |
| `java.lang.Thread`, `java.util.concurrent` | ни потоков, ни пулов, ни таймеров, кроме `ThreadLocalRandom` |
| `java.lang.Class`, `java.lang.reflect`, `kotlin.reflect` | рефлексии нет; проходят только типы `KProperty` |
| `java.lang.ClassLoader`, `java.lang.System` | ни загрузки классов, ни доступа к процессу и окружению |
| `net.minecraft.**`, `fun.nursultan.**` | внутренностей игры и клиента напрямую нет |
| `nursultan.internal.BudgetControl` | единственный запрещённый класс внутри `nursultan` |
| нативные методы | `native`-метод в классе скрипта отклоняется |
| `invokedynamic` | проходят только bootstrap'ы `LambdaMetafactory` и `StringConcatFactory` |

Проверка идёт по скомпилированным классам до запуска: нарушение валит загрузку сообщением `script class <name> references blocked symbol: <symbol>`, и ничего не выполняется.

## Бюджеты

| Что | Лимит |
|---|---|
| один вызов обработчика, команды или колбэка | 250 мс |
| верхний уровень файла при загрузке | 5 с |
| падений подряд до автоотключения | 5 |
| отчётов об ошибке на скрипт | один раз в 3000 мс, остальные считаются как `(+N suppressed)` |

Превышение бросает `ScriptTimeoutError`, пишет отчёт и выключает скрипт; дедлайн проверяется на каждом 512-м инструментированном переходе.
Один удачный вызов обнуляет счётчик падений.

## Исключения

| Исключение | Наследует | Когда бросается |
|---|---|---|
| `ScriptException` | `RuntimeException` | плохой аргумент, неизвестная ссылка на модуль/настройку/entry, невалидный id или url |
| `ScriptStateException` | `ScriptException` | нет мира или игрока, сущность ушла из мира, скрипт уже выгружен |
| `ScriptThreadException` | `ScriptException` | вызов, которому нужен клиентский поток, сделан не из него |
| `ScriptApiException` | `ScriptException` | `requireApi(n)` с `n` больше `ApiVersion.CURRENT` |
| `ScriptTimeoutError` | `Error` | кончился бюджет обработчика |

У каждого есть конструктор от `String message`; у `ScriptException` есть ещё `(message, cause)`.
`ScriptTimeoutError` — это `Error`, а не `ScriptException`: ловить его бессмысленно, скрипт всё равно получит отчёт и выключится.

## Лимиты ресурсов

| Ресурс | Лимит |
|---|---|
| ключей в конфиге | 4096, на следующем новом ключе `ScriptException` |
| символов в значении конфига | 65536, выше — `ScriptException` |
| файл в assets | 8 МиБ, выше `bytes()` и `text()` вернут null |
| записей в `assets.list()` | 1024, лишние имена отброшены с одним предупреждением в консоль |
| встроенный шрифт или PNG | 8 МиБ, выше игнорируется с одним предупреждением |
| живых gpu-ресурсов на скрипт | 64 меша, пайплайна и render type'а вместе, `IllegalStateException` |
| атрибутов на формат вершины | 16, `IllegalArgumentException` |
| вершин в меше | 1 048 576, `IllegalStateException` |
| индексов в меше | 4 194 304, `IllegalStateException` |

Буферы мешей освобождаются, пока скрипт выключен; в лимит 64 эти ресурсы считаются до выгрузки скрипта.
