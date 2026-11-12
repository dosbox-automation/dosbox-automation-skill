---
name: dosbox-automation
description: Use when the user wants to run, install, test, record or debug a DOS program or game in the dosbox-automation emulator, type at a DOS prompt, read a DOS screen, swap floppy or CD images, run a Lua script inside the emulator, or find and freeze a value in a game's memory.
license: GPL-2.0-or-later
---
<!--
This file is part of the dosbox-automation Project.
License: GPL-2.0-or-later. Contact: dosbox-automation-project@trinity2k.net
-->

# Driving DOS with dosbox-automation

dosbox-automation is a DOS emulator with an HTTP REST API that listens on the
user's own machine, on 127.0.0.1 by default. This skill installs nothing. The
user installs the emulator, and you talk to it by one of the routes below. The
handbook in `references/` covers every task in detail.

## Pick the route

1. **The dosbox MCP tools**, when they are in your tool list (`screen_text`,
   `input_type`, `bridge_status` and the rest). Use them first.
2. **A shell, without the tools:** `dosboxctl` from the user's
   dosbox-automation install, or `curl` against the REST API. Every handbook
   file shows both. In the Linux and Windows release packages dosboxctl
   sits beside the `dosbox-automation` binary; the AppImage runs it as
   `dosbox-automation-*.AppImage --dosboxctl ...`. The macOS package has no
   dosboxctl: use curl there.
3. **Neither:** tell the user what is missing (next section) and stop there.

Never say the tools are there when they are not in your tool list. This
skill does not bring them.

## What the user sets up

The emulator comes from the dosbox-automation releases; the user installs it.
For automated runs its primary config needs these lines in `[webserver]`
(handbook 102 and 103):

```ini
webserver_enabled = true
webserver_token_file = true
mount_allowed_bases = /home/you/dos
mount_allowed_image_roots = /home/you/dos
```

`mount_allowed_bases` covers folders; disk images, archives and disk swaps
need `mount_allowed_image_roots`. These lines apply to every start of the
emulator. Only one running emulator can own the token file: a second one
started with them and without `DOSBOX_API_TOKEN` overwrites it with its own
token.

The MCP tools come from dosbox-mcp, a separate package. The user adds it
once, in their own terminal, then starts a new Claude Code session:

```sh
claude mcp add dosbox -- uvx dosbox-mcp==0.2.4
```

It needs `uv` on the PATH. On Linux with the usual config folder the tools
find the API token by themselves. On Windows, on macOS, or with
`XDG_CONFIG_HOME` set, the user points them at the token once:

```sh
uvx dosbox-mcp==0.2.4 setup --token-file <emulator config folder>/webserver/api_token
```

Give the user these lines when they ask how to set it up; do not run them on
their behalf.

## The API token

Every `/api/v1/` call needs the 64 hex digit token in
`Authorization: Bearer <token>`. With the MCP tools the bridge handles it.
With curl or dosboxctl you have two ways (handbook 101):

- Start the emulator yourself with a token you generate into a file of your
  own (101, "From an agent").
- Use the emulator the user started: with `webserver_token_file = true` it
  writes the token to `<config dir>/webserver/api_token`.

That file is a credential on the user's machine. Ask the user before you read
it the first time, and wait for the answer. A dosboxctl call with neither
`DOSBOX_TOKEN_FILE` nor `DOSBOX_API_TOKEN` set reads that file by itself, so
the question comes before your first dosboxctl call against the user's
emulator. Send the token only to `127.0.0.1`, and never print it or copy it
into the conversation. Prefer dosboxctl with `DOSBOX_TOKEN_FILE`: it never
puts the token on a command line. curl with `$(cat <file>)` does, and other
users of the machine can see it while curl runs.

An emulator you start with your own `DOSBOX_API_TOKEN` writes no token file,
so the MCP tools cannot reach it; drive it with dosboxctl or curl. If the
user's emulator is already running, give yours another port
(`--set webserver_port=<n>`, and `DOSBOX_PORT=<n>` for dosboxctl, 102). On a
taken port the log says "failed to bind", the API stays off, and your calls
reach the user's emulator and get 401.

Never open the emulator's Cheat Workbench page, `/tools/cheat-workbench.html`
or the old `/cheat-workbench.html`, with any tool: the engine writes the API
token into it.

## Working rules

- After every input, read the screen before the next step. DOS programs react
  to timing; check what happened instead of assuming it.
