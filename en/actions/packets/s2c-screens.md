# Screen and chat packets

Server-to-client records about screen handlers, inventories, trades, the recipe book, chat, titles, the tab list, the scoreboard and connection housekeeping. All 63 arrive through `PacketReceiveEvent` and none of them can be sent; text in them is a plain string, an item is a namespaced id plus a count, and an enum component is `null` when the vanilla constant has no same-named twin — [Packets](../packets.md).

```kotlin
on<PacketReceiveEvent> { e ->
    val packet = e.packet()
    if (packet !is S2CScreenHandlerSlotUpdatePacket) return@on
    if (packet.item() == "minecraft:air") return@on

    log.info("slot ${packet.slot()}: ${packet.item()} x${packet.count()}")
}
```

## S2CAdvancementUpdatePacket

Advancement list sync; the earned and progress maps and the display data are not exposed.

| Component | Type | Description |
|---|---|---|
| `clearCurrent()` | `boolean` | replace the whole advancement set instead of merging |
| `toRemove()` | `List<String>` | namespaced ids of the removed advancements |
| `showToast()` | `boolean` | show the toast for newly earned advancements |

## S2CBossBarPacket

Boss bar added, removed or updated; title, percent, colour and style are not exposed.

| Component | Type | Description |
|---|---|---|
| `uuid()` | `String` | boss bar uuid string |
| `action()` | `BossBarAction?` | which field group this packet changes |

## S2CBundleDelimiterPacket

Marks the boundary of a packet bundle. No components.

## S2CBundlePacket

Packet bundle wrapper; each contained packet fires its own event. No components.

## S2CChatMessagePacket

Signed player chat; the signed body text and the signature data are not exposed.

| Component | Type | Description |
|---|---|---|
| `globalIndex()` | `int` | server-wide message index |
| `sender()` | `String` | sender uuid string |
| `index()` | `int` | per-sender message index |
| `unsignedContent()` | `String?` | server-modified display text, null when the signed body is shown |

## S2CChatSuggestionsPacket

Server-driven chat completion list.

| Component | Type | Description |
|---|---|---|
| `action()` | `ChatSuggestionAction?` | add, remove or replace the list |
| `entries()` | `List<String>` | suggestion strings |

## S2CClearDialogPacket

Closes the open server dialog. No components.

## S2CClearTitlePacket

Clears the title and the subtitle.

| Component | Type | Description |
|---|---|---|
| `reset()` | `boolean` | also reset the fade-in, stay and fade-out timings |

## S2CCloseScreenPacket

The server closes a screen handler.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | screen handler sync id, 0 is the player inventory |

## S2CCommandSuggestionsPacket

Tab-completion reply; the suggestion tooltips are not exposed.

| Component | Type | Description |
|---|---|---|
| `id()` | `int` | completion id matching the request |
| `start()` | `int` | start index of the replaced range in the input |
| `length()` | `int` | length of the replaced range |
| `suggestions()` | `List<String>` | completion texts |

## S2CCommandTreePacket

Brigadier command tree; the tree itself is not exposed. No components.

## S2CCommonPingPacket

Server ping; the client answers with `C2SCommonPongPacket`.

| Component | Type | Description |
|---|---|---|
| `parameter()` | `int` | value to echo back |

## S2CCookieRequestPacket

The server asks for a stored cookie.

| Component | Type | Description |
|---|---|---|
| `key()` | `String` | namespaced cookie id |

## S2CCooldownUpdatePacket

Item-use cooldown set or cleared.

| Component | Type | Description |
|---|---|---|
| `cooldownGroup()` | `String` | namespaced cooldown group id |
| `cooldown()` | `int` | cooldown length in ticks, 0 clears it |

## S2CCraftFailedResponsePacket

A recipe-book craft request was rejected.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | screen handler sync id of the rejected request |

## S2CCustomPayloadPacket

Plugin-channel message; the payload bytes are not exposed.

| Component | Type | Description |
|---|---|---|
| `channel()` | `String` | namespaced channel id |

## S2CCustomReportDetailsPacket

Key/value details the server wants attached to crash reports.

