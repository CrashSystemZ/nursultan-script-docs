# Сообщения в группе

`client.party()` — в DSL просто `party` — передаёт небольшие бинарные сообщения между скриптами всех участников твоей группы, а раскладку каждого задаёт форма, которую ты объявляешь сам. Событие доходит до тех, кто онлайн в этот момент; состояние — это последнее значение, которое ты опубликовал под темой, и сервер отдаёт его тем, кто зайдёт позже.

```kotlin
val Appearance = shape("appearance") {
    string(1, "hat")
    int(2, "tint", default = 0xFFFFFF)
}

val cosmetics = party.channel("crashsystem:cosmetics")
val worn = mutableMapOf<String, String>()

cosmetics.onState(Appearance) { message ->
    worn[message.sender().login()] = message.string("hat")
}

cosmetics.onStateCleared(Appearance) { message ->
    worn.remove(message.sender().login())
}

onEnable {
    cosmetics.publish(Appearance, "hat" to "halo")
}
```

## Группа

| Метод | Тип | Описание |
|---|---|---|
| `party.connected()` | `boolean` | у клиента живая сессия сокета (API 4) |
| `party.inParty()` | `boolean` | ты сейчас в группе (API 4) |
| `party.self()` | `PartyMember?` | ты, null вне группы (API 4) |
| `party.leader()` | `PartyMember?` | текущий лидер, null вне группы (API 4) |
| `party.isLeader()` | `boolean` | ты лидер этой группы (API 4) |
| `party.code()` | `String` | код входа, вне группы пустой (API 5) |
| `party.members()` | `List<PartyMember>` | неизменяемый снимок, вместе с тобой (API 4) |
| `party.member(login)` | `PartyMember?` | участник по логину аккаунта, null если такого нет (API 4) |
| `party.channel(namespace)` | `PartyChannel` | открывает канал или возвращает уже открытый (API 4) (бросает `ScriptException` на кривой namespace, на занятый другим загруженным скриптом и после 8 каналов) |
| `party.buffer()` | `PartyPayloadWriter` | пустой writer, чтобы упаковать поле `bytes` вручную (API 4) |
| `party.reader(data)` | `PartyPayloadReader` | reader по полю `bytes`, которое ты упаковал (API 4) (бросает `ScriptException`, если `data` — null) |
| `party.maxEventBytes()` | `int` | 2048 (API 4) |
| `party.maxStateBytes()` | `int` | 1024 (API 4) |
| `party.maxStateTopics()` | `int` | 8 (API 4) |

## Участник

| Метод | Тип | Описание |
|---|---|---|
| `login()` | `String` | логин аккаунта, одинаков на любом сервере и в любом мире (API 4) |
| `name()` | `String` | ник Minecraft, меняется вместе с ником аккаунта (API 4) |
| `leader()` | `boolean` | лидер группы (API 4) |
| `self()` | `boolean` | это ты (API 4) |
| `color()` | `int` | цвет участника от сервера, непрозрачный ARGB, 0 если отправитель неизвестен (API 5) |
| `position()` | `Vec?` | последняя известная позиция, null пока её нет (API 5) |
| `positionAge()` | `long` | мс с той позиции, `0` для тебя самого, -1 если её нет (API 5) |

Позиции приходят дважды в секунду, только по остальным участникам и только пока они онлайн; `position()` интерполирует между двумя последними и останавливается на свежей, а не улетает дальше, а твоя собственная запись отдаёт живую позицию игрока.
Цвет берётся из палитры на десять записей, у каждого участника свой, пока живёт группа, — поэтому двое рисуют одного и того же человека одинаково.

## Форма

