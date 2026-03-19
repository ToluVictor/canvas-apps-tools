# Canvas Apps Tools

> Generate paste-ready Power Apps Canvas App YAML from screenshots, mockups, or text — directly inside Claude Code.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](.claude-plugin/plugin.json)


## What it does

This plugin gives Claude Code a single skill — `canvas-apps-ui-gen` — that acts as a Power Apps Canvas App developer. Point it at a mockup, describe a screen, or paste a screenshot of an existing app and it produces YAML you can paste straight into Power Apps Studio.

- **Replicate** a UI mockup or wireframe as Canvas App YAML
- **Improve** an existing Canvas App screen (modernize spacing, upgrade controls, add hover/focus states)
- **Build from scratch** from a text description alone
- Enforces Power Apps YAML rules (PA2105 / PA2108 / PA1001) automatically
- Saves output to `skills/canvas-apps-ui-gen/output/` and displays it inline when small (< 400 lines)


## Requirements

- [Claude Code](https://claude.ai/claude-code) (v1.0.33 or later — run `claude --version` to check)
- Git (for installation methods 1 and 3)
- Power Apps Studio (to paste and run the generated YAML)


## Installation

Choose whichever method fits your workflow. All three end with the same command: `/canvas-apps-ui-gen`.

---

### Method 1 — Marketplace (recommended)

Add this repo as a marketplace, then install the plugin:

```shell
/plugin marketplace add ToluVictor/canvas-apps-tools
```

Then install the plugin:

```shell
/plugin install canvas-apps-ui-gen@ToluVictor/canvas-apps-tools
```

Activate without restarting:

```shell
/reload-plugins
```

Run the skill:

```shell
/canvas-apps-ui-gen
```

---

### Method 2 — Individual skill install (skills CLI)

If you only want this one skill without installing the full plugin:

```shell
npx skills add ToluVictor/canvas-apps-tools --skill canvas-apps-ui-gen
```

Run the skill:

```shell
/canvas-apps-ui-gen
```

> This uses the [skills.sh](https://skills.sh) Agent Skills CLI. The skill is installed into your Claude Code skills folder and is available immediately in any new session.

---

### Method 3 — Local clone

Clone the repo to your machine, then load it as a local plugin.

```bash
git clone https://github.com/ToluVictor/canvas-apps-tools.git
```

**Load for a single session:**

```bash
claude --plugin-dir /path/to/canvas-apps-tools
```

**Install persistently for yourself (all projects):**

```bash
claude plugin install /path/to/canvas-apps-tools --scope user
```

**Install for everyone on a specific project:**

```bash
claude plugin install /path/to/canvas-apps-tools --scope project
```

Run the skill:

```shell
/canvas-apps-ui-gen
```

**Keeping it up to date:**

```bash
cd /path/to/canvas-apps-tools
git pull
```

Then run `/reload-plugins` inside Claude Code to pick up the changes.

---


## Usage

Once installed, run `/canvas-apps-ui-gen` and Claude will ask which mode you want:

### From a screenshot or mockup

Pass a file path as an argument:

```shell
/canvas-apps-ui-gen C:\path\to\mockup.png
```

Or paste the image directly into the chat and run `/canvas-apps-ui-gen` — Claude detects it automatically.

### From a text description

Run `/canvas-apps-ui-gen` with no arguments, choose **Build from scratch**, and describe what you want:

> "Sales order entry with a search bar, a data grid, and a totals row at the bottom"

### Improve an existing screen

Paste a screenshot of your current Power Apps screen into the chat, run `/canvas-apps-ui-gen`, and choose **Improvement**. Describe the changes you want and Claude will modernize the layout, controls, and styling.


## Paste into Power Apps Studio

When generation finishes, Claude shows a link to the output file and paste instructions. The general flow is:

1. Open the generated YAML file in `skills/canvas-apps-ui-gen/output/`
2. Press `Ctrl+A` then `Ctrl+C` to copy everything
3. In Power Apps Studio, right-click the target screen or container in the tree view
4. Select **Paste code**
5. Validate and adjust as needed


## How it works

The skill runs a multi-agent pipeline internally:

1. Detects input mode and parses the image or text
2. Describes the layout and asks a few quick questions before generating
3. Runs three specialist agents in parallel — layout & sizing, controls, and styling
4. An assembly agent merges their outputs, runs QA checks, and writes the final YAML


## Troubleshooting

| Symptom | Fix |
|---|---|
| `/canvas-apps-ui-gen` not recognized | Run `/reload-plugins` then try again; verify the install with `/plugin` → Installed tab |
| No output file appears | Check `skills/canvas-apps-ui-gen/output/`; rerun with a clearer description or smaller scope |
| YAML shows errors in Power Apps Studio | Copy from the file directly (do not retype); if errors persist, rerun with a simpler prompt and fewer nested controls |
| PA2105 version warning | Claude will self-heal the version number if you mention the warning in the same session |


## Contributing

- Fork this repo
- Add or update content under `skills/canvas-apps-ui-gen/`
- Submit a pull request


## License

MIT — see [LICENSE](LICENSE)

---

*By Tolu Victor Sanwoolu*
