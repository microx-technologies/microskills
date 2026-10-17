# microskills

Collection of **Official AI Agent Skills** (Claude Code, Cursor, Copilot, Antigravity, etc.) for the **microx** ecosystem.

## 📦 Installation with `npx skills`

Install skills directly into your project workspace using the open skills CLI:

### Install `microform` skill
```bash
# Via --skill flag
npx skills add microx-technologies/microskills --skill microform

# Or pointing directly to the skill subdirectory
npx skills add microx-technologies/microskills/skills/microform
```

### Install `microwireflow` skill
```bash
# Via --skill flag
npx skills add microx-technologies/microskills --skill microwireflow

# Or pointing directly to the skill subdirectory
npx skills add microx-technologies/microskills/skills/microwireflow
```

### Install interactively or pull all skills
```bash
npx skills add microx-technologies/microskills
```

---

## 🛠 Available Skills

| Skill | Description | Path |
| :--- | :--- | :--- |
| **`microform`** | Headless form engine, Zod defaults derivation, 12-column grid layout, typed control registry, draft persistence, and submit orchestration. | [`skills/microform`](skills/microform) |
| **`microwireflow`** | Code-first wireflow documentation for React: real TSX screens, trigger element connector anchoring, interactive canvas, and image export. | [`skills/microwireflow`](skills/microwireflow) |

---

## 📐 Architecture: Progressive Disclosure

This repository strictly implements the **Progressive Disclosure** pattern to preserve agent context windows:

```text
microskills/
├── package.json
├── README.md
└── skills/
    ├── microform/
    │   ├── SKILL.md            # Entry point: metadata, core rules, and capability router
    │   ├── rules/              # Normative specifications
    │   │   └── form-spec.md    # Form construction specification (binding on agents)
    │   └── references/         # On-demand technical references
    │       ├── 01-registry-and-controls.md
    │       ├── 02-structure-and-rendering.md
    │       ├── 03-ports-and-persistence.md
    │       ├── 04-async-and-submit-flow.md
    │       └── 05-recipes-and-cheatsheet.md
    └── microwireflow/
        ├── SKILL.md            # Entry point: metadata, core rules, and capability router
        ├── rules/              # Normative specifications
        │   └── wireflow-spec.md# Wireflow construction specification (binding on agents)
        └── references/         # On-demand technical references
            ├── 01-screen-authoring-and-triggers.md
            ├── 02-flow-definition-and-edges.md
            ├── 03-canvas-rendering-and-chrome.md
            └── 04-recipes-and-cheatsheet.md
```
