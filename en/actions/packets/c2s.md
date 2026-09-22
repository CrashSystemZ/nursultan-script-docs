# Packets you can send

65 client-to-server records; each is a Java record you build with its canonical constructor and hand to `packets.send` or `packets.sendSequenced` — see [Packets](../packets.md#sending). 14 of them are observe-only: `send` returns `false` and warns once in the script console. A send also fails on a null field, an enum constant with no vanilla twin, an entity outside the loaded world or a slot out of range; the enum types are listed in [Packet enums](enums.md).

```kotlin
name("Instant respawn")
key(Key.R)

onEnable {
    // the death screen's Respawn button, as a packet
    packets.send(C2SClientStatusPacket(ClientStatus.PERFORM_RESPAWN))
}
```

## C2SAcknowledgeChunksPacket

Chunk-batch rate the client asks for after finishing a batch.

| Component | Type | Description |
|---|---|---|
| `desiredChunksPerTick()` | `float` | requested chunks delivered per tick |

## C2SAcknowledgeReconfigurationPacket

Acknowledges the server's switch to the configuration phase; no components.

## C2SAdvancementTabPacket

Advancement screen tab opened or closed. Observe-only.

| Component | Type | Description |
|---|---|---|
| `action()` | `AdvancementTabAction` | tab opened or screen closed |
| `tabToOpen()` | `String?` | namespaced advancement tab id, null when closing |

## C2SBoatPaddleStatePacket

Paddle animation state while riding a boat.

| Component | Type | Description |
|---|---|---|
| `leftPaddling()` | `boolean` | left paddle turning |
| `rightPaddling()` | `boolean` | right paddle turning |

## C2SBookUpdatePacket

Writable-book edit or signing. Observe-only.

| Component | Type | Description |
|---|---|---|
| `slot()` | `int` | hotbar slot holding the book |
| `pages()` | `List<String>` | page texts in order |
| `title()` | `String?` | title given at signing, null while unsigned |

## C2SBundleItemSelectedPacket

Selects the active item inside a bundle.

| Component | Type | Description |
|---|---|---|
| `slotId()` | `int` | screen-handler slot holding the bundle |
| `selectedItemIndex()` | `int` | index inside the bundle contents |

## C2SButtonClickPacket

Button pressed inside a screen handler: enchanting table, trade, loom, stonecutter.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | screen handler sync id |
| `buttonId()` | `int` | button index, meaning depends on the handler |

## C2SChangeGameModePacket

Client asks to change its own game mode.

| Component | Type | Description |
|---|---|---|
| `mode()` | `GameMode` | requested game mode; send fails when unmapped |

## C2SChatCommandSignedPacket

Signed command execution. Observe-only; signatures and acknowledgments are not exposed.

| Component | Type | Description |
|---|---|---|
| `command()` | `String` | command text without the leading slash |
| `timestamp()` | `long` | signing time, epoch milliseconds |
| `salt()` | `long` | signing salt |

## C2SChatMessagePacket

Plain chat message from the player. Observe-only; signature data is not exposed.

| Component | Type | Description |
|---|---|---|
| `chatMessage()` | `String` | raw message text |
| `timestamp()` | `long` | signing time, epoch milliseconds |
| `salt()` | `long` | signing salt |

## C2SClickSlotPacket

Inventory slot click. Observe-only; the changed-stack map and the cursor stack are not exposed.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | screen handler sync id |
| `revision()` | `int` | handler revision the click was made against |
| `slot()` | `short` | clicked slot index, -999 outside the window |
| `button()` | `byte` | mouse button or hotbar index, meaning depends on actionType |
| `actionType()` | `SlotActionType` | click semantics |

## C2SClientCommandPacket

Player state command: leave bed, sprint, horse jump, ride inventory, elytra.

| Component | Type | Description |
|---|---|---|
| `mode()` | `ClientCommand` | requested state change; send fails without a player or when unmapped |

## C2SClientOptionsPacket

Full client settings block, sent on join and on every option change.

| Component | Type | Description |
|---|---|---|
| `language()` | `String` | locale code, e.g. `en_us` |
| `viewDistance()` | `int` | render distance in chunks |
| `chatVisibility()` | `ChatVisibility` | chat display preference; send fails when unmapped |
| `chatColorsEnabled()` | `boolean` | colored chat allowed |
| `playerModelParts()` | `int` | skin layer bitmask: cape, jacket, sleeves, pants, hat |
| `mainArm()` | `Arm` | main hand side; send fails when unmapped |
| `filtersText()` | `boolean` | profanity filtering enabled |
| `allowsServerListing()` | `boolean` | may appear in server listings |
| `particleStatus()` | `ParticlesMode` | particle quality; send fails when unmapped |

## C2SClientStatusPacket

Respawn request or statistics request.

| Component | Type | Description |
|---|---|---|
| `mode()` | `ClientStatus` | respawn or request stats; send fails when unmapped |

## C2SClientTickEndPacket

Marks the end of a client tick; no components.

## C2SCloseHandledScreenPacket

Client closed a screen handler.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | sync id of the closed handler |

## C2SCommandExecutionPacket

Unsigned command execution.

| Component | Type | Description |
|---|---|---|
| `command()` | `String` | command text without the leading slash |

## C2SCommonPongPacket

Reply to `S2CCommonPingPacket`.

| Component | Type | Description |
|---|---|---|
| `parameter()` | `int` | value echoed back from the ping |

## C2SCookieResponsePacket

Cookie value returned to the server. Observe-only; the payload bytes are not exposed.

| Component | Type | Description |
|---|---|---|
| `key()` | `String` | namespaced cookie id |

## C2SCraftRequestPacket

Recipe-book craft request.

| Component | Type | Description |
|---|---|---|
| `syncId()` | `int` | crafting screen handler sync id |
| `recipeId()` | `int` | network recipe index |
| `craftAll()` | `boolean` | craft a full stack instead of one |

## C2SCreativeInventoryActionPacket

Creative-mode slot set. Observe-only; item components and NBT are not exposed.

| Component | Type | Description |
|---|---|---|
| `slot()` | `int` | screen-handler slot index |
| `item()` | `String` | namespaced item id of the placed stack |
| `count()` | `int` | stack size |

## C2SCustomClickActionPacket

Custom click action from a server dialog. Observe-only; the payload NBT is not exposed.

| Component | Type | Description |
|---|---|---|
| `id()` | `String` | namespaced action id |

## C2SCustomPayloadPacket

Plugin-channel message. Observe-only; the payload bytes are not exposed.

| Component | Type | Description |
|---|---|---|
| `channel()` | `String` | namespaced channel id |

## C2SDebugSubscriptionRequestPacket

Debug-sample subscription request. Observe-only; no components.

## C2SHandSwingPacket

Arm swing animation.

| Component | Type | Description |
|---|---|---|
| `offhand()` | `boolean` | true swings the off hand, false the main hand |

## C2SJigsawGeneratingPacket

Generate pressed on a jigsaw block.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | jigsaw block x |
| `y()` | `int` | jigsaw block y |
| `z()` | `int` | jigsaw block z |
| `maxDepth()` | `int` | generation depth in levels |
| `keepJigsaws()` | `boolean` | leave jigsaw blocks in the result |

## C2SKeepAlivePacket

Keep-alive reply.

| Component | Type | Description |
|---|---|---|
| `id()` | `long` | id copied from the server keep-alive |

## C2SMessageAcknowledgmentPacket

Acknowledges received chat messages.

| Component | Type | Description |
|---|---|---|
| `offset()` | `int` | number of newly acknowledged messages |

## C2SMoveFullPacket

Movement packet carrying position and rotation.

| Component | Type | Description |
|---|---|---|
| `x()` | `double` | absolute x in blocks |
| `y()` | `double` | absolute y in blocks, feet level |
| `z()` | `double` | absolute z in blocks |
| `yaw()` | `float` | yaw in degrees |
| `pitch()` | `float` | pitch in degrees, -90..90 |
| `onGround()` | `boolean` | standing on ground flag |
| `horizontalCollision()` | `boolean` | collided horizontally this tick |

## C2SMoveLookPacket

Movement packet carrying rotation only.

| Component | Type | Description |
|---|---|---|
| `yaw()` | `float` | yaw in degrees |
| `pitch()` | `float` | pitch in degrees, -90..90 |
| `onGround()` | `boolean` | standing on ground flag |
| `horizontalCollision()` | `boolean` | collided horizontally this tick |

## C2SMoveOnGroundPacket

Movement packet carrying flags only, sent on an idle tick.

| Component | Type | Description |
|---|---|---|
| `onGround()` | `boolean` | standing on ground flag |
| `horizontalCollision()` | `boolean` | collided horizontally this tick |

## C2SMovePositionPacket

Movement packet carrying position only.

| Component | Type | Description |
|---|---|---|
| `x()` | `double` | absolute x in blocks |
| `y()` | `double` | absolute y in blocks, feet level |
| `z()` | `double` | absolute z in blocks |
| `onGround()` | `boolean` | standing on ground flag |
| `horizontalCollision()` | `boolean` | collided horizontally this tick |

## C2SPickItemFromBlockPacket

Middle-click pick block.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | target block x |
| `y()` | `int` | target block y |
| `z()` | `int` | target block z |
| `includeData()` | `boolean` | copy block-entity data into the picked stack |

## C2SPickItemFromEntityPacket

Middle-click pick entity.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | target entity network id |
| `includeData()` | `boolean` | copy entity data into the picked stack |

## C2SPlayerActionPacket

Block-break stage, item drop, hand swap or use-release.

| Component | Type | Description |
|---|---|---|
| `action()` | `PlayerAction` | which action; send fails when unmapped |
| `x()` | `int` | target block x |
| `y()` | `int` | target block y |
| `z()` | `int` | target block z |
| `side()` | `Side` | clicked block face; send fails when unmapped |
| `sequence()` | `int` | interaction sequence; used as-is by `send`, replaced by `sendSequenced` |

## C2SPlayerInputPacket

Raw movement key state, one flag per input.

| Component | Type | Description |
|---|---|---|
| `forward()` | `boolean` | forward key held |
| `backward()` | `boolean` | backward key held |
| `left()` | `boolean` | strafe-left key held |
| `right()` | `boolean` | strafe-right key held |
| `jump()` | `boolean` | jump key held |
| `sneak()` | `boolean` | sneak key held |
| `sprint()` | `boolean` | sprint key held |

## C2SPlayerInteractBlockPacket

Right-click on a block.

| Component | Type | Description |
|---|---|---|
| `offhand()` | `boolean` | true uses the off hand |
| `x()` | `int` | clicked block x |
| `y()` | `int` | clicked block y |
| `z()` | `int` | clicked block z |
| `side()` | `Side` | clicked face; send fails when unmapped |
| `hitX()` | `double` | absolute world x of the hit point |
| `hitY()` | `double` | absolute world y of the hit point |
| `hitZ()` | `double` | absolute world z of the hit point |
| `insideBlock()` | `boolean` | ray started inside the block shape |
| `sequence()` | `int` | interaction sequence; used as-is by `send`, replaced by `sendSequenced` |

## C2SPlayerInteractEntityPacket

Attack, interact or interact-at against an entity.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | target entity network id; send fails when it is not in the loaded world |
| `sneaking()` | `boolean` | player was sneaking |
| `type()` | `InteractType` | ATTACK, INTERACT or INTERACT_AT; send fails when null |
| `hand()` | `Hand?` | used hand, always null for ATTACK; send fails when null otherwise |
| `hitX()` | `Double?` | hit x relative to the entity, INTERACT_AT only |
| `hitY()` | `Double?` | hit y relative to the entity, INTERACT_AT only |
| `hitZ()` | `Double?` | hit z relative to the entity, INTERACT_AT only |

## C2SPlayerInteractItemPacket

Right-click with the held item, not aimed at a block.

| Component | Type | Description |
|---|---|---|
| `offhand()` | `boolean` | true uses the off hand |
| `sequence()` | `int` | interaction sequence; used as-is by `send`, replaced by `sendSequenced` |
| `yaw()` | `float` | yaw in degrees at use time |
| `pitch()` | `float` | pitch in degrees at use time |

## C2SPlayerLoadedPacket

Client finished loading into the world; no components.

## C2SPlayerSessionPacket

Chat session and public-key announcement. Observe-only; no components.

## C2SQueryBlockNbtPacket

Operator block-NBT query, answered by `S2CNbtQueryResponsePacket`.

| Component | Type | Description |
|---|---|---|
| `transactionId()` | `int` | id matched by the response |
| `x()` | `int` | queried block x |
| `y()` | `int` | queried block y |
| `z()` | `int` | queried block z |

## C2SQueryEntityNbtPacket

Operator entity-NBT query, answered by `S2CNbtQueryResponsePacket`.

| Component | Type | Description |
|---|---|---|
| `transactionId()` | `int` | id matched by the response |
| `entityId()` | `int` | queried entity network id |

## C2SRecipeBookDataPacket

Recipe clicked in the recipe book.

| Component | Type | Description |
|---|---|---|
| `recipeId()` | `int` | network recipe index |

## C2SRecipeCategoryOptionsPacket

Per-category recipe-book UI state.

| Component | Type | Description |
|---|---|---|
| `category()` | `RecipeBookCategory` | recipe book tab; send fails when unmapped |
| `guiOpen()` | `boolean` | category panel expanded |
| `filteringCraftable()` | `boolean` | show craftable only |

## C2SRenameItemPacket

Anvil rename field text.

| Component | Type | Description |
|---|---|---|
| `name()` | `String` | new item name |

## C2SRequestCommandCompletionsPacket

Tab-completion request, answered by `S2CCommandSuggestionsPacket`.

| Component | Type | Description |
|---|---|---|
| `completionId()` | `int` | id matched by the response |
| `partialCommand()` | `String` | text typed so far, including the leading slash |

## C2SResourcePackStatusPacket

Resource-pack download and apply status. Observe-only.

| Component | Type | Description |
|---|---|---|
| `id()` | `String` | resource pack uuid string |
| `status()` | `ResourcePackStatus` | accepted, declined, loaded or failed |

## C2SSelectMerchantTradePacket

Villager trade selection.

| Component | Type | Description |
|---|---|---|
| `tradeId()` | `int` | index into the offer list |

## C2SSelectSlotPacket

Hotbar slot selection.

| Component | Type | Description |
|---|---|---|
| `slot()` | `int` | hotbar index 0..8; send fails outside that range |

## C2SSetTestBlockPacket

Test block configuration write.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | test block x |
| `y()` | `int` | test block y |
| `z()` | `int` | test block z |
| `mode()` | `TestBlockMode` | test block mode; send fails when unmapped |
| `message()` | `String` | message field text |

## C2SSlotChangedStatePacket

Crafter slot enabled or disabled.

| Component | Type | Description |
|---|---|---|
| `slotId()` | `int` | crafter slot index |
| `screenHandlerId()` | `int` | screen handler sync id |
| `newState()` | `boolean` | true enables the slot |

## C2SSpectatorTeleportPacket

Spectator teleport to a player by uuid.

| Component | Type | Description |
|---|---|---|
| `uuid()` | `String` | target player uuid string; send fails when null or unparseable |

## C2STeleportConfirmPacket

Confirms a teleport id from `S2CPlayerPositionLookPacket`.

| Component | Type | Description |
|---|---|---|
| `teleportId()` | `int` | id copied from the server teleport |

## C2STestInstanceBlockActionPacket

Button pressed on a test instance block. Observe-only.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | test instance block x |
| `y()` | `int` | test instance block y |
| `z()` | `int` | test instance block z |
| `action()` | `TestInstanceBlockAction` | which button was pressed |

## C2SUpdateBeaconPacket

Beacon effect selection.

| Component | Type | Description |
|---|---|---|
| `primary()` | `String?` | namespaced status effect id, null or blank means none |
| `secondary()` | `String?` | namespaced status effect id, null or blank means none |

An unknown or unparseable id is sent as "none" instead of failing the send.

## C2SUpdateCommandBlockMinecartPacket

Command-block minecart edit.

| Component | Type | Description |
|---|---|---|
| `command()` | `String` | command text |
| `trackOutput()` | `boolean` | store the last output |
| `entityId()` | `int` | minecart entity network id |

## C2SUpdateCommandBlockPacket

Command block edit.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | command block x |
| `y()` | `int` | command block y |
| `z()` | `int` | command block z |
| `command()` | `String` | command text |
| `type()` | `CommandBlockType` | impulse, chain or repeat; send fails when unmapped |
| `trackOutput()` | `boolean` | store the last output |
| `conditional()` | `boolean` | conditional mode |
| `alwaysActive()` | `boolean` | always active instead of needing redstone |

## C2SUpdateDifficultyLockPacket

Difficulty lock toggle; single-player or LAN host only.

| Component | Type | Description |
|---|---|---|
| `difficultyLocked()` | `boolean` | lock the world difficulty |

## C2SUpdateDifficultyPacket

Difficulty change; single-player or LAN host only.

| Component | Type | Description |
|---|---|---|
| `difficulty()` | `Difficulty` | requested difficulty; send fails when unmapped |

## C2SUpdateJigsawPacket

Jigsaw block edit. Observe-only.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | jigsaw block x |
| `y()` | `int` | jigsaw block y |
| `z()` | `int` | jigsaw block z |
| `name()` | `String` | namespaced jigsaw name |
| `target()` | `String` | namespaced target jigsaw name |
| `pool()` | `String` | namespaced template pool id |
| `finalState()` | `String` | block state string placed after generation |
| `jointType()` | `JigsawJoint` | rollable or aligned joint |
| `selectionPriority()` | `int` | selection order weight |
| `placementPriority()` | `int` | placement order weight |

## C2SUpdatePlayerAbilitiesPacket

Flight toggle.

| Component | Type | Description |
|---|---|---|
| `flying()` | `boolean` | player is flying |

`send` builds a fresh abilities object with only `flying` set; the other ability flags are not carried.

## C2SUpdateSignPacket

Sign text write.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | sign block x |
| `y()` | `int` | sign block y |
| `z()` | `int` | sign block z |
| `front()` | `boolean` | true edits the front face, false the back |
| `line1()` | `String` | first text line |
| `line2()` | `String` | second text line |
| `line3()` | `String` | third text line |
| `line4()` | `String` | fourth text line |

## C2SUpdateStructureBlockPacket

Structure block edit.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | structure block x |
| `y()` | `int` | structure block y |
| `z()` | `int` | structure block z |
| `action()` | `StructureBlockAction` | update, save, load or detect; send fails when unmapped |
| `mode()` | `StructureBlockMode` | save, load, corner or data; send fails when unmapped |
| `templateName()` | `String` | structure name field |
| `offsetX()` | `int` | relative offset x in blocks |
| `offsetY()` | `int` | relative offset y in blocks |
| `offsetZ()` | `int` | relative offset z in blocks |
| `sizeX()` | `int` | structure size x in blocks |
| `sizeY()` | `int` | structure size y in blocks |
| `sizeZ()` | `int` | structure size z in blocks |
| `mirror()` | `BlockMirror` | mirror setting; send fails when unmapped |
| `rotation()` | `BlockRotation` | rotation setting; send fails when unmapped |
| `metadata()` | `String` | data-mode metadata string |
| `ignoreEntities()` | `boolean` | exclude entities from save and load |
| `strict()` | `boolean` | strict placement mode |
| `showAir()` | `boolean` | render invisible blocks |
| `showBoundingBox()` | `boolean` | render the bounding box |
| `integrity()` | `float` | load integrity, 0..1 |
| `seed()` | `long` | integrity randomization seed |

## C2SVehicleMovePacket

Position and rotation of the vehicle the player drives.

| Component | Type | Description |
|---|---|---|
| `x()` | `double` | absolute vehicle x in blocks |
| `y()` | `double` | absolute vehicle y in blocks |
| `z()` | `double` | absolute vehicle z in blocks |
| `yaw()` | `float` | vehicle yaw in degrees |
| `pitch()` | `float` | vehicle pitch in degrees |
| `onGround()` | `boolean` | vehicle standing on ground flag |
