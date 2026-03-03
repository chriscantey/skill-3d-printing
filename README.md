# 3D Printing Skill for Claude Code

A skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that gives your AI assistant context about 3D printing workflows, OpenSCAD patterns, and design principles. Drop it into your skills folder and start designing printable objects through conversation.

This grew out of a few months of experimenting with AI-assisted 3D print design. It covers what worked for me: using OpenSCAD as the primary format, starting from 2D descriptions, iterating with test prints, and keeping things parametric.

> **Work in progress.** This reflects my experience so far and probably has gaps. If you know more about OpenSCAD, 3D modeling, or mechanical engineering, I'd love for you to improve on it and share what you learn.

## Prerequisites

You need [OpenSCAD](https://openscad.org/) installed for your assistant to render models and export STL files.

**Debian/Ubuntu:**
```bash
sudo apt install openscad
```

**macOS (Homebrew):**
```bash
brew install openscad
```

**Other platforms:** See [openscad.org/downloads](https://openscad.org/downloads.html)

For headless preview rendering (generating PNG images without a display), you may also need `xvfb`:
```bash
sudo apt install xvfb
```

If your assistant has permission to install packages, it can handle this for you. Otherwise, install manually before starting your first project.

## What's Inside

**Skill file** (`SKILL.md`) with printer specs template and workflow routing.

**Workflows:**

- **Design** - The core design workflow. How to go from concept to parametric OpenSCAD model to export-ready STL. Covers 2D-first approach, standard file structure, CLI rendering, working with existing STL files, manifold geometry rules, functional parts guidance, and versioning.

- **GalleryView** - Visual gallery pages for tracking your designs. Uses Three.js for interactive STL viewers in the browser, OpenSCAD CLI for multi-angle preview renders, and assembles them into a viewable page. Works with any web server, a local Python HTTP server, or a portal setup.

## Install

### Option A: Manual

1. Copy the `3DPrinting/` folder into `~/.claude/skills/`:
   ```bash
   cp -r 3DPrinting/ ~/.claude/skills/3DPrinting/
   ```

2. Open `~/.claude/skills/3DPrinting/SKILL.md` and fill in your printer specs where you see `[placeholder]` values.

3. Your assistant should pick up the skill automatically. Claude Code detects new skills in the skills folder without a restart.

### Option B: AI-Assisted

Clone the repo and point your Claude Code assistant at the install instructions:

```bash
cd ~ && git clone https://github.com/chriscantey/skill-3d-printing.git
```

Then tell your assistant:

```
Read ~/skill-3d-printing/INSTALL.md and follow its instructions.
```

The assistant should ask you about your printer specs and fill everything in for you.

## Compatibility

This skill works with:

- **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** - Drop the `3DPrinting/` folder into `~/.claude/skills/` and it loads automatically.
- **[PAI](https://github.com/danielmiessler/PAI)** (Personal AI Infrastructure) - Same skills folder structure. If you're using the [PAI Companion](https://github.com/chriscantey/pai-companion), the portal gives you a natural place to serve gallery pages for your designs.

## Context

For more background on how this came together: [AI-Assisted 3D Printing: What Works and What Doesn't](https://chriscantey.com/posts/2026-02-17-ai-assisted-3d-printing/)

## License

Use it however you like. If you build on it and learn something, sharing back is appreciated but not required.
