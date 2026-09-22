# Party messages

`client.party()` — in the DSL just `party` — carries small binary messages between the scripts of everyone in your party, laid out by a shape you declare. An event reaches whoever is online at that moment; a state is the last value you published under a topic, and the server hands it to whoever joins later.

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

## The party

| Method | Type | Description |
|---|---|---|
| `party.connected()` | `boolean` | the client has a live socket session (API 4) |
| `party.inParty()` | `boolean` | you are in a party right now (API 4) |
| `party.self()` | `PartyMember?` | you, null outside a party (API 4) |
| `party.leader()` | `PartyMember?` | current leader, null outside a party (API 4) |
| `party.isLeader()` | `boolean` | you lead this party (API 4) |
| `party.code()` | `String` | the join code, empty outside a party (API 5) |
| `party.members()` | `List<PartyMember>` | immutable snapshot, you included (API 4) |
| `party.member(login)` | `PartyMember?` | member by account login, null when absent (API 4) |
| `party.channel(namespace)` | `PartyChannel` | opens the channel, or returns the one already open (API 4) (throws `ScriptException` on a malformed namespace, on one another loaded script holds, or past 8 channels) |
| `party.buffer()` | `PartyPayloadWriter` | empty writer for packing a `bytes` field yourself (API 4) |
| `party.reader(data)` | `PartyPayloadReader` | reader over a `bytes` field you packed (API 4) (throws `ScriptException` when `data` is null) |
| `party.maxEventBytes()` | `int` | 2048 (API 4) |
| `party.maxStateBytes()` | `int` | 1024 (API 4) |
| `party.maxStateTopics()` | `int` | 8 (API 4) |

## A member

| Method | Type | Description |
|---|---|---|
| `login()` | `String` | account login, stable across servers and worlds (API 4) |
| `name()` | `String` | Minecraft name, changes with the account's nickname (API 4) |
| `leader()` | `boolean` | leads the party (API 4) |
| `self()` | `boolean` | this member is you (API 4) |
| `color()` | `int` | server-assigned member colour, opaque ARGB, 0 when the sender is unknown (API 5) |
| `position()` | `Vec?` | last known position, null until one arrives (API 5) |
| `positionAge()` | `long` | ms since that position, `0` for yourself, -1 when there is none (API 5) |

Positions arrive twice a second, only for the other members and only while they are online; `position()` interpolates between the last two and stops at the newest instead of running past it, and your own entry returns the live player position.
The colour comes from a ten entry palette, distinct per member for as long as the party lives, so two members draw the same person the same way.

## A shape

| Declaration | Takes | Description |
|---|---|---|
| `shape(topic) { … }` | | declares the shape, the name is the topic (API 4) |
| `bool(id, name)` | `Boolean` | one varint byte (API 4) |
| `int(id, name)` | `Int` | zigzag varint, 1 to 5 bytes (API 4) |
| `long(id, name)` | `Long` | zigzag varint, 1 to 10 bytes (API 4) |
| `float(id, name)` | `Float` | 4 bytes (API 4) |
| `double(id, name)` | `Double` | 8 bytes (API 4) |
| `string(id, name)` | `String` | length plus utf-8, at most 512 bytes (API 4) |
| `bytes(id, name)` | `ByteArray` | length plus the raw array (API 4) |
| `uuid(id, name)` | `UUID` | 16 bytes (API 4) |
| `vec(id, name)` | `Vec` | three doubles, 24 bytes (API 4) |
| `struct(id, name, shape)` | `PartyStruct` | one nested shape (API 4) |
| `list(id, name, type)` | `List<Any>` | a list of one `PartyFieldType` (API 4) |
| `list(id, name, shape)` | `List<PartyStruct>` | a list of nested shapes (API 4) |
| `ints(id, name)` | `List<Int>` | a list of `INT` (API 4) |
| `longs(id, name)` | `List<Long>` | a list of `LONG` (API 4) |
| `floats(id, name)` | `List<Float>` | a list of `FLOAT` (API 4) |
| `doubles(id, name)` | `List<Double>` | a list of `DOUBLE` (API 4) |
| `strings(id, name)` | `List<String>` | a list of `STRING` (API 4) |
| `vecs(id, name)` | `List<Vec>` | a list of `VEC` (API 4) |
| `uuids(id, name)` | `List<UUID>` | a list of `UUID` (API 4) |

