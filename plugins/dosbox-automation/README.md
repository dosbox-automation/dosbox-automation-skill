<!--
This file is part of the dosbox-automation Project.
License: GPL-2.0-or-later. Contact: dosbox-automation-project@trinity2k.net
-->

# dosbox-automation for Claude

This plugin teaches Claude to drive DOS programs and games in the
[dosbox-automation](https://dosbox-automation.org) emulator. Claude learns
to start and stop the emulator, type at the DOS prompt and press keys, read
the screen, mount folders, disk images and archives, swap floppies during an
install, record video and audio, run Lua scripts inside the emulator, and
find and hold values in a game's memory. The plugin is a skill and the
emulator's agent handbook, all of it Markdown.

dosbox-automation is a DOS emulator of its own. It began as a fork of
dosbox-staging, which descends from DOSBox, and has been developed
independently since version 0.85.0. This plugin is published by the
dosbox-automation project and is not affiliated with the DOSBox project.

The emulator runs on your computer, so the skill helps in Claude Code on your
own machine. Chat on claude.ai cannot reach a program on your computer.

## Install

In Claude Code:

```
/plugin install dosbox-automation --marketplace dosbox-automation/dosbox-automation-skill
```

## What you need

- **dosbox-automation**, the emulator, installed on your computer. This
  plugin does not bring it; get it from the
  [releases page](https://github.com/dosbox-automation/dosbox-automation/releases).
- These lines in the `[webserver]` section of the emulator's primary config:

  ```ini
  webserver_enabled = true
  webserver_token_file = true
  mount_allowed_bases = /home/you/dos
  mount_allowed_image_roots = /home/you/dos
  ```

  The last two lines name where DOS may mount from: `mount_allowed_bases`
  for folders, `mount_allowed_image_roots` for disk images, archives and
  disk swaps. Besides these, the emulator allows only the folder of the last
  config it loaded, an archive or game folder named on its command line, and,
  for folders only, its own `drives` folder (a `drives` folder in the folder
  it starts in replaces the bundled one); it refuses everything else. These
  settings apply to every start of the emulator, and only one running
  emulator can own the token file: a second one started without
  `DOSBOX_API_TOKEN` overwrites it with its own token.

Claude then talks to the emulator through its REST API with `curl`, or with
`dosboxctl`, the command-line client that comes with the emulator.

If you want Claude to use dedicated tools instead of shell commands, add the
dosbox-mcp bridge once, in your own terminal (it needs
[uv](https://docs.astral.sh/uv/)):

```
claude mcp add dosbox -- uvx dosbox-mcp==0.2.4
```

The skill uses those tools when they are there. On Windows and macOS, or with
`XDG_CONFIG_HOME` set, tell the bridge once where the emulator keeps its
token:

```
uvx dosbox-mcp==0.2.4 setup --token-file <emulator config folder>/webserver/api_token
```

dosbox-mcp is a separate program with its own
[README](https://github.com/dosbox-automation/dosbox-mcp) and privacy
policy. This plugin does not install it.

## Use it

Start the emulator, then ask Claude to run a program, type a command, read
what is on the screen, record a session or find a value in a game's memory.

These three requests need no other software: they run at the emulator's own
Z:\> prompt, so they also make a good first test.

- "Run DIR Z: and tell me which programs are on drive Z:."
- "Record a short video while you type VER at the DOS prompt, then stop the
  recording and tell me whether it ended cleanly."
- "Read the BIOS keyboard flags at 0040:0017 and tell me whether Num Lock
  is on."

Claude reads the screen as text whenever the program runs in a text mode,
which costs far fewer tokens than a screenshot. Video and audio capture
records the emulator's own output, what the DOS program draws and plays;
this plugin contains no generative model and creates no media of its own.

## What it does on your machine

- **The plugin itself:** Markdown only. It contains no programs, starts no
  server, and has no network access of its own.
- **Commands Claude runs:** following the skill, Claude runs `curl` or
  `dosboxctl` against the emulator on 127.0.0.1, and it may start the
  emulator for you, which also uses `python3` (to make a token), `chmod`,
  `setsid` and `nohup`, and `kill` on that emulator's own process if it does
  not shut down. Claude Code asks you before it runs a command, unless
  you have allowed that command in its permission settings or run Claude Code
  in auto mode.
- **Your API token:** every call to the emulator needs its API token. The
  skill tells Claude to use the token file the emulator writes
  (`<config dir>/webserver/api_token`), or, when Claude starts the emulator
  itself, a token Claude generates into a file in your working folder. That
  file is a credential on your machine. The skill tells Claude to ask you
  before it reads it the first time and wait for your answer, to send the
  token only to 127.0.0.1, and never to print it or put it into the
  conversation. It prefers `dosboxctl`, which keeps the token off command
  lines; a `curl` call carries the token on its command line, where other
  users of the same machine can see it while the call runs.
- **The Cheat Workbench page:** the emulator serves its tool page at
  `/tools/cheat-workbench.html` without a token and writes the API token into
  it for any program on your machine that asks over 127.0.0.1. Set
  `webserver_tool_token = prompt` if you do not want that. The skill tells
  Claude never to open that page, or its old address
  `/cheat-workbench.html`, with any tool.
- **Files Claude writes:** in your working folder, when Claude starts the
  emulator itself, the token file, a config file, the emulator's log and its
  process number, and the folders and files a task needs, such as a
  downloaded game. Screenshots go where Claude saves them, recordings into
  the emulator's capture folder. DOS programs write into the folders Claude
  mounts (an installer fills the folder mounted as its drive). Writes to a
  mounted archive go into a separate save folder, by default in the
  emulator's data folder, or one beside the archive if it already exists
  there, or where the `archive_sidecar_path` setting or a mount option puts
  it; the archive itself stays unchanged. RAR 5, 7z and tar archives, disc
  images and read-only disk images taken out of any archive, archives inside
  archives, and members larger than `archive_memory_limit` (16 MiB by
  default) that a DOS program opens are copied into the emulator's cache
  folder (or the folder the `archive_cache_path` setting names) and stay
  there until the cache is trimmed or emptied. Every mounted archive also
  leaves a small record with its path there. A Lua script run with debugging
  on writes a trace into the emulator's log folder.
- **Your typing:** Claude can record the keys you press and the mouse moves
  you make in the emulator's window and read the recording back, for example
  to replay an install you did by hand.
- **What Claude can reach:** everything inside the emulated DOS machine.
  Claude reaches host files only through MOUNT and disk swaps, below
  `mount_allowed_bases` (folders), `mount_allowed_image_roots` (images and
  archives), the folder of the last config the emulator loaded, an archive
  or game folder named on its command line, or its `drives` folder (folders
  only), and through the audio path below.
  For an emulator you started without a config file of your own, that last
  config is the primary one, so its folder, which also holds the token file
  and the recordings, can be mounted. The emulator's API listens on
  127.0.0.1 by default and asks for the token on every API request; its
  start page, the API documentation and the Cheat Workbench page open
  without one.
- **Recordings:** the capture routes write numbered files into the
  emulator's capture folder. An audio recording started through the REST API
  or a Lua script can take a file path of its own. The emulator refuses system
  folders there, but writes anywhere else your user account can write and
  replaces a file that already exists.
- **Your conversation:** the screen text, screenshots, memory contents and
  command output that Claude asks for become part of your conversation, like
  any other tool result.
- **Accounts:** the plugin sends no telemetry, needs no account and stores
  no credentials.

## Links

- Emulator: https://github.com/dosbox-automation/dosbox-automation
- Project website: https://dosbox-automation.org
- dosbox-mcp, the optional bridge: https://github.com/dosbox-automation/dosbox-mcp
- Privacy policy: [PRIVACY.md](PRIVACY.md)
- License: GPL-2.0-or-later.

---
dosbox-automation has been written with the help of tooled assistance, but
has been designed, reviewed and thoroughly tested and signed off by its
developer.
