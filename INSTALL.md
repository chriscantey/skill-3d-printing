# 3D Printing Skill: Setup Instructions

> **For AI Assistants.** Follow these phases to install the 3D printing skill. Each phase is independently verifiable. If interrupted, resume from any phase.

**Prerequisites:**
- Claude Code installed and working
- A `~/.claude/skills/` directory exists (standard Claude Code setup)

---

### Phase 1: Check Dependencies

OpenSCAD is required for rendering models and exporting STL files.

**Steps:**
1. Check if OpenSCAD is installed:
   ```bash
   which openscad && openscad --version
   ```

2. If not installed, tell the user:
   > OpenSCAD needs to be installed for 3D model rendering. On Debian/Ubuntu: `sudo apt install openscad`. On macOS with Homebrew: `brew install openscad`. See https://openscad.org/downloads.html for other platforms.

   If the user gives you permission to install it, run the appropriate command for their platform.

3. For headless preview rendering, xvfb may also be needed:
   ```bash
   which xvfb-run || echo "xvfb not found (needed for headless OpenSCAD rendering)"
   ```
   Install if needed: `sudo apt install xvfb` (Debian/Ubuntu).

**Verification:**
```bash
which openscad >/dev/null 2>&1 && echo "PASS" || echo "FAIL: OpenSCAD not found"
```

---

### Phase 2: Gather Printer Specs

Ask the user about their 3D printer. You need these values to fill in the skill file.

**Ask the user:**

1. **What printer do you have?**
2. **What's your build volume?** (X x Y x Z in mm)
3. **What nozzle diameter are you using?** (Most common: 0.4mm)
4. **What's your default layer height?** (Most common: 0.2mm)
5. **What layer height range do you use for fine detail?**

If the user doesn't know exact values, look them up based on their printer model.

**Save the answers for Phase 4.**

**Verification:** You have all 5 values from the user.

---

### Phase 3: Copy Skill Folder

Copy the skill into the Claude Code skills directory.

**Steps:**
1. Copy the `3DPrinting/` folder:
   ```bash
   cp -r 3DPrinting/ ~/.claude/skills/3DPrinting/
   ```

**Verification:**
```bash
test -f ~/.claude/skills/3DPrinting/SKILL.md && echo "PASS" || echo "FAIL"
```

---

### Phase 4: Fill In Printer Specs

Open `~/.claude/skills/3DPrinting/SKILL.md` and replace the placeholder values in the Printer Specifications section with the user's answers from Phase 2.

**Replace these lines:**
```
| Printer | [Your printer model] |
| Build Volume | [X] x [Y] x [Z] mm |
| Nozzle | [diameter] mm |
| Default Layer Height | [height] mm |
| Fine Detail Layer | [range] mm |
```

**With the user's actual values.**

**Verification:**
```bash
grep -c "\[Your\|\\[X\]\|\\[Y\]\|\\[Z\]\|\\[diameter\]\|\\[height\]\|\\[range\]" ~/.claude/skills/3DPrinting/SKILL.md | awk '{print ($1 == 0) ? "PASS" : "FAIL: " $1 " placeholders remaining"}'
```

---

### Phase 5: Verify Skill Loads

Confirm the skill file has valid frontmatter and is in place.

**Steps:**
1. Check frontmatter:
   ```bash
   head -5 ~/.claude/skills/3DPrinting/SKILL.md
   ```
   Should show `---` delimiters with `name:` and `description:` fields.

2. Check the full skill structure:
   ```bash
   find ~/.claude/skills/3DPrinting/ -type f
   ```
   Should show `SKILL.md` and files under `Workflows/`.

Claude Code automatically detects new skills in the skills folder. No restart is needed.

**Verification:**
```bash
head -1 ~/.claude/skills/3DPrinting/SKILL.md | grep -q "^---" && echo "PASS" || echo "FAIL"
```

---

### Final Verification

```bash
echo "=== 3D Printing Skill Verification ==="
echo ""

echo -n "OpenSCAD installed: "
which openscad >/dev/null 2>&1 && echo "PASS" || echo "FAIL (install with: sudo apt install openscad)"

echo -n "Skill file exists: "
test -f ~/.claude/skills/3DPrinting/SKILL.md && echo "PASS" || echo "FAIL"

echo -n "Workflows exist: "
test -f ~/.claude/skills/3DPrinting/Workflows/Design.md && test -f ~/.claude/skills/3DPrinting/Workflows/GalleryView.md && echo "PASS" || echo "FAIL"

echo -n "Frontmatter valid: "
head -1 ~/.claude/skills/3DPrinting/SKILL.md | grep -q "^---" && echo "PASS" || echo "FAIL"

echo -n "Printer specs filled in: "
grep -c "\[Your\|\\[X\]\|\\[Y\]\|\\[Z\]\|\\[diameter\]\|\\[height\]\|\\[range\]" ~/.claude/skills/3DPrinting/SKILL.md | awk '{print ($1 == 0) ? "PASS" : "FAIL: placeholders remaining"}'

echo ""
echo "=== Verification Complete ==="
```

**Tell the user:**
> The 3D printing skill is installed. Your assistant now has context about your printer, OpenSCAD workflows, and a gallery view workflow for visualizing your designs. Try describing something you want to print.