Every declaration except `bytes`, `uuid`, `struct` and the list forms also takes `default = …`, used when the sender did not send that field. Ids 1 to 15 cost one byte on the wire.
Only the id and the type go on the wire, so writing and reading in different orders is fine and a field the receiver does not declare is skipped by its wire type; reusing or reordering an id makes two versions of a script read each other wrong.

### PartyShape

| Method | Type | Description |
|---|---|---|
| `PartyShape.builder(name)` | `PartyShapeBuilder` | starts a shape without the DSL (API 4) (throws `ScriptException` when the name is not lowercase ASCII or is over 32 characters) |
| `shape.name()` | `String` | the topic this shape names (API 4) |
| `shape.fields()` | `List<PartyField>` | declared fields, in declaration order (API 4) |
| `shape.field(name)` | `PartyField?` | field by name, null when absent (API 4) |
| `shape.field(id)` | `PartyField?` | field by id, null when absent (API 4) |
| `shape.capacity()` | `int` | highest declared id plus one (API 4) |
| `shape.fieldNames()` | `String` | names joined by a comma (API 4) |
| `shape.toString()` | `String` | `name {id:field type, …}` (API 4) |
| `PartyShape.MAX_FIELD_ID` | `int` | 2047 (API 4) |
| `PartyShape.MAX_FIELDS` | `int` | 64 (API 4) |
| `PartyShape.MAX_FIELD_NAME_LENGTH` | `int` | 32 (API 4) |
| `Shape("field" to value, …)` | `PartyStruct` | builds a value for a `struct` or list field (API 4) |

### PartyShapeBuilder

| Method | Type | Description |
|---|---|---|
| `boolField(id, field)` | `PartyShapeBuilder` | Java name behind `bool(id, name)` (API 4) |
| `boolField(id, field, default)` | `PartyShapeBuilder` | bool field with a default (API 4) |
| `intField(id, field)` | `PartyShapeBuilder` | Java name behind `int(id, name)` (API 4) |
| `intField(id, field, default)` | `PartyShapeBuilder` | int field with a default (API 4) |
| `longField(id, field)` | `PartyShapeBuilder` | Java name behind `long(id, name)` (API 4) |
| `longField(id, field, default)` | `PartyShapeBuilder` | long field with a default (API 4) |
| `floatField(id, field)` | `PartyShapeBuilder` | Java name behind `float(id, name)` (API 4) |
| `floatField(id, field, default)` | `PartyShapeBuilder` | float field with a default (API 4) |
| `doubleField(id, field)` | `PartyShapeBuilder` | Java name behind `double(id, name)` (API 4) |
| `doubleField(id, field, default)` | `PartyShapeBuilder` | double field with a default (API 4) |
| `stringField(id, field)` | `PartyShapeBuilder` | Java name behind `string(id, name)` (API 4) |
| `stringField(id, field, default)` | `PartyShapeBuilder` | string field with a default (API 4) |
| `bytesField(id, field)` | `PartyShapeBuilder` | Java name behind `bytes(id, name)`, no default form (API 4) |
| `uuidField(id, field)` | `PartyShapeBuilder` | Java name behind `uuid(id, name)`, no default form (API 4) |
| `vecField(id, field)` | `PartyShapeBuilder` | Java name behind `vec(id, name)` (API 4) |
| `vecField(id, field, default)` | `PartyShapeBuilder` | vec field with a default (API 4) |
| `structField(id, field, shape)` | `PartyShapeBuilder` | Java name behind `struct(id, name, shape)` (API 4) (throws `ScriptException` when `shape` is null) |
| `listField(id, field, type)` | `PartyShapeBuilder` | list of one `PartyFieldType` (API 4) (throws `ScriptException` when the type is `LIST` or `STRUCT`) |
| `listField(id, field, shape)` | `PartyShapeBuilder` | list of nested shapes (API 4) (throws `ScriptException` when `shape` is null) |
| `build()` | `PartyShape` | finishes the shape, what `shape(topic) { … }` calls last (API 4) (throws `ScriptException` when no field was declared) |

