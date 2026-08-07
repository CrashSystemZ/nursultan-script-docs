# Модули клиента

`client.modules()` даёт доступ к встроенным модулям Nursultan: можно прочитать их состояние, включить, выключить и покрутить настройки.

## Найти модуль

```kotlin
client.modules().all()             // все модули
client.modules().exists("AttackAura")
client.modules().find("AttackAura")   // null, если нет
client.modules().get("AttackAura")    // ошибка, если нет
```

`find` возвращает `null`, `get` кидает понятную ошибку — бери то, что удобнее в конкретном месте.

Искать можно и по внутреннему имени, и по тому, как модуль подписан в меню; регистр, пробелы и подчёркивания не важны:

```kotlin
client.modules().get("attack aura")
```

## Что можно с модулем

```kotlin
val aura = client.modules().get("AttackAura")

aura.name()
aura.displayName()
aura.enabled()
aura.setEnabled(true)
aura.toggle()
aura.bind()
```

## Настройки модуля

```kotlin
aura.settings()                    // все настройки списком
aura.setting("range")              // одна, или null
```

Если известен тип — бери сразу типизированно, тогда не придётся приводить:

```kotlin
aura.checkBox("...")
aura.slider("...")
aura.rangeSlider("...")
aura.input("...")
aura.selectable("...")
aura.combo("...")
aura.colorPicker("...")
aura.hotkey("...")
```

Если настройка окажется другого типа, получишь понятную ошибку.

## Ссылка на настройку

В меню по правому клику на настройке можно скопировать её ссылку — она выглядит так:

```
module.attackaura.setting.targets.players
```

Это и есть самый надёжный способ обратиться к настройке:

```kotlin
aura.slider("module.attackaura.setting.range").value(3.5f)
```

Работает и короче — путь внутри модуля или последний сегмент:

```kotlin
aura.slider("range")
aura.combo("targets")
```

Короткие формы удобны, но если у модуля две настройки с похожим именем, надёжнее полная ссылка. Регистр, точки, дефисы и подчёркивания при поиске игнорируются.

## Варианты настроек

Для `selectable` и `combo` варианты — это [entry](../settings/entries.md), и на них тоже есть ссылки:

```kotlin
val targets = aura.combo("targets")

targets.options()                  // имена вариантов
targets.set("players", true)
targets.set(targets.entry("entry.animals"), false)

val mode = aura.selectable("mode")
mode.select("silent")
mode.value().name()
```

## Чего нельзя

Модуль принадлежит клиенту, а не скрипту. Поэтому:

* нельзя подписаться на изменение его настройки (`onChange`);
* нельзя менять его видимость (`visibleWhen`);
* нельзя поменять единицу у его слайдера (`postfix`);
* нельзя привязать логику к его entry.

Всё это доступно только для собственных настроек скрипта.

## Узнать, что что-то переключили

```kotlin
on<ModuleToggleEvent> { e ->
    chat.print(e.name() + " -> " + e.enabled())
}
```

Событие приходит и на модули клиента, и на скрипты; `fromScript()` покажет, кто именно.