| Объявление | Принимает | Описание |
|---|---|---|
| `shape(topic) { … }` | | объявляет форму, её имя и есть тема (API 4) |
| `bool(id, name)` | `Boolean` | один байт varint (API 4) |
| `int(id, name)` | `Int` | zigzag varint, от 1 до 5 байт (API 4) |
| `long(id, name)` | `Long` | zigzag varint, от 1 до 10 байт (API 4) |
| `float(id, name)` | `Float` | 4 байта (API 4) |
| `double(id, name)` | `Double` | 8 байт (API 4) |
| `string(id, name)` | `String` | длина плюс utf-8, не больше 512 байт (API 4) |
| `bytes(id, name)` | `ByteArray` | длина плюс сырой массив (API 4) |
| `uuid(id, name)` | `UUID` | 16 байт (API 4) |
| `vec(id, name)` | `Vec` | три double, 24 байта (API 4) |
| `struct(id, name, shape)` | `PartyStruct` | одна вложенная форма (API 4) |
| `list(id, name, type)` | `List<Any>` | список одного `PartyFieldType` (API 4) |
| `list(id, name, shape)` | `List<PartyStruct>` | список вложенных форм (API 4) |
| `ints(id, name)` | `List<Int>` | список `INT` (API 4) |
| `longs(id, name)` | `List<Long>` | список `LONG` (API 4) |
| `floats(id, name)` | `List<Float>` | список `FLOAT` (API 4) |
| `doubles(id, name)` | `List<Double>` | список `DOUBLE` (API 4) |
| `strings(id, name)` | `List<String>` | список `STRING` (API 4) |
| `vecs(id, name)` | `List<Vec>` | список `VEC` (API 4) |
| `uuids(id, name)` | `List<UUID>` | список `UUID` (API 4) |

Любое объявление, кроме `bytes`, `uuid`, `struct` и списков, принимает ещё и `default = …` — его берут, когда отправитель это поле не прислал. Id с 1 по 15 стоят одного байта на проводе.
На провод уходят только id и тип, поэтому писать и читать можно в разном порядке, а поле, которого получатель не объявил, пропускается по своему wire-типу; перестановка или повторное использование id заставит две версии скрипта читать друг друга неправильно.

### PartyShape

| Метод | Тип | Описание |
|---|---|---|
| `PartyShape.builder(name)` | `PartyShapeBuilder` | начинает форму без DSL (API 4) (бросает `ScriptException`, если имя не ASCII в нижнем регистре или длиннее 32 символов) |
| `shape.name()` | `String` | тема, которую называет эта форма (API 4) |
| `shape.fields()` | `List<PartyField>` | объявленные поля, в порядке объявления (API 4) |
| `shape.field(name)` | `PartyField?` | поле по имени, null если такого нет (API 4) |
| `shape.field(id)` | `PartyField?` | поле по id, null если такого нет (API 4) |
| `shape.capacity()` | `int` | наибольший объявленный id плюс один (API 4) |
| `shape.fieldNames()` | `String` | имена через запятую (API 4) |
| `shape.toString()` | `String` | `name {id:field type, …}` (API 4) |
| `PartyShape.MAX_FIELD_ID` | `int` | 2047 (API 4) |
| `PartyShape.MAX_FIELDS` | `int` | 64 (API 4) |
| `PartyShape.MAX_FIELD_NAME_LENGTH` | `int` | 32 (API 4) |
| `Shape("field" to value, …)` | `PartyStruct` | собирает значение для поля `struct` или списка (API 4) |

### PartyShapeBuilder