Every method returns the builder, and every field declaration throws `ScriptException` when the id is outside `1..2047`, the field name is not ASCII or is over 32 characters, the shape already holds 64 fields, or the name or the id is taken.

### PartyField

| Method | Type | Description |
|---|---|---|
| `field.id()` | `int` | wire id, `1..2047` (API 4) |
| `field.name()` | `String` | field name, never sent on the wire (API 4) |
| `field.type()` | `PartyFieldType` | declared type (API 4) |
| `field.element()` | `PartyFieldType?` | list element type, null unless the field is a list (API 4) |
| `field.shape()` | `PartyShape?` | nested shape, null unless struct or list of structs (API 4) |
| `field.hasDefault()` | `boolean` | a default was declared (API 4) |
| `field.defaultValue()` | `Object?` | the declared default, null when there is none (API 4) |
| `field.typeLabel()` | `String` | `list of vec`, the nested shape name, or the type (API 4) |
| `field.toString()` | `String` | `name (id N, type)` (API 4) |

### PartyFieldType

| Value | Type | Description |
|---|---|---|
| `BOOL` | `PartyFieldType` | varint, 0 or 1 (API 4) |
| `INT` | `PartyFieldType` | zigzag varint (API 4) |
| `LONG` | `PartyFieldType` | zigzag varint (API 4) |
| `FLOAT` | `PartyFieldType` | fixed 4 bytes (API 4) |
| `DOUBLE` | `PartyFieldType` | fixed 8 bytes (API 4) |
| `STRING` | `PartyFieldType` | length plus utf-8, at most 512 bytes (API 4) |
| `BYTES` | `PartyFieldType` | length plus the raw bytes (API 4) |
| `UUID` | `PartyFieldType` | length plus 16 bytes (API 4) |
| `VEC` | `PartyFieldType` | length plus three doubles (API 4) |
| `STRUCT` | `PartyFieldType` | length plus a nested shape (API 4) |
| `LIST` | `PartyFieldType` | length plus a count and the elements (API 4) |
| `type.wire()` | `PartyWire` | the wire type this one maps to (API 4) |
| `type.label()` | `String` | lowercase constant name (API 4) |

### PartyWire

| Value | Type | Description |
|---|---|---|
| `VARINT` | `PartyWire` | code 0 (API 4) |
| `FIXED64` | `PartyWire` | code 1 (API 4) |
| `LENGTH` | `PartyWire` | code 2, length prefixed (API 4) |
| `FIXED32` | `PartyWire` | code 5 (API 4) |
| `wire.code()` | `int` | the 3-bit code packed into a field tag (API 4) |

### PartyStruct

| Method | Type | Description |
|---|---|---|
| `PartyStruct(shape)` | `PartyStruct` | empty value of that shape (API 4) (throws `ScriptException` when `shape` is null) |
| `struct.set(name, value)` | `PartyStruct` | sets one field (API 4) (throws `ScriptException` on an unknown field or a null value) |
| `struct.shape()` | `PartyShape` | the shape this value belongs to (API 4) |
| `struct.values()` | `Map<String, Object>` | the fields set so far, in insertion order (API 4) |

## A channel

| Method | Type | Description |
|---|---|---|
| `channel.namespace()` | `String` | namespace this channel was opened on (API 4) |
| `channel.send(shape, "field" to value, …)` | `PartySendResult` | sends an event to the whole party (API 4) |
| `channel.send(shape, …, target = target)` | `PartySendResult` | same, to one target (API 4) |
| `channel.send(shape, target) { set(name, value) }` | `PartySendResult` | block form of the same event (API 4) |
| `channel.publish(shape, "field" to value, …)` | `PartySendResult` | publishes state under the shape's topic (API 4) |
| `channel.publish(shape) { set(name, value) }` | `PartySendResult` | block form of the same publish (API 4) |
| `channel.clearState(shape)` | `PartySendResult` | drops that state on every member and on the server (API 4) |
| `channel.clearStates()` | `void` | clears every state this channel published (API 4) |
| `channel.publishedStates()` | `List<String>` | topics this channel currently holds state on (API 4) |
| `channel.onEvent(shape, handler)` | `Subscription` | handler for events on that shape's topic (API 4) |
| `channel.onEvent(shape, rule, handler)` | `Subscription` | same, filtered by `SenderRule` (API 4) |
| `channel.onState(shape, handler)` | `Subscription` | handler for state, also replays every sender's last known value (API 4) |
| `channel.onState(shape, rule, handler)` | `Subscription` | same, filtered by `SenderRule` (API 4) |
| `channel.onStateCleared(shape, handler)` | `Subscription` | handler for a cleared state (API 4) |
| `channel.onStateCleared(shape, rule, handler)` | `Subscription` | same, filtered by `SenderRule` (API 4) |

