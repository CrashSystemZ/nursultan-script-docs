# MCP for AI agents

**This page is written for AI coding agents — Claude, Codex and anything else that speaks MCP.** If you are a
human reading it, the only part you need is "Turning it on" below; the rest tells your agent what it can do
once you hand it the link.

The client can expose its scripting runtime as an MCP server. An agent connected to it writes a script,
deploys it, switches it on and reads the in-game console — the whole loop, without you copying files or
retyping errors.

## Turning it on

In the client menu open the **Scripts** tab and press the **MCP** button in the header, left of the console
button. A small window opens; tick **Enabled**. A notification tells you whether the server came up, and on
success the link is already in your clipboard. The header button lights up while the server is on.

Three more buttons appear in that window once it is running:

* **Copy link** — puts the link back in your clipboard.
* **New token** — issues a fresh token. The old link stops working at once; use this if you pasted it
  somewhere you should not have.
* **New port** — picks a different port, for when something else has taken the old one.

Both of the last two change the link, so update it in your agent afterwards.

Give that link to your agent:

```bash
claude mcp add --transport http nursultan http://127.0.0.1:PORT/mcp/TOKEN
```

The link is stable — the port and token are saved, so you add the server once and never touch it again. The
tick is saved too: leave it on and the server comes back by itself the next time you start the client, with
the same link and without stealing your clipboard. Untick it to stop the server and to stop it coming back.

Two things worth knowing. The link contains a token, so **treat it like a password** — anyone who has it can
run scripts in your client while the server is on. And the server only listens on `127.0.0.1`, so nothing
outside your machine can reach it.

## What the agent gets

Eleven tools. Everything below is what the agent sees.

| Tool | Arguments | What comes back |
|---|---|---|
| `scripts_list` | — | every script: `id`, `name`, `enabled`, `status`, and `error` when it failed to compile |
| `script_source` | `id` | the current source of that script |
| `script_deploy` | `name`, `source` | writes `<name>.kts` into the scripts folder; the script recompiles at once |
| `script_delete` | `id` | unloads the script and deletes its file |
| `script_enable` | `id` | switches the script on, exactly like pressing its bind |
| `script_disable` | `id` | switches it off |
| `script_settings` | `id` | the load `status`, the bind, and every setting with its kind, value, limits and options |
| `script_setting_set` | `id`, `key`, `value` | writes one setting; pressing a `button` setting runs it |
| `script_bind_set` | `id`, `key`, `mods` | rebinds the script, or clears it with `UNKNOWN` |
| `console_tail` | `sinceId`, `limit` | console lines, newest last, plus `lastId` to pass in next time |
| `game_state` | — | `inGame`, `singleplayer`, dimension, player position |

### Settings

`script_settings` reports a `kind` per setting, and `script_setting_set` expects a value shaped to match it:

| kind | value to send |
|---|---|
| `checkbox` | `true` / `false` |
| `slider` | a number inside `min`..`max` |
| `rangeSlider` | `{"min": 1, "max": 4}` |
| `input` | a string |
| `colorPicker` | an ARGB integer |
| `selectable` | the `name` of one option |
| `combo` | an array of option names — everything not listed is deselected |
| `hotkey` | `"G"`, or `{"key": "G", "mods": 2}` |
| `button` | any value; sending it runs the button |

The three shapes that are not a bare scalar — `rangeSlider`, `combo` and the object form of `hotkey` — are
accepted either as real JSON or as a JSON string, whichever your client produces.

A setting nested under a checkbox is reported with a dotted key and a `parent`, and you write it with that
same dotted key:

```json
{"key": "Fancy",       "kind": "checkbox", "value": true}
{"key": "Fancy.Alpha", "kind": "slider",   "value": 0.5, "parent": "Fancy"}
```

Read before you write: a setting can be invisible (`visible: false`) because another setting turned it off,
and options carry their own `name` you must use verbatim.

## The loop

```
script_deploy  ->  scripts_list  ->  script_enable  ->  console_tail  ->  fix  ->  script_deploy
```

`script_deploy` only writes the file. Compilation happens right after, on its own — so call `scripts_list`
next and look at `status`. `ERROR` means it did not compile and `error` holds the message; the script cannot
be enabled until that is fixed.

`console_tail` is incremental. The first call returns the buffer and a `lastId`; pass that back as `sinceId`
and you only get what is new. That is the cheapest way to watch a script without re-reading everything.

Line ids restart when the human restarts the game. You do not have to handle it: a cursor ahead of the newest
line is treated as a fresh start, and the answer carries `"reset": true` so you know the numbering began again
rather than the console having gone quiet.

## What you cannot do

**You cannot play the game.** Walking around, right-clicking a villager, opening a chest — no. When a test
needs that, either ask the human to do it, or have the script set the world up itself — a script can run
server commands:

```kotlin
chat.sendToServer("/summon wandering_trader ~ ~ ~3")
chat.sendToServer("/setblock ~ ~ ~2 minecraft:suspicious_gravel{item:{id:\"minecraft:emerald\",count:1}}")
```

**Check `game_state` before you do that.** Commands like these only work in singleplayer with cheats on; on a
real server they will be refused at best, and at worst they get the human flagged for cheating. If
`singleplayer` is false, ask instead of sending.

**Compiling is not running.** A script that compiles can still die the moment a handler fires — an unresolved
symbol inside `on<...>` only links when that handler first runs. So never call a script verified because
`status` is `READY`; enable it and read the console.

**`onEnable` runs before handlers are subscribed.** A packet sent from inside `onEnable` will not be seen by
that script's own `on<PacketSendEvent>`. Send from a tick handler instead.

## Writing the script itself

Everything about the language and the API is in the rest of these docs — start with
[Your first script](../start/first-script.md), and note that
[the sandbox](limits.md) blocks reflection, files and the network, so a script that reaches for them fails to
load rather than misbehaving.

Say which API version you need on the first line; the details are in [API versions](api-versions.md):

```kotlin
requireApi(1)
```