| Component | Type | Description |
|---|---|---|
| `details()` | `Map<String, String>` | immutable copy of the detail entries |

## S2CDebugSamplePacket

Debug profiler sample push.

| Component | Type | Description |
|---|---|---|
| `sample()` | `List<Long>` | raw sample values, meaning depends on `debugSampleType()` |
| `debugSampleType()` | `DebugSampleType?` | which profiler produced the sample |

## S2CDisconnectPacket

The server disconnects the client.

| Component | Type | Description |
|---|---|---|
| `reason()` | `String` | plain-text disconnect reason |

## S2CEnterReconfigurationPacket

The connection moves to the configuration phase. No components.

## S2CEventDebugPacket

Debug event push; no fields are exposed. No components.

## S2CGameMessagePacket

System message; player chat arrives as `S2CChatMessagePacket` instead.

| Component | Type | Description |
|---|---|---|
| `content()` | `String` | plain-text message |
| `overlay()` | `boolean` | draw on the action bar instead of the chat log |

## S2CInventoryPacket

Full contents resend for one screen handler.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | screen handler sync id, 0 is the player inventory |
| `revision()` | `int` | handler state revision |
| `contents()` | `List<Stack>` | slot contents in slot order |
| `cursorItem()` | `String` | namespaced item id on the cursor, `minecraft:air` when empty |
| `cursorCount()` | `int` | cursor stack size |

### S2CInventoryPacket.Stack

One entry of `contents()`.

| Component | Type | Description |
|---|---|---|
| `item()` | `String` | namespaced item id, `minecraft:air` when empty |
| `count()` | `int` | stack size |

## S2CKeepAlivePacket

Keep-alive; the client echoes the id back in `C2SKeepAlivePacket`.

| Component | Type | Description |
|---|---|---|
| `id()` | `long` | keep-alive id to echo back |

## S2CNbtQueryResponsePacket

Reply to `C2SQueryBlockNbtPacket` or `C2SQueryEntityNbtPacket`.

| Component | Type | Description |
|---|---|---|
| `transactionId()` | `int` | id matching the request |
| `nbt()` | `String` | SNBT text of the result, empty when the query returned nothing |

## S2COpenMountScreenPacket

Opens the horse or mount inventory screen.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | screen handler sync id |
| `slotColumnCount()` | `int` | number of chest slot columns |
| `mountId()` | `int` | mount entity network id |

## S2COpenScreenPacket

Opens a container screen.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | screen handler sync id |
| `screenHandler()` | `String` | namespaced screen handler type id |
| `name()` | `String` | plain-text window title |

## S2COpenWrittenBookPacket

Opens the written-book reader for the held book.

| Component | Type | Description |
|---|---|---|
| `hand()` | `Hand?` | hand holding the book — [Inventory and items](../../game/inventory.md) |

## S2COverlayMessagePacket

Action bar text.

| Component | Type | Description |
|---|---|---|
| `text()` | `String` | plain-text action bar message |

## S2CPlayerListHeaderPacket

Tab-list header and footer.

| Component | Type | Description |
|---|---|---|
| `header()` | `String` | plain-text header |
| `footer()` | `String` | plain-text footer |

## S2CPlayerListPacket

Tab-list update; the per-player entries are not exposed, only which fields changed.

| Component | Type | Description |
|---|---|---|
| `actions()` | `List<PlayerListAction>` | field groups this packet carries, unmapped constants dropped |

## S2CPlayerRemovePacket

Players removed from the tab list.

| Component | Type | Description |
|---|---|---|
| `profileIds()` | `List<String>` | removed player uuid strings |

## S2CProfilelessChatMessagePacket

Chat message with no player profile behind it: console output, command output.

| Component | Type | Description |
|---|---|---|
| `message()` | `String` | plain-text message |

## S2CRecipeBookAddPacket

Recipe-book entries added; the entries themselves are not exposed.

| Component | Type | Description |
|---|---|---|
| `replace()` | `boolean` | replace the whole known-recipe set instead of merging |

## S2CRecipeBookRemovePacket

Recipe-book entries removed.