A namespace is `author:feature`: lowercase ASCII `[a-z0-9][a-z0-9._-]*` on both sides of the colon, at most 48 characters, no empty half. One loaded script holds a namespace at a time.
Handlers run on the client thread, one message at a time, inside the same error handling and CPU budget as any other script handler, and only while the script is loaded and enabled.

### PartyChannel

| Method | Type | Description |
|---|---|---|
| `channel.sendEvent(shape, body)` | `PartySendResult` | Java name behind `send(shape, …)` (API 4) |
| `channel.sendEvent(shape, target, body)` | `PartySendResult` | Java name behind `send(shape, …, target = …)` (API 4) |
| `channel.publishState(shape, body)` | `PartySendResult` | Java name behind `publish(shape, …)` (API 4) |
| `channel.clearState(topic)` | `PartySendResult` | topic form, for a shape you do not declare (API 4) (throws `ScriptException` on a malformed topic) |
| `channel.onStateCleared(topic, handler)` | `Subscription` | topic form of the cleared-state handler (API 4) |
| `channel.onStateCleared(topic, rule, handler)` | `Subscription` | same, filtered by `SenderRule` (API 4) |

### PartyShapedWriter

| Method | Type | Description |
|---|---|---|
| `writer.shape()` | `PartyShape` | the shape being written (API 4) |
| `writer.set(name, value)` | `PartyShapedWriter` | writes one field (API 4) (throws `ScriptException` on an unknown field, a null value, a wrong type, a field set twice, or a payload that no longer fits) |
| `writer.size()` | `int` | bytes written, including the 1-byte format marker (API 4) |
| `writer.remaining()` | `int` | bytes left before the limit (API 4) |
| `writer.limit()` | `int` | 2048 for an event, 1024 for state (API 4) |

The writer is the receiver of the `{ set(name, value) }` block. The order of `set` calls does not matter, and a field left unset never goes on the wire, so the receiver reads its declared default.

## Choosing who receives

| Method | Type | Description |
|---|---|---|
| `PartyTarget.all()` | `PartyTarget` | every other online member; you never receive your own broadcast (API 4) |
| `PartyTarget.leader()` | `PartyTarget` | the current leader only, nobody when you are the leader (API 4) |
| `PartyTarget.member(login)` | `PartyTarget` | one member by account login (API 4) |
| `PartyTarget.member(member)` | `PartyTarget` | one member by `PartyMember`, a null member sends `INVALID_TARGET` (API 4) |
| `target.kind()` | `PartyTargetKind` | `ALL`, `LEADER` or `MEMBER` (API 4) |
| `target.login()` | `String?` | login for `MEMBER`, null otherwise (API 4) |
| `target.toString()` | `String` | `all`, `leader` or `member(login)` (API 4) |

State is always party-wide: `publish` and `clearState` take no target.

### PartyTargetKind

| Value | Type | Description |
|---|---|---|
| `ALL` | `PartyTargetKind` | every other member (API 4) |
| `LEADER` | `PartyTargetKind` | the leader only (API 4) |
| `MEMBER` | `PartyTargetKind` | one named member (API 4) |

## Filtering who sent it

| Value | Type | Description |
|---|---|---|
| `SenderRule.ANY_MEMBER` | `SenderRule` | default, any member of your party (API 4) |
| `SenderRule.LEADER_ONLY` | `SenderRule` | only what the server marked as sent by the leader (API 4) |

## The result of a send

