<!--
This file is part of the dosbox-automation Project.
License: GPL-2.0-or-later. Contact: dosbox-automation-project@trinity2k.net
-->

# dosbox-automation plugin for Claude

A Claude plugin that teaches Claude to drive DOS programs and games in the
[dosbox-automation](https://dosbox-automation.org) emulator: start it, type
and press keys, read the screen, mount disk images and archives, record
video and audio, run Lua scripts inside the emulator, and find values in a
game's memory.

The repository is a plugin marketplace. Install the plugin in Claude Code
with:

```
/plugin install dosbox-automation --marketplace dosbox-automation/dosbox-automation-skill
```

The plugin is in [plugins/dosbox-automation](plugins/dosbox-automation/).
Its [README](plugins/dosbox-automation/README.md) says what you need and what
it does on your machine.

To try it without installing anything, load the folder for one session:

```
claude --plugin-dir /path/to/dosbox-automation-skill/plugins/dosbox-automation
```

License: GPL-2.0-or-later, see [LICENSE](LICENSE) and [NOTICE](NOTICE).

---
dosbox-automation has been written with the help of tooled assistance, but
has been designed, reviewed and thoroughly tested and signed off by its
developer.
