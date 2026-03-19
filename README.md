# Canvas Apps Tools

> AI skills for Power Apps Canvas Apps — generate, deploy, and integrate.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](.claude-plugin/plugin.json)

## Skills

| Skill | Invoke | Description |
|-------|--------|-------------|
| **canvas-apps-ui-gen** | `/canvas-apps-ui-gen` | Generate paste-ready YAML from mockups, screenshots, or text descriptions |

## Requirements

- [Claude Code](https://claude.ai/claude-code) (latest version)
- Git

## Install

**Option 1 — Plugin directory (recommended):**

```powershell
# Windows — clone to a permanent location
git clone https://github.com/ToluVictor/canvas-apps-tools.git "$env:USERPROFILE\claude-skills\canvas-apps-tools"

# Add to Claude Code (run once — persists permanently)
claude config add addDir "$env:USERPROFILE\claude-skills\canvas-apps-tools\skills"
```

```bash
# macOS / Linux
git clone https://github.com/ToluVictor/canvas-apps-tools.git ~/claude-skills/canvas-apps-tools
claude config add addDir ~/claude-skills/canvas-apps-tools/skills
```

**Option 2 — Per-session plugin dir:**

```bash
claude --plugin-dir /path/to/canvas-apps-tools
```

**Option 3 — skills.sh:**

```bash
npx skills add ToluVictor/canvas-apps-tools
```

## Upgrade

```powershell
# Windows
cd "$env:USERPROFILE\claude-skills\canvas-apps-tools"
git pull
```

```bash
# macOS / Linux
cd ~/claude-skills/canvas-apps-tools
git pull
```

No reinstall needed — new skills and updates are available immediately after `git pull`.

## Usage

### Generate from a screenshot or mockup

```
/canvas-apps-ui-gen C:\path\to\mockup.png
```

Or paste an image directly into the chat, then type:

```
/canvas-apps-ui-gen
```

### Build from a text description

```
/canvas-apps-ui-gen
```

Select "Build from scratch" when prompted, then describe the screen you want.

### Improve an existing Canvas App screen

Take a screenshot of your current screen, paste it into the chat, and invoke:

```
/canvas-apps-ui-gen
```

Select "Improvement" when prompted.

## How It Works

The skill uses a 4-phase multi-agent pipeline:

1. **Mode detection + image loading** — determines your intent and loads the design input
2. **Analysis + questions** — describes the layout, checks Canvas Apps compatibility, asks focused pre-generation questions
3. **Parallel specialist agents** — three agents run simultaneously:
   - **Layout + Sizing** — dimensions, padding, alignment, scroll containers
   - **Controls** — correct control types, variants, semantic properties
   - **Styling** — colors, typography, borders, hover/pressed/disabled states
4. **Assembly + QA** — merges annotations, self-validates against PA2105/PA2108/PA1001 rules, writes clean YAML

Output is written to `skills/canvas-apps-ui-gen/output/` and displayed inline if under 400 lines.

## Pasting into Power Apps Studio

1. Open the generated `.yaml` file, press **Ctrl+A** then **Ctrl+C**
2. In PA Studio, right-click the target screen or container in the tree view
3. Select **Paste code**

## Notes

- Optimized for Claude Code — uses multi-agent orchestration features specific to Claude Code
- Sized for tablet canvas (1366px wide) by default
- Generates Classic controls by default; Modern (Fluent 2) controls available on request
- Output YAML is paste-ready — no manual edits required in most cases

## License

MIT — see [LICENSE](LICENSE)

---

*By Tolu Victor Sanwoolu*