| Метод | Тип | Описание |
|---|---|---|
| `boolField(id, field)` | `PartyShapeBuilder` | java-имя за `bool(id, name)` (API 4) |
| `boolField(id, field, default)` | `PartyShapeBuilder` | поле bool со значением по умолчанию (API 4) |
| `intField(id, field)` | `PartyShapeBuilder` | java-имя за `int(id, name)` (API 4) |
| `intField(id, field, default)` | `PartyShapeBuilder` | поле int со значением по умолчанию (API 4) |
| `longField(id, field)` | `PartyShapeBuilder` | java-имя за `long(id, name)` (API 4) |
| `longField(id, field, default)` | `PartyShapeBuilder` | поле long со значением по умолчанию (API 4) |
| `floatField(id, field)` | `PartyShapeBuilder` | java-имя за `float(id, name)` (API 4) |
| `floatField(id, field, default)` | `PartyShapeBuilder` | поле float со значением по умолчанию (API 4) |
| `doubleField(id, field)` | `PartyShapeBuilder` | java-имя за `double(id, name)` (API 4) |
| `doubleField(id, field, default)` | `PartyShapeBuilder` | поле double со значением по умолчанию (API 4) |
| `stringField(id, field)` | `PartyShapeBuilder` | java-имя за `string(id, name)` (API 4) |
| `stringField(id, field, default)` | `PartyShapeBuilder` | поле string со значением по умолчанию (API 4) |
| `bytesField(id, field)` | `PartyShapeBuilder` | java-имя за `bytes(id, name)`, без формы с default (API 4) |
| `uuidField(id, field)` | `PartyShapeBuilder` | java-имя за `uuid(id, name)`, без формы с default (API 4) |
| `vecField(id, field)` | `PartyShapeBuilder` | java-имя за `vec(id, name)` (API 4) |
| `vecField(id, field, default)` | `PartyShapeBuilder` | поле vec со значением по умолчанию (API 4) |
| `structField(id, field, shape)` | `PartyShapeBuilder` | java-имя за `struct(id, name, shape)` (API 4) (бросает `ScriptException`, если `shape` — null) |
| `listField(id, field, type)` | `PartyShapeBuilder` | список одного `PartyFieldType` (API 4) (бросает `ScriptException`, если тип `LIST` или `STRUCT`) |
| `listField(id, field, shape)` | `PartyShapeBuilder` | список вложенных форм (API 4) (бросает `ScriptException`, если `shape` — null) |
| `build()` | `PartyShape` | завершает форму, этим и заканчивается `shape(topic) { … }` (API 4) (бросает `ScriptException`, если не объявлено ни одного поля) |

Каждый метод возвращает билдер, а любое объявление поля бросает `ScriptException`, если id вне `1..2047`, имя поля не ASCII или длиннее 32 символов, в форме уже 64 поля или имя либо id заняты.

### PartyField

| Метод | Тип | Описание |
|---|---|---|
| `field.id()` | `int` | id на проводе, `1..2047` (API 4) |
| `field.name()` | `String` | имя поля, на провод не уходит (API 4) |
| `field.type()` | `PartyFieldType` | объявленный тип (API 4) |
| `field.element()` | `PartyFieldType?` | тип элемента списка, null если поле не список (API 4) |
| `field.shape()` | `PartyShape?` | вложенная форма, null если это не struct и не список struct (API 4) |
| `field.hasDefault()` | `boolean` | значение по умолчанию объявлено (API 4) |
| `field.defaultValue()` | `Object?` | объявленное значение по умолчанию, null если его нет (API 4) |
| `field.typeLabel()` | `String` | `list of vec`, имя вложенной формы или сам тип (API 4) |
| `field.toString()` | `String` | `name (id N, type)` (API 4) |

### PartyFieldType

| Значение | Тип | Описание |
|---|---|---|
| `BOOL` | `PartyFieldType` | varint, 0 или 1 (API 4) |
| `INT` | `PartyFieldType` | zigzag varint (API 4) |
| `LONG` | `PartyFieldType` | zigzag varint (API 4) |
| `FLOAT` | `PartyFieldType` | фиксированные 4 байта (API 4) |
| `DOUBLE` | `PartyFieldType` | фиксированные 8 байт (API 4) |
| `STRING` | `PartyFieldType` | длина плюс utf-8, не больше 512 байт (API 4) |
| `BYTES` | `PartyFieldType` | длина плюс сырые байты (API 4) |
| `UUID` | `PartyFieldType` | длина плюс 16 байт (API 4) |
| `VEC` | `PartyFieldType` | длина плюс три double (API 4) |
| `STRUCT` | `PartyFieldType` | длина плюс вложенная форма (API 4) |
| `LIST` | `PartyFieldType` | длина плюс счётчик и элементы (API 4) |
| `type.wire()` | `PartyWire` | wire-тип, в который он ложится (API 4) |
| `type.label()` | `String` | имя константы в нижнем регистре (API 4) |