- Read the screen as text first (`screen_text`, `GET /video/text`,
  `dosboxctl screen`). It costs far fewer tokens than a screenshot. Take a
  screenshot only in graphics modes, where the text comes back empty.
- Wait for a screen state, never for a fixed number of keys or seconds.
- Folders mount only below the conf anchor or `mount_allowed_bases`; disk
  images, archives and swaps only below the conf anchor or
  `mount_allowed_image_roots`. The conf anchor is the folder of the last conf
  loaded; an archive or game folder named on the emulator's command line
  counts too, and for folders only so does the emulator's `drives` folder (a
  `drives` folder in the folder it starts in replaces the bundled one). A
  refusal on the DOS screen says why (103).
- Stop with `dosbox_shutdown` or `POST /api/v1/dosbox/shutdown`. Never kill
  the emulator by name (`pkill dosbox`): other sessions on the machine may run
  one.
- For real work use the emulator the user started. One started with the
  `bridge_start` tool runs on the bridge's own temporary config, so the
  user's `mount_allowed_bases` and `mount_allowed_image_roots` do not apply;
  on macOS it reads the user's own config instead and does not come up.
  `bridge_stop` ends it and deletes its temporary folder, recordings
  included: tell the user before you stop it.

## MCP tools and the routes behind them

The handbook speaks in REST routes and dosboxctl commands. This table maps
them to the tools.

| Need | Tools | Route in the handbook |
|---|---|---|
| Machine state | `dosbox_status`, `bridge_status` | `/status`, `/program/state` |
| Type, press keys, mouse | `input_type` (`\n` is Enter), `input_key`, `input_sequence` (`KBD_` names) | `/input/type`, `/input/sequence` (201) |
| Read the screen | `screen_text`, `screen_capture` (`mode`: `raw` or `rendered`), `screen_info` | `/video/text`, `/video/frame`, `/video/frame/info` (201) |
| Record | `video_capture_start`/`_stop`/`_status`, `video_capture_compression`/`_set`, `audio_capture_start`/`_stop`/`_status` | `/capture/*` (204) |
| Lua inside the emulator | `script_run`, `script_status`, `script_stop` | `/script/*` (301) |
| Memory, registers, ports | `mem_read`, `mem_write`, `mem_search`, `freeze_set`/`_list`/`_clear`, `dos_memory_map`, `cpu_write_register`, `port_read`, `port_write` | `/memory/*`, `/cpu/register`, `/io/port`, `/dos/internals` (203, 303) |
| Sound level | `mixer_status`, `mixer_volume`, `mixer_channel_volume`, `mixer_mute`, `mixer_unmute` | `/mixer/*` (601) |
| Stop | `dosbox_shutdown` | `/dosbox/shutdown` |

No tool swaps a disk, sets the speed or locks the mounts. With the tools,
swap disks from Lua (`dosbox.drive_swap("a", "disk2.img")` through
`script_run`, 301); everything else goes through REST or dosboxctl
(`/drive/swap`, `/cpu/cycles`, `/mount/lock`).

## The handbook

Read the matching file before the task. Each one opens with the same short
block (enable the API, the token, the base URL, how to stop) and then gives
the commands.

| Question | File in `references/` |
|---|---|
| Set up, start, talk to it, stop it | dosbox-automation-101-setup-running.md |
| A conf for automated runs, where settings come from | dosbox-automation-102-recommended-settings.md |
| Mount a folder, floppy or CD image, swap disks, why MOUNT is refused | dosbox-automation-103-mounting-guide.md |
| Run a game from zip, rar, 7z or iso | dosbox-automation-104-working-with-archive-mounts.md |
| Type, press keys, click, read the screen, screenshots | dosbox-automation-201-basic-usage.md |
| Install and start a game end to end | dosbox-automation-202-how-to-run-a-game.md |
| Registers, memory, ports, search, freeze | dosbox-automation-203-how-to-debug-a-game.md |
| Record video, audio or input | dosbox-automation-204-how-to-record-video-audio.md |
| Lua scripts inside the engine and their limits | dosbox-automation-301-lua-scripting.md |
| Record a person's input and replay it, timed key sequences | dosbox-automation-302-input-recording-replay.md |
| Find a game's value in memory (lives, ammo) and hold it | dosbox-automation-303-memory-trainer.md |
| Every route, Lua function and setting | dosbox-automation-601-reference-guide.md |
