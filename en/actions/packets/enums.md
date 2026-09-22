# Packet enums

34 enums, 180 constants, used as field types on the packet records in [Packets](../packets.md). Conversion is by constant name in both directions: a vanilla constant with no twin here decodes to `null`, and a constant with no vanilla twin makes the send fail.

```kotlin
on<PacketSendEvent> { e ->
    val packet = e.packet()
    if (packet !is C2SPlayerInteractEntityPacket) return@on
    if (packet.type() == InteractType.ATTACK) {
        log.info("attacked entity ${packet.entityId()}")
    }
}
```

Inside a `List<...>` field an unmapped constant is dropped from the list instead of becoming `null`.
`InteractType` is the one exception to name matching: it is rebuilt through the vanilla `PlayerInteractEntityC2SPacket.Handler` visitor on decode and branched on when sending.

## AdvancementTabAction

Field of `C2SAdvancementTabPacket`, vanilla `AdvancementTabC2SPacket.Action`. Decode only — that packet has no send builder.

| Constant | Description |
|---|---|
| `OPENED_TAB` | an advancement tab was opened |
| `CLOSED_SCREEN` | the advancements screen was closed |

## BlockMirror

Field of `C2SUpdateStructureBlockPacket`, vanilla `net.minecraft.util.BlockMirror`.

| Constant | Description |
|---|---|
| `NONE` | no mirroring |
| `LEFT_RIGHT` | mirrored across the left-right axis |
| `FRONT_BACK` | mirrored across the front-back axis |

## BlockRotation

Field of `C2SUpdateStructureBlockPacket`, vanilla `net.minecraft.util.BlockRotation`.

| Constant | Description |
|---|---|
| `NONE` | 0 degrees |
| `CLOCKWISE_90` | 90 degrees clockwise |
| `CLOCKWISE_180` | 180 degrees |
| `COUNTERCLOCKWISE_90` | 90 degrees counter-clockwise, 270 clockwise |

## BossBarAction

Field of `S2CBossBarPacket`, vanilla `BossBarS2CPacket.Type`.

| Constant | Description |
|---|---|
| `ADD` | new boss bar created |
| `REMOVE` | boss bar removed |
| `UPDATE_PROGRESS` | fill fraction changed, 0..1 |
| `UPDATE_NAME` | title text changed |
| `UPDATE_STYLE` | bar colour and notch division changed |
| `UPDATE_PROPERTIES` | darken-sky, dragon-music and fog flags changed |

## ChatSuggestionAction

Field of `S2CChatSuggestionsPacket`, vanilla `ChatSuggestionsS2CPacket.Action`.

| Constant | Description |
|---|---|
| `ADD` | append the entries to the current list |
| `REMOVE` | remove the entries from the list |
| `SET` | replace the whole list with the entries |

## ChatVisibility

Field of `C2SClientOptionsPacket`, vanilla `net.minecraft.network.message.ChatVisibility`.

| Constant | Description |
|---|---|
| `FULL` | all chat shown |
| `SYSTEM` | only system and command output shown |
| `HIDDEN` | chat hidden |

## ClientCommand

Field of `C2SClientCommandPacket`, vanilla `ClientCommandC2SPacket.Mode`.

| Constant | Description |
|---|---|
| `STOP_SLEEPING` | leave the bed |
| `START_SPRINTING` | begin sprinting |
| `STOP_SPRINTING` | end sprinting |
| `START_RIDING_JUMP` | begin charging a horse jump |
| `STOP_RIDING_JUMP` | release the horse jump |
| `OPEN_INVENTORY` | open the ridden entity's inventory |
| `START_FALL_FLYING` | start elytra flight |

The send builder re-attaches the local player as the packet's entity, so the send fails while `game.player()` is absent.

## ClientStatus

Field of `C2SClientStatusPacket`, vanilla `ClientStatusC2SPacket.Mode`.

| Constant | Description |
|---|---|
| `PERFORM_RESPAWN` | respawn after death |
| `REQUEST_STATS` | request statistics screen data |

## CommandBlockType

Field of `C2SUpdateCommandBlockPacket`, vanilla `CommandBlockBlockEntity.Type`.

| Constant | Description |
|---|---|
| `SEQUENCE` | chain command block |
| `AUTO` | repeating command block |
| `REDSTONE` | impulse command block |