### PartyWire

| Значение | Тип | Описание |
|---|---|---|
| `VARINT` | `PartyWire` | код 0 (API 4) |
| `FIXED64` | `PartyWire` | код 1 (API 4) |
| `LENGTH` | `PartyWire` | код 2, с префиксом длины (API 4) |
| `FIXED32` | `PartyWire` | код 5 (API 4) |
| `wire.code()` | `int` | 3-битный код, упакованный в тег поля (API 4) |

### PartyStruct

| Метод | Тип | Описание |
|---|---|---|
| `PartyStruct(shape)` | `PartyStruct` | пустое значение этой формы (API 4) (бросает `ScriptException`, если `shape` — null) |
| `struct.set(name, value)` | `PartyStruct` | выставляет одно поле (API 4) (бросает `ScriptException` на неизвестном поле или значении null) |
| `struct.shape()` | `PartyShape` | форма, к которой относится значение (API 4) |
| `struct.values()` | `Map<String, Object>` | выставленные поля, в порядке вставки (API 4) |

## Канал

| Метод | Тип | Описание |
|---|---|---|
| `channel.namespace()` | `String` | namespace, на котором открыт канал (API 4) |
| `channel.send(shape, "field" to value, …)` | `PartySendResult` | шлёт событие всей группе (API 4) |
| `channel.send(shape, …, target = target)` | `PartySendResult` | то же, одному адресату (API 4) |
| `channel.send(shape, target) { set(name, value) }` | `PartySendResult` | то же событие формой блоком (API 4) |
| `channel.publish(shape, "field" to value, …)` | `PartySendResult` | публикует состояние под темой формы (API 4) |
| `channel.publish(shape) { set(name, value) }` | `PartySendResult` | та же публикация формой блоком (API 4) |
| `channel.clearState(shape)` | `PartySendResult` | убирает это состояние у всех участников и на сервере (API 4) |
| `channel.clearStates()` | `void` | убирает все состояния, опубликованные этим каналом (API 4) |
| `channel.publishedStates()` | `List<String>` | темы, по которым канал сейчас держит состояние (API 4) |
| `channel.onEvent(shape, handler)` | `Subscription` | обработчик событий по теме этой формы (API 4) |
| `channel.onEvent(shape, rule, handler)` | `Subscription` | то же, с фильтром `SenderRule` (API 4) |
| `channel.onState(shape, handler)` | `Subscription` | обработчик состояния, заодно проигрывает последнее известное значение каждого отправителя (API 4) |
| `channel.onState(shape, rule, handler)` | `Subscription` | то же, с фильтром `SenderRule` (API 4) |
| `channel.onStateCleared(shape, handler)` | `Subscription` | обработчик снятого состояния (API 4) |
| `channel.onStateCleared(shape, rule, handler)` | `Subscription` | то же, с фильтром `SenderRule` (API 4) |

Namespace — это `author:feature`: ASCII в нижнем регистре `[a-z0-9][a-z0-9._-]*` по обе стороны двоеточия, не длиннее 48 символов, обе половины непустые. Namespace одновременно держит один загруженный скрипт.
Обработчики идут в клиентском потоке по одному сообщению за раз, в той же обработке ошибок и в том же бюджете CPU, что и любой другой обработчик скрипта, и только пока скрипт загружен и включён.

### PartyChannel

| Метод | Тип | Описание |
|---|---|---|
| `channel.sendEvent(shape, body)` | `PartySendResult` | java-имя за `send(shape, …)` (API 4) |
| `channel.sendEvent(shape, target, body)` | `PartySendResult` | java-имя за `send(shape, …, target = …)` (API 4) |
| `channel.publishState(shape, body)` | `PartySendResult` | java-имя за `publish(shape, …)` (API 4) |
| `channel.clearState(topic)` | `PartySendResult` | форма по теме, для формы, которую ты не объявлял (API 4) (бросает `ScriptException` на кривой теме) |
| `channel.onStateCleared(topic, handler)` | `Subscription` | форма по теме для обработчика снятого состояния (API 4) |
| `channel.onStateCleared(topic, rule, handler)` | `Subscription` | то же, с фильтром `SenderRule` (API 4) |

