# MCP for AI agents

The client can expose its scripting runtime as an MCP server, so an agent can deploy, enable, configure and read
scripts; it is off by default and turned on in the Scripts tab. The script API itself is the rest of these docs,
starting at [Your first script](../start/first-script.md) and [Sandbox and limits](limits.md).

## Turning it on

Open the **Scripts** tab, press the **MCP** button in the header — left of the console button — and tick
**Enabled**. A notification reports whether the server came up; on success the link is already in the clipboard,
and the header button stays lit while the server runs.

| Control | Effect |
|---|---|
| `Enabled` | starts or stops the server; saved and restored on the next launch |
| `Copy link` | puts the current link back in the clipboard |
| `New token` | issues a new 32-byte token and restarts; the old link stops working at once |
| `New port` | takes a free port and restarts; the link changes |

The last three appear only while the server runs. The link is `http://127.0.0.1:<port>/mcp/<token>`; port and
token are saved, so it stays the same across launches.

The server binds `127.0.0.1` only and speaks JSON-RPC over HTTP POST (MCP protocol `2025-06-18`, 1 MiB body
limit); the token in the path is the only credential it checks, and a request carrying an `Origin` header or a
foreign `Host` gets a 404.

Register the link with the agent:

```bash
claude mcp add --transport http nursultan http://127.0.0.1:PORT/mcp/TOKEN
```

## The tools

| Tool | Arguments | Returns |
|---|---|---|
| `scripts_list` | — | `scriptsDir`, `assetsDir`, and per script `id`, `file`, `name`, `description`, `enabled`, `status` (`LOADING`/`WAITING`/`READY`/`ERROR`), `phase` (`Reading`/`Compiling`/`Starting`, absent once loaded), `error` |
| `script_source` | `id` | `id`, `file`, `source` |
| `script_deploy` | `name` (1..48 chars of `A-Za-z0-9 _-`), `source` (max 256 KiB) | writes `<name>.kts` into the scripts folder; `written`, `bytes` |
| `script_delete` | `id` | unloads the script and deletes its file; `unloaded`, `deleted` |
| `script_enable` | `id` | switches the script on, as its bind does; `enabled` |
| `script_disable` | `id` | switches the script off; `enabled` |
| `script_settings` | `id` | `id`, `name`, `enabled`, `status`, `bind`, and every setting |
| `script_setting_set` | `id`, `key`, `value` | writes one setting; `kind` plus the whole `settings` list |
| `script_bind_set` | `id`, `key` (`UNKNOWN` clears the bind), `mods` (GLFW bitmask, 0 when omitted) | rebinds the script; the new `bind` |
| `console_tail` | `sinceId` (omit for the whole buffer), `limit` (default 200, capped at the 1000-line buffer) | `lines` of `id`, `time`, `level` (`INFO`/`WARN`/`ERROR`), `text`, plus `lastId`, `revision`, `dropped` |
| `game_state` | — | `inGame`, `singleplayer`, `server`, and `dimension`, `x`, `y`, `z`, `username` while in a world |

No tool plays the game: nothing here moves the player, opens a screen or clicks a block. Every tool except
`scripts_list`, `script_source` and `script_deploy` runs on the client thread and fails after 5 s if the client
does not answer.

`script_deploy` only writes the file — the watcher recompiles on its own, so the outcome shows up in the next
`scripts_list`. `console_tail` is incremental: pass the previous `lastId` as `sinceId`; a `sinceId` past the
newest id returns the whole buffer with `reset: true`, because line ids restart with the client.

## Setting value shapes

`script_settings` reports a `kind` per setting, and `script_setting_set` expects a value shaped to match it.

| Kind | Value to send |
|---|---|
| `checkbox` | `true` / `false` |
| `slider` | a number; the report carries `min`, `max`, `step` |
| `rangeSlider` | `{"min": 1, "max": 4}`; the report carries `limitMin`, `limitMax`, `step` |
| `input` | a string; the report carries `placeholder` when the setting has one |
| `colorPicker` | an ARGB integer |
| `selectable` | the `name` of one option, matched case-insensitively |
| `combo` | an array of option names; every option not listed is deselected |
| `hotkey` | `"G"` or `{"key": "G", "mods": 2}`; `mods` is a GLFW bitmask |
| `button` | any value; sending it presses the button |
| `group` | nothing — a group holds no value and the write fails |

Every reported setting carries `key`, `label`, `kind`, `visible`, `modified` and `value`; a nested setting adds
`parent` and uses a dotted key (`Toggle.Only when toggled`) for both reading and writing.

Objects and arrays are accepted either as real JSON or as a JSON string; option names come from `options[].name`,
not from `label`.