## DebugSampleType

Field of `S2CDebugSamplePacket`, vanilla `net.minecraft.util.profiler.DebugSampleType`.

| Constant | Description |
|---|---|
| `TICK_TIME` | server tick timings, nanoseconds, in the packet's number list |

## Difficulty

Field of `C2SUpdateDifficultyPacket` and `S2CDifficultyPacket`, vanilla `net.minecraft.world.Difficulty`.

| Constant | Description |
|---|---|
| `PEACEFUL` | id 0, no hostile mobs |
| `EASY` | id 1 |
| `NORMAL` | id 2 |
| `HARD` | id 3 |

## EntityAnchor

Field of `S2CLookAtPacket`, vanilla `EntityAnchorArgumentType.EntityAnchor`.

| Constant | Description |
|---|---|
| `FEET` | anchor at the entity's feet, y offset 0 |
| `EYES` | anchor at the entity's eye height |

## Formatting

Team colour in `S2CTeamPacket`, vanilla `net.minecraft.util.Formatting`.

| Constant | Description |
|---|---|
| `BLACK` | colour, code `0` |
| `DARK_BLUE` | colour, code `1` |
| `DARK_GREEN` | colour, code `2` |
| `DARK_AQUA` | colour, code `3` |
| `DARK_RED` | colour, code `4` |
| `DARK_PURPLE` | colour, code `5` |
| `GOLD` | colour, code `6` |
| `GRAY` | colour, code `7` |
| `DARK_GRAY` | colour, code `8` |
| `BLUE` | colour, code `9` |
| `GREEN` | colour, code `a` |
| `AQUA` | colour, code `b` |
| `RED` | colour, code `c` |
| `LIGHT_PURPLE` | colour, code `d` |
| `YELLOW` | colour, code `e` |
| `WHITE` | colour, code `f` |
| `OBFUSCATED` | style, code `k` |
| `BOLD` | style, code `l` |
| `STRIKETHROUGH` | style, code `m` |
| `UNDERLINE` | style, code `n` |
| `ITALIC` | style, code `o` |
| `RESET` | style, code `r`; as a team colour means no colour |

## InteractType

Which of the three variants `C2SPlayerInteractEntityPacket` carries; rebuilt through the vanilla visitor, not by name.

| Constant | Description |
|---|---|
| `INTERACT` | right-click on entity; `hand` set, hit coordinates null |
| `INTERACT_AT` | right-click at a point; `hand` and hit coordinates set |
| `ATTACK` | left-click attack; `hand` and hit coordinates null |

## JigsawJoint

Field of `C2SUpdateJigsawPacket`, vanilla `JigsawBlockEntity.Joint`. Decode only — that packet has no send builder.

| Constant | Description |
|---|---|
| `ROLLABLE` | attached piece may rotate about the connection axis |
| `ALIGNED` | attached piece keeps a fixed orientation |

## ParticlesMode

Field of `C2SClientOptionsPacket`, vanilla `net.minecraft.particle.ParticlesMode`.

| Constant | Description |
|---|---|
| `ALL` | all particles |
| `DECREASED` | reduced particles |
| `MINIMAL` | minimal particles |

## PlayerAction

Field of `C2SPlayerActionPacket`, vanilla `PlayerActionC2SPacket.Action`.

| Constant | Description |
|---|---|
| `START_DESTROY_BLOCK` | begin breaking the block at the packet position |
| `ABORT_DESTROY_BLOCK` | cancel the in-progress break |
| `STOP_DESTROY_BLOCK` | finish breaking the block |
| `DROP_ALL_ITEMS` | drop the whole held stack |
| `DROP_ITEM` | drop one item from the held stack |
| `RELEASE_USE_ITEM` | release a charged or held-use item |
| `SWAP_ITEM_WITH_OFFHAND` | swap main hand and off hand |
| `STAB` | mace and spear stab, added in 1.21.11 |

## PlayerListAction

`S2CPlayerListPacket` carries a `List<PlayerListAction>`, vanilla `PlayerListS2CPacket.Action`.

