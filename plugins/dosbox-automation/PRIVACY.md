<!--
This file is part of the dosbox-automation Project.
License: GPL-2.0-or-later. Contact: dosbox-automation-project@trinity2k.net
-->

# Privacy policy of the dosbox-automation plugin

Last changed on 2026-11-12. This policy covers the dosbox-automation plugin
for Claude, which the dosbox-automation project publishes. You can reach us
at dosbox-automation-project@trinity2k.net.

## What we collect

We collect nothing. The plugin consists of Markdown files: a skill and the
emulator's agent handbook. It contains no programs, sends no data to the
dosbox-automation project or to anyone else, and has no network access of
its own. It has no telemetry, no analytics, no crash reports and no accounts.

## What happens on your computer

The plugin tells Claude how to work with the dosbox-automation emulator on
your computer. Following it, Claude runs commands such as `curl` and
`dosboxctl` against the emulator's API on a loopback address such as
127.0.0.1, and it may start the emulator for you. Claude Code asks you before
it runs a command unless you have allowed that command in its settings or run
Claude Code in auto mode.

Every call to the emulator's API needs its API token. The plugin tells Claude
to read the token from the file the emulator writes, or, when Claude starts
the emulator itself, from a token file Claude creates in your working folder.
It tells Claude to ask you before reading the emulator's token file the first
time and to wait for your answer, to send the token only to the emulator's
loopback address, never to show it in the conversation, and never to open
the emulator's Cheat Workbench page with any tool, since the emulator serves
that page with the token written into it.

When Claude starts the emulator itself, the token file, a config file, the
emulator's log and its process number stay in your working folder until you
delete them, together with the files a task needs, such as a downloaded
game and screenshots. DOS programs write into the folders Claude mounts.
Writes to a mounted archive go into a save folder; archives in some formats,
disc and disk images taken out of archives, archives inside archives and
large archive members that a DOS program opens are copied into the
emulator's cache folder, which also keeps a small record of each mounted
archive's path; and a Lua script run with
debugging on writes a trace into the emulator's log folder. Recordings go
into the emulator's capture folder, or to a path that a Lua script or an API
call names, as the plugin's README describes. Claude can also record the keys
you press and the mouse moves you make in the emulator's window and read that
recording. Everything stays on your computer.

## What goes into your conversation

The screen text, screenshots, memory contents, command output and other
results that Claude asks for become part of your conversation with Claude,
the same way any tool result does. Anthropic handles your conversation under
its own privacy policy. The dosbox-automation project never sees it.

## Other software

The emulator and the optional dosbox-mcp bridge are separate programs that
you install yourself. They are not part of this plugin; dosbox-mcp has its
own privacy policy in its repository.

## Children

The plugin is a tool for developers and is not intended for people under 18.
It drives whatever DOS software you run in the emulator, and some DOS games
were made for adults.

## Changes to this policy

When this policy changes, the new text replaces this file, and the
repository's history keeps every earlier version.