| Component | Type | Description |
|---|---|---|
| `recipes()` | `List<Integer>` | removed network recipe indices |

## S2CRecipeBookSettingsPacket

Per-category recipe-book UI settings; no fields are exposed. No components.

## S2CRemoveMessagePacket

The server deletes a chat message it sent; the signature is not exposed. No components.

## S2CResourcePackRemovePacket

Removes a server resource pack.

| Component | Type | Description |
|---|---|---|
| `id()` | `String?` | pack uuid string, null removes every server pack |

## S2CResourcePackSendPacket

The server offers a resource pack.

| Component | Type | Description |
|---|---|---|
| `id()` | `String` | pack uuid string |
| `url()` | `String` | download URL |
| `hash()` | `String` | SHA-1 hex digest of the pack, empty when not provided |
| `required()` | `boolean` | declining disconnects the client |
| `prompt()` | `String?` | plain-text prompt shown in the dialog |

## S2CScoreboardDisplayPacket

Assigns an objective to a display slot.

| Component | Type | Description |
|---|---|---|
| `slot()` | `ScoreboardSlot?` | sidebar, tab list or below-name slot |
| `name()` | `String?` | objective name, null clears the slot |

## S2CScoreboardObjectiveUpdatePacket

Objective created, removed or updated; the number format is not exposed.

| Component | Type | Description |
|---|---|---|
| `name()` | `String` | internal objective name |
| `displayName()` | `String` | plain-text display title |
| `type()` | `ScoreboardRenderType?` | number or hearts rendering |
| `mode()` | `int` | 0 add, 1 remove, 2 update |

## S2CScoreboardScoreResetPacket

Removes score entries.

| Component | Type | Description |
|---|---|---|
| `scoreHolderName()` | `String` | score holder, a player name or a selector result |
| `objectiveName()` | `String?` | objective to clear, null clears the holder everywhere |

## S2CScoreboardScoreUpdatePacket

Score value set; the number format is not exposed.

| Component | Type | Description |
|---|---|---|
| `scoreHolderName()` | `String` | score holder name |
| `objectiveName()` | `String` | objective name |
| `score()` | `int` | new score value |
| `display()` | `String?` | plain-text override for the displayed line |

## S2CScreenHandlerPropertyUpdatePacket

One screen handler property: furnace progress, enchantment levels, beacon effect.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | screen handler sync id |
| `propertyId()` | `int` | handler-specific property index |
| `value()` | `int` | new property value |

## S2CScreenHandlerSlotUpdatePacket

One slot's contents changed.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | screen handler sync id, -1 targets the cursor stack |
| `revision()` | `int` | handler state revision |
| `slot()` | `int` | slot index |
| `item()` | `String` | namespaced item id, `minecraft:air` when empty |
| `count()` | `int` | stack size |

## S2CSelectAdvancementTabPacket

The server switches the open advancement tab.

| Component | Type | Description |
|---|---|---|
| `tabId()` | `String?` | namespaced advancement tab id, null deselects |

## S2CServerLinksPacket

Server-provided links; the link labels are not exposed.

| Component | Type | Description |
|---|---|---|
| `links()` | `List<String>` | link URLs |

## S2CServerMetadataPacket

In-play server metadata; the favicon bytes are not exposed.

| Component | Type | Description |
|---|---|---|
| `description()` | `String` | plain-text MOTD |

## S2CServerTransferPacket

The server tells the client to connect elsewhere.

| Component | Type | Description |
|---|---|---|
| `host()` | `String` | target hostname or address |
| `port()` | `int` | target port |

## S2CSetCursorItemPacket

Sets the stack held on the mouse cursor.

| Component | Type | Description |
|---|---|---|
| `item()` | `String` | namespaced item id, `minecraft:air` when empty |
| `count()` | `int` | stack size |

## S2CSetPlayerInventoryPacket

Sets one player-inventory slot outside any open handler.

| Component | Type | Description |
|---|---|---|
| `slot()` | `int` | player inventory slot index |
| `item()` | `String` | namespaced item id, `minecraft:air` when empty |
| `count()` | `int` | stack size |