| Constant | Description |
|---|---|
| `ADD_PLAYER` | new profile entry, name and skin properties |
| `INITIALIZE_CHAT` | chat session and public key data |
| `UPDATE_GAME_MODE` | game mode changed |
| `UPDATE_LISTED` | shown or hidden in the tab list |
| `UPDATE_LATENCY` | ping in ms changed |
| `UPDATE_DISPLAY_NAME` | tab-list display name changed |
| `UPDATE_LIST_ORDER` | explicit sort order changed |
| `UPDATE_HAT` | hat layer visibility changed |

## PositionFlag

`S2CEntityPositionPacket` carries a `List<PositionFlag>`, vanilla `net.minecraft.network.packet.s2c.play.PositionFlag`.

| Constant | Description |
|---|---|
| `X` | x is relative to the current x |
| `Y` | y is relative |
| `Z` | z is relative |
| `Y_ROT` | yaw is relative, degrees |
| `X_ROT` | pitch is relative, degrees |
| `DELTA_X` | x velocity is relative |
| `DELTA_Y` | y velocity is relative |
| `DELTA_Z` | z velocity is relative |
| `ROTATE_DELTA` | existing velocity is rotated by the rotation delta |

## RecipeBookCategory

Field of `C2SRecipeCategoryOptionsPacket`, vanilla `net.minecraft.recipe.book.RecipeBookType`.

| Constant | Description |
|---|---|
| `CRAFTING` | crafting table and inventory book |
| `FURNACE` | furnace book |
| `BLAST_FURNACE` | blast furnace book |
| `SMOKER` | smoker book |

## ResourcePackStatus

Field of `C2SResourcePackStatusPacket`, vanilla `ResourcePackStatusC2SPacket.Status`. Decode only — that packet has no send builder.

| Constant | Description |
|---|---|
| `SUCCESSFULLY_LOADED` | pack applied |
| `DECLINED` | user refused the pack |
| `FAILED_DOWNLOAD` | download failed |
| `ACCEPTED` | user accepted, download starting |
| `DOWNLOADED` | download finished, not yet applied |
| `INVALID_URL` | pack url could not be parsed |
| `FAILED_RELOAD` | resource reload failed after applying |
| `DISCARDED` | pack dropped without being applied |

## ScoreboardRenderType

Field of `S2CScoreboardObjectiveUpdatePacket`, vanilla `ScoreboardCriterion.RenderType`.

| Constant | Description |
|---|---|
| `INTEGER` | score drawn as a number |
| `HEARTS` | score drawn as hearts |

## ScoreboardSlot

Field of `S2CScoreboardDisplayPacket`, vanilla `net.minecraft.scoreboard.ScoreboardDisplaySlot`.

| Constant | Description |
|---|---|
| `LIST` | tab list |
| `SIDEBAR` | generic sidebar |
| `BELOW_NAME` | under the player's nametag |
| `TEAM_BLACK` | sidebar shown only to the black team |
| `TEAM_DARK_BLUE` | sidebar for the dark blue team |
| `TEAM_DARK_GREEN` | sidebar for the dark green team |
| `TEAM_DARK_AQUA` | sidebar for the dark aqua team |
| `TEAM_DARK_RED` | sidebar for the dark red team |
| `TEAM_DARK_PURPLE` | sidebar for the dark purple team |
| `TEAM_GOLD` | sidebar for the gold team |
| `TEAM_GRAY` | sidebar for the gray team |
| `TEAM_DARK_GRAY` | sidebar for the dark gray team |
| `TEAM_BLUE` | sidebar for the blue team |
| `TEAM_GREEN` | sidebar for the green team |
| `TEAM_AQUA` | sidebar for the aqua team |
| `TEAM_RED` | sidebar for the red team |
| `TEAM_LIGHT_PURPLE` | sidebar for the light purple team |
| `TEAM_YELLOW` | sidebar for the yellow team |
| `TEAM_WHITE` | sidebar for the white team |

## SlotActionType

Field of `C2SClickSlotPacket`, vanilla `net.minecraft.screen.slot.SlotActionType`. Decode only — that packet has no send builder.

| Constant | Description |
|---|---|
| `PICKUP` | normal left or right click pickup and place |
| `QUICK_MOVE` | shift-click transfer |
| `SWAP` | hotbar or offhand swap; `button` is the hotbar index, 40 = offhand |
| `CLONE` | creative middle-click clone |
| `THROW` | drop key; `button` 0 = one item, 1 = whole stack |
| `QUICK_CRAFT` | drag-distribute across slots |
| `PICKUP_ALL` | double-click collect of matching items |