| Value | Type | Description |
|---|---|---|
| `SENT` | `PartySendResult` | handed to the socket (API 4) |
| `COALESCED` | `PartySendResult` | replaced a state publish that had not gone out yet (API 4) |
| `NOT_CONNECTED` | `PartySendResult` | no socket session; a state publish is remembered and republished on reconnect (API 4) |
| `NOT_IN_PARTY` | `PartySendResult` | you are in no party; a state publish is remembered and republished on join (API 4) |
| `INVALID_TARGET` | `PartySendResult` | the target login is not a current party member (API 4) |
| `TOO_LARGE` | `PartySendResult` | payload over the limit for its kind (API 4) |
| `STATE_LIMIT` | `PartySendResult` | you already hold 8 state topics (API 4) |
| `RATE_LIMITED` | `PartySendResult` | the client rate limiter dropped it (API 4) |
| `result.ok()` | `boolean` | true for `SENT` and `COALESCED` (API 4) |

## The message

| Method | Type | Description |
|---|---|---|
| `sender()` | `PartyMember` | filled in by the server, never by the payload (API 4) |
| `namespace()` | `String` | namespace it arrived on (API 4) |
| `topic()` | `String` | topic it arrived on (API 4) |
| `kind()` | `PartyMessageKind` | `EVENT`, `STATE_SET` or `STATE_CLEAR` (API 4) |

`sender().self()` is always false on a received message. An `onEvent` or `onState` handler receives a `PartyShapedMessage`, which is this table plus the decoded fields below; an `onStateCleared` handler receives this table only.

### PartyMessageKind

| Value | Type | Description |
|---|---|---|
| `EVENT` | `PartyMessageKind` | one-shot, never stored (API 4) |
| `STATE_SET` | `PartyMessageKind` | the last value for a topic (API 4) |
| `STATE_CLEAR` | `PartyMessageKind` | a stored state was dropped (API 4) |

## Reading fields

| Method | Type | Description |
|---|---|---|
| `shape()` | `PartyShape` | the shape it was decoded with (API 4) |
| `has(name)` | `boolean` | the sender actually sent this field (API 4) |
| `bool(name)` | `boolean` | (API 4) |
| `int(name)` | `int` | (API 4) |
| `long(name)` | `long` | (API 4) |
| `float(name)` | `float` | (API 4) |
| `double(name)` | `double` | (API 4) |
| `string(name)` | `String` | (API 4) |
| `bytes(name)` | `ByteArray` | fresh array (API 4) |
| `uuid(name)` | `UUID` | (API 4) |
| `vec(name)` | `Vec` | (API 4) |
| `struct(name)` | `PartyFields` | one nested shape, read with these same methods (API 4) |
| `structs(name)` | `List<PartyFields>` | a list of nested shapes (API 4) |
| `ints(name)` | `List<Int>` | a list of int (API 4) |
| `floats(name)` | `List<Float>` | a list of float (API 4) |
| `doubles(name)` | `List<Double>` | a list of double (API 4) |
| `strings(name)` | `List<String>` | a list of string (API 4) |
| `vecs(name)` | `List<Vec>` | a list of vec (API 4) |
| `dump()` | `String` | every field as one line, defaults marked (API 4) |

A field the sender left out returns its declared default; without a default it throws `ScriptException` naming the field. Asking for the wrong type throws too, and so does a name the shape does not declare.

### PartyFields

| Method | Type | Description |
|---|---|---|
| `booleanValue(name)` | `boolean` | Java name behind `bool(name)` (API 4) |
| `intValue(name)` | `int` | Java name behind `int(name)` (API 4) |
| `longValue(name)` | `long` | Java name behind `long(name)` (API 4) |
| `floatValue(name)` | `float` | Java name behind `float(name)` (API 4) |
| `doubleValue(name)` | `double` | Java name behind `double(name)` (API 4) |
| `values(name)` | `List<Object>` | boxed list field, behind `ints` `floats` `doubles` `strings` `vecs` `structs` (API 4) |

## Packing bytes yourself