### PartyShapedWriter

| Метод | Тип | Описание |
|---|---|---|
| `writer.shape()` | `PartyShape` | форма, которую пишут (API 4) |
| `writer.set(name, value)` | `PartyShapedWriter` | пишет одно поле (API 4) (бросает `ScriptException` на неизвестном поле, значении null, не том типе, дважды выставленном поле и на payload, который больше не влезает) |
| `writer.size()` | `int` | записано байт, включая байт маркера формата (API 4) |
| `writer.remaining()` | `int` | сколько байт осталось до лимита (API 4) |
| `writer.limit()` | `int` | 2048 для события, 1024 для состояния (API 4) |

Writer — это получатель блока `{ set(name, value) }`. Порядок вызовов `set` значения не имеет, а невыставленное поле на провод не уходит, поэтому получатель прочитает своё объявленное значение по умолчанию.

## Кому уйдёт

| Метод | Тип | Описание |
|---|---|---|
| `PartyTarget.all()` | `PartyTarget` | всем остальным участникам онлайн; свою же рассылку ты не получаешь (API 4) |
| `PartyTarget.leader()` | `PartyTarget` | только текущему лидеру, никому если лидер — ты (API 4) |
| `PartyTarget.member(login)` | `PartyTarget` | одному участнику по логину аккаунта (API 4) |
| `PartyTarget.member(member)` | `PartyTarget` | одному участнику по `PartyMember`, на null-участнике отправка даст `INVALID_TARGET` (API 4) |
| `target.kind()` | `PartyTargetKind` | `ALL`, `LEADER` или `MEMBER` (API 4) |
| `target.login()` | `String?` | логин для `MEMBER`, иначе null (API 4) |
| `target.toString()` | `String` | `all`, `leader` или `member(login)` (API 4) |

Состояние всегда общегрупповое: `publish` и `clearState` адресата не принимают.

### PartyTargetKind

| Значение | Тип | Описание |
|---|---|---|
| `ALL` | `PartyTargetKind` | всем остальным участникам (API 4) |
| `LEADER` | `PartyTargetKind` | только лидеру (API 4) |
| `MEMBER` | `PartyTargetKind` | одному названному участнику (API 4) |

## Фильтр по отправителю

| Значение | Тип | Описание |
|---|---|---|
| `SenderRule.ANY_MEMBER` | `SenderRule` | по умолчанию, любой участник твоей группы (API 4) |
| `SenderRule.LEADER_ONLY` | `SenderRule` | только то, что сервер пометил как присланное лидером (API 4) |

## Результат отправки

| Значение | Тип | Описание |
|---|---|---|
| `SENT` | `PartySendResult` | отдано в сокет (API 4) |
| `COALESCED` | `PartySendResult` | заменило публикацию состояния, которая ещё не ушла (API 4) |
| `NOT_CONNECTED` | `PartySendResult` | сессии сокета нет; публикация состояния запомнена и уйдёт после переподключения (API 4) |
| `NOT_IN_PARTY` | `PartySendResult` | ты не в группе; публикация состояния запомнена и уйдёт после входа (API 4) |
| `INVALID_TARGET` | `PartySendResult` | логин адресата не участник текущей группы (API 4) |
| `TOO_LARGE` | `PartySendResult` | payload больше лимита своего вида (API 4) |
| `STATE_LIMIT` | `PartySendResult` | у тебя уже 8 тем состояния (API 4) |
| `RATE_LIMITED` | `PartySendResult` | клиентский лимитер отбросил сообщение (API 4) |
| `result.ok()` | `boolean` | true для `SENT` и `COALESCED` (API 4) |

## Сообщение