## S2CSetTradeOffersPacket

Villager or wandering-trader offer list.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | merchant screen handler sync id |
| `levelProgress()` | `int` | merchant level, 1..5 |
| `experience()` | `int` | merchant experience points |
| `leveled()` | `boolean` | merchant can level up, false for a wandering trader |
| `refreshable()` | `boolean` | offers restock over time |
| `offers()` | `List<Offer>` | trades in display order |

### S2CSetTradeOffersPacket.Offer

One trade.

| Component | Type | Description |
|---|---|---|
| `buyItem()` | `String` | namespaced first input id, after price adjustment |
| `buyCount()` | `int` | first input stack size, after price adjustment |
| `secondBuyItem()` | `String?` | namespaced second input id, null on one-input trades |
| `secondBuyCount()` | `int` | second input stack size, 0 when absent |
| `sellItem()` | `String` | namespaced output item id |
| `sellCount()` | `int` | output stack size |
| `uses()` | `int` | times this trade has been used |
| `maxUses()` | `int` | uses before the trade locks |
| `disabled()` | `boolean` | trade is currently locked out |

## S2CShowDialogPacket

Opens a server dialog; the dialog contents are not exposed. No components.

## S2CSignEditorOpenPacket

Opens the sign text editor.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | sign block x |
| `y()` | `int` | sign block y |
| `z()` | `int` | sign block z |
| `front()` | `boolean` | true edits the front face, false the back |

## S2CStatisticsPacket

Statistics screen data.

| Component | Type | Description |
|---|---|---|
| `stats()` | `List<Stat>` | statistic entries |

### S2CStatisticsPacket.Stat

One statistic.

| Component | Type | Description |
|---|---|---|
| `stat()` | `String` | stat type and value joined, e.g. `minecraft:mined:minecraft:stone` |
| `value()` | `int` | raw counter, time stats in ticks, distance stats in cm |

## S2CStoreCookiePacket

The server stores a cookie on the client; the payload bytes are not exposed.

| Component | Type | Description |
|---|---|---|
| `key()` | `String` | namespaced cookie id |

## S2CSubtitlePacket

Title subtitle line.

| Component | Type | Description |
|---|---|---|
| `text()` | `String` | plain-text subtitle |

## S2CSynchronizeRecipesPacket

Recipe registry sync; the recipes are not exposed. No components.

## S2CSynchronizeTagsPacket

Registry tag sync; the tags are not exposed. No components.

## S2CTeamPacket

Scoreboard team created, updated or removed, or members added to it and removed from it.

| Component | Type | Description |
|---|---|---|
| `teamName()` | `String` | internal team name |
| `playerNames()` | `List<String>` | affected member names, empty on a pure team update |
| `teamOperation()` | `TeamOperation?` | what happens to the team itself |
| `playerListOperation()` | `TeamPlayerListOperation?` | what happens to the member list |
| `displayName()` | `String?` | plain-text team display name |
| `prefix()` | `String?` | plain-text name prefix |
| `suffix()` | `String?` | plain-text name suffix |
| `friendlyFlags()` | `Integer?` | bitmask, 1 friendly fire, 2 see invisible teammates |
| `color()` | `Formatting?` | team colour formatting code |
| `nameTagVisibilityRule()` | `TeamVisibilityRule?` | when nametags render |
| `collisionRule()` | `TeamCollisionRule?` | when entities collide |

The seven detail components from `displayName()` down are null unless the packet carries team data, that is on create and update.

## S2CTitleFadePacket

Title animation timings.

| Component | Type | Description |
|---|---|---|
| `fadeInTicks()` | `int` | fade-in duration in ticks |
| `stayTicks()` | `int` | hold duration in ticks |
| `fadeOutTicks()` | `int` | fade-out duration in ticks |

## S2CTitlePacket

Title main line.

| Component | Type | Description |
|---|---|---|
| `text()` | `String` | plain-text title |

## S2CUpdateSelectedSlotPacket

The server forces the hotbar selection.

| Component | Type | Description |
|---|---|---|
| `slot()` | `int` | hotbar index 0..8 |