| Method | Type | Description |
|---|---|---|
| `writeBoolean(value)` | `PartyPayloadWriter` | 1 byte (API 4) |
| `writeByte(value)` | `PartyPayloadWriter` | 1 byte, low 8 bits of the int (API 4) |
| `writeShort(value)` | `PartyPayloadWriter` | 2 bytes, big endian (API 4) |
| `writeInt(value)` | `PartyPayloadWriter` | 4 bytes, big endian (API 4) |
| `writeLong(value)` | `PartyPayloadWriter` | 8 bytes, big endian (API 4) |
| `writeFloat(value)` | `PartyPayloadWriter` | 4 bytes (API 4) |
| `writeDouble(value)` | `PartyPayloadWriter` | 8 bytes (API 4) |
| `writeString(value)` | `PartyPayloadWriter` | 2-byte length plus utf-8, at most 512 bytes (API 4) (throws `ScriptException` when null or longer) |
| `writeBytes(value)` | `PartyPayloadWriter` | 2-byte length plus a copy of the array (API 4) (throws `ScriptException` when null) |
| `writeUuid(value)` | `PartyPayloadWriter` | 16 bytes (API 4) (throws `ScriptException` when null) |
| `toByteArray()` | `ByteArray` | what you wrote, ready for a `bytes` field (API 4) |
| `size()` | `int` | bytes written (API 4) |
| `remaining()` | `int` | bytes left before the limit (API 4) |
| `limit()` | `int` | 2048 from `party.buffer()` (API 4) |

A `bytes` field carries whatever you put in it: `party.buffer()` hands out the writer, `party.reader(data)` reads it back, and both are positional, so read order has to match write order. Any write past the limit throws `ScriptException`.

### PartyPayloadReader

| Method | Type | Description |
|---|---|---|
| `readBoolean()` | `boolean` | 1 byte, non-zero is true (API 4) |
| `readByte()` | `byte` | one signed byte (API 4) |
| `readShort()` | `short` | 2 bytes, big endian (API 4) |
| `readInt()` | `int` | 4 bytes, big endian (API 4) |
| `readLong()` | `long` | 8 bytes, big endian (API 4) |
| `readFloat()` | `float` | 4 bytes (API 4) |
| `readDouble()` | `double` | 8 bytes (API 4) |
| `readString()` | `String` | 2-byte length plus utf-8 (API 4) |
| `readBytes()` | `ByteArray` | 2-byte length plus the bytes (API 4) |
| `readUuid()` | `UUID` | 16 bytes (API 4) |
| `size()` | `int` | payload length (API 4) |
| `remaining()` | `int` | bytes not read yet (API 4) |
| `hasMore()` | `boolean` | whether any bytes are left (API 4) |

Reading past the end throws `ScriptException`; the payload is a plain array, so a truncated or reordered read gives garbage rather than an error.

## Limits

| Limit | Value | Description |
|---|---|---|
| Event payload | `2048` | bytes per message |
| State payload | `1024` | bytes per topic |
| String inside a payload | `512` | bytes, and it counts toward the payload budget |
| Fields in a shape | `64` | |
| Field id | `1..2047` | 1 to 15 take one byte on the wire |
| Field name | `32` | ASCII characters |
| State topics | `8` | per account, across all your scripts |
| Namespace | `48` | ASCII characters |
| Topic | `32` | ASCII characters, the shape name is the topic |
| Channels | `8` | per script |
| State lifetime | `5 min` | since the last publish, unless republished |

Sends beyond the client rate limiter come back `RATE_LIMITED`; the server keeps its own, stricter budgets and drops what exceeds them without an error. State publishes coalesce per topic, so publishing on every tick costs one packet every few ticks and only the newest value goes out.

## What the server decides

| Fact | Description |
|---|---|
| Payload | opaque bytes; the server never reads a field or knows your shape |
| Sender | derived from the authenticated session, so a payload cannot claim to be someone else |
| Party | derived from the sender's current party; a message never leaves it |
| Leader | derived from the party, so `SenderRule.LEADER_ONLY` cannot be spoofed |
| Recipients | derived from the party; the client never sends a recipient list |
| State snapshot | sent to a member on join, with events never replayed |
| State cleanup | on leave, kick, disband, disconnect and after the lifetime above |

A namespace is a routing name, not an identity: it proves nothing about which script sent a message, only which account did.