| Метод | Тип | Описание |
|---|---|---|
| `sender()` | `PartyMember` | проставляет сервер, а не payload (API 4) |
| `namespace()` | `String` | namespace, на который пришло (API 4) |
| `topic()` | `String` | тема, на которую пришло (API 4) |
| `kind()` | `PartyMessageKind` | `EVENT`, `STATE_SET` или `STATE_CLEAR` (API 4) |

У полученного сообщения `sender().self()` всегда false. Обработчик `onEvent` или `onState` получает `PartyShapedMessage` — эту таблицу плюс разобранные поля ниже; обработчику `onStateCleared` достаётся только эта таблица.

### PartyMessageKind

| Значение | Тип | Описание |
|---|---|---|
| `EVENT` | `PartyMessageKind` | одноразовое, нигде не хранится (API 4) |
| `STATE_SET` | `PartyMessageKind` | последнее значение по теме (API 4) |
| `STATE_CLEAR` | `PartyMessageKind` | сохранённое состояние сняли (API 4) |

## Чтение полей

| Метод | Тип | Описание |
|---|---|---|
| `shape()` | `PartyShape` | форма, которой его разобрали (API 4) |
| `has(name)` | `boolean` | отправитель действительно прислал это поле (API 4) |
| `bool(name)` | `boolean` | (API 4) |
| `int(name)` | `int` | (API 4) |
| `long(name)` | `long` | (API 4) |
| `float(name)` | `float` | (API 4) |
| `double(name)` | `double` | (API 4) |
| `string(name)` | `String` | (API 4) |
| `bytes(name)` | `ByteArray` | новый массив (API 4) |
| `uuid(name)` | `UUID` | (API 4) |
| `vec(name)` | `Vec` | (API 4) |
| `struct(name)` | `PartyFields` | одна вложенная форма, читается этими же методами (API 4) |
| `structs(name)` | `List<PartyFields>` | список вложенных форм (API 4) |
| `ints(name)` | `List<Int>` | список int (API 4) |
| `floats(name)` | `List<Float>` | список float (API 4) |
| `doubles(name)` | `List<Double>` | список double (API 4) |
| `strings(name)` | `List<String>` | список string (API 4) |
| `vecs(name)` | `List<Vec>` | список vec (API 4) |
| `dump()` | `String` | все поля одной строкой, значения по умолчанию помечены (API 4) |

Поле, которое отправитель не прислал, отдаёт объявленное значение по умолчанию; без него бросает `ScriptException` с именем поля. Запрос не того типа тоже бросает, и имя, которого нет в форме, — тоже.

### PartyFields

| Метод | Тип | Описание |
|---|---|---|
| `booleanValue(name)` | `boolean` | java-имя за `bool(name)` (API 4) |
| `intValue(name)` | `int` | java-имя за `int(name)` (API 4) |
| `longValue(name)` | `long` | java-имя за `long(name)` (API 4) |
| `floatValue(name)` | `float` | java-имя за `float(name)` (API 4) |
| `doubleValue(name)` | `double` | java-имя за `double(name)` (API 4) |
| `values(name)` | `List<Object>` | список в боксах, за ним стоят `ints` `floats` `doubles` `strings` `vecs` `structs` (API 4) |

## Своя упаковка байт

| Метод | Тип | Описание |
|---|---|---|
| `writeBoolean(value)` | `PartyPayloadWriter` | 1 байт (API 4) |
| `writeByte(value)` | `PartyPayloadWriter` | 1 байт, младшие 8 бит int (API 4) |
| `writeShort(value)` | `PartyPayloadWriter` | 2 байта, big endian (API 4) |
| `writeInt(value)` | `PartyPayloadWriter` | 4 байта, big endian (API 4) |
| `writeLong(value)` | `PartyPayloadWriter` | 8 байт, big endian (API 4) |
| `writeFloat(value)` | `PartyPayloadWriter` | 4 байта (API 4) |
| `writeDouble(value)` | `PartyPayloadWriter` | 8 байт (API 4) |
| `writeString(value)` | `PartyPayloadWriter` | 2 байта длины плюс utf-8, не больше 512 байт (API 4) (бросает `ScriptException` на null и на строке длиннее) |
| `writeBytes(value)` | `PartyPayloadWriter` | 2 байта длины плюс копия массива (API 4) (бросает `ScriptException` на null) |
| `writeUuid(value)` | `PartyPayloadWriter` | 16 байт (API 4) (бросает `ScriptException` на null) |
| `toByteArray()` | `ByteArray` | то, что написал, готово для поля `bytes` (API 4) |
| `size()` | `int` | сколько байт записано (API 4) |
| `remaining()` | `int` | сколько байт осталось до лимита (API 4) |
| `limit()` | `int` | 2048 у `party.buffer()` (API 4) |