## SoundCategory

Field of `S2CPlaySoundPacket`, `S2CPlaySoundFromEntityPacket` and `S2CStopSoundPacket`, vanilla `net.minecraft.sound.SoundCategory`.

| Constant | Description |
|---|---|
| `MASTER` | master channel |
| `MUSIC` | background music |
| `RECORDS` | jukebox and note blocks |
| `WEATHER` | rain and thunder |
| `BLOCKS` | block sounds |
| `HOSTILE` | hostile mob sounds |
| `NEUTRAL` | friendly mob sounds |
| `PLAYERS` | player sounds |
| `AMBIENT` | ambient and environment |
| `VOICE` | voice and speech |
| `UI` | interface sounds |

## StructureBlockAction

Field of `C2SUpdateStructureBlockPacket`, vanilla `StructureBlockBlockEntity.Action`.

| Constant | Description |
|---|---|
| `UPDATE_DATA` | write the settings without acting |
| `SAVE_AREA` | save the selected region to the template |
| `LOAD_AREA` | place the template into the world |
| `SCAN_AREA` | detect structure bounds from the corner blocks |

## StructureBlockMode

Field of `C2SUpdateStructureBlockPacket`, vanilla `net.minecraft.block.enums.StructureBlockMode`.

| Constant | Description |
|---|---|
| `SAVE` | save mode |
| `LOAD` | load mode |
| `CORNER` | corner marker mode |
| `DATA` | data marker mode |

## TeamCollisionRule

Field of `S2CTeamPacket`, null when the packet carries no team info, vanilla `AbstractTeam.CollisionRule`.

| Constant | Description |
|---|---|
| `ALWAYS` | always collides |
| `NEVER` | never collides |
| `PUSH_OTHER_TEAMS` | collides only with members of other teams |
| `PUSH_OWN_TEAM` | collides only with own team members |

## TeamOperation

What `S2CTeamPacket` does to the team itself, vanilla `TeamS2CPacket.Operation`.

| Constant | Description |
|---|---|
| `ADD` | team created or its info updated |
| `REMOVE` | team deleted |

## TeamPlayerListOperation

What `S2CTeamPacket` does to the team's member list; the same vanilla `TeamS2CPacket.Operation` on a separate field.

| Constant | Description |
|---|---|
| `ADD` | the listed players joined the team |
| `REMOVE` | the listed players left the team |

## TeamVisibilityRule

Field of `S2CTeamPacket`, null when the packet carries no team info, vanilla `AbstractTeam.VisibilityRule`.

| Constant | Description |
|---|---|
| `ALWAYS` | nametags always visible |
| `NEVER` | nametags never visible |
| `HIDE_FOR_OTHER_TEAMS` | hidden from players not on the team |
| `HIDE_FOR_OWN_TEAM` | hidden from the team's own members |

## TestBlockMode

Field of `C2SSetTestBlockPacket`, vanilla `net.minecraft.block.enums.TestBlockMode`.

| Constant | Description |
|---|---|
| `START` | starts the test run |
| `LOG` | logs its message when powered |
| `FAIL` | fails the test when powered |
| `ACCEPT` | marks the test passed when powered |

## TestInstanceBlockAction

Field of `C2STestInstanceBlockActionPacket`, vanilla `TestInstanceBlockActionC2SPacket.Action`. Decode only — that packet has no send builder.

| Constant | Description |
|---|---|
| `INIT` | create the test structure area |
| `QUERY` | ask the server for the block's current state |
| `SET` | write the edited settings |
| `RESET` | reset the test instance |
| `SAVE` | save the test structure |
| `EXPORT` | export the structure to a file |
| `RUN` | run the test |

## WaypointOperation

What `S2CWaypointPacket` does to a tracked waypoint on the locator bar, vanilla `WaypointS2CPacket.Operation`.

| Constant | Description |
|---|---|
| `TRACK` | start tracking a waypoint |
| `UNTRACK` | stop tracking a waypoint |
| `UPDATE` | update an already tracked waypoint |