Поле `bytes` несёт что положишь: `party.buffer()` даёт writer, `party.reader(data)` читает обратно, и оба позиционные, поэтому порядок чтения должен совпадать с порядком записи. Запись за лимит бросает `ScriptException`.

### PartyPayloadReader

| Метод | Тип | Описание |
|---|---|---|
| `readBoolean()` | `boolean` | 1 байт, ненулевой — true (API 4) |
| `readByte()` | `byte` | один знаковый байт (API 4) |
| `readShort()` | `short` | 2 байта, big endian (API 4) |
| `readInt()` | `int` | 4 байта, big endian (API 4) |
| `readLong()` | `long` | 8 байт, big endian (API 4) |
| `readFloat()` | `float` | 4 байта (API 4) |
| `readDouble()` | `double` | 8 байт (API 4) |
| `readString()` | `String` | 2 байта длины плюс utf-8 (API 4) |
| `readBytes()` | `ByteArray` | 2 байта длины плюс сами байты (API 4) |
| `readUuid()` | `UUID` | 16 байт (API 4) |
| `size()` | `int` | длина payload (API 4) |
| `remaining()` | `int` | сколько байт ещё не прочитано (API 4) |
| `hasMore()` | `boolean` | осталось ли что читать (API 4) |

Чтение за концом бросает `ScriptException`; payload — обычный массив, поэтому обрезанное или перепутанное чтение вернёт мусор, а не ошибку.

## Лимиты

| Лимит | Значение | Описание |
|---|---|---|
| Payload события | `2048` | байт на сообщение |
| Payload состояния | `1024` | байт на тему |
| Строка внутри payload | `512` | байт, и она входит в бюджет payload |
| Полей в форме | `64` | |
| Id поля | `1..2047` | с 1 по 15 занимают на проводе один байт |
| Имя поля | `32` | символа ASCII |
| Темы состояния | `8` | на аккаунт, по всем твоим скриптам |
| Namespace | `48` | символов ASCII |
| Тема | `32` | символа ASCII, имя формы и есть тема |
| Каналы | `8` | на скрипт |
| Время жизни состояния | `5 мин` | с последней публикации, если не опубликовать заново |

Отправки сверх клиентского лимитера возвращают `RATE_LIMITED`; у сервера свои, более жёсткие бюджеты, и всё сверх них он отбрасывает молча. Публикации состояния схлопываются по теме, поэтому публикация каждый тик стоит одного пакета раз в несколько тиков и уходит только самое свежее значение.

## Что решает сервер

| Факт | Описание |
|---|---|
| Payload | непрозрачные байты; сервер не читает поля и не знает твоей формы |
| Отправитель | берётся из авторизованной сессии, поэтому payload не может выдать себя за другого |
| Группа | берётся из текущей группы отправителя; сообщение за её пределы не выходит |
| Лидер | берётся из группы, поэтому `SenderRule.LEADER_ONLY` не подделать |
| Получатели | берутся из группы; клиент список получателей не присылает |
| Снимок состояний | уходит участнику при входе, события заново не проигрываются |
| Очистка состояний | при выходе, кике, роспуске, отключении и по времени жизни выше |

Namespace — это имя маршрута, а не удостоверение: он ничего не доказывает о том, какой скрипт прислал сообщение, только о том, какой аккаунт.
