---
name: microstory
description: >-
  Storybook organization with modules, pages, and sections for React. Defines folder rules
  (modules/<domain>/<page>/page.tsx + sections/), componentization (thin page with DesktopFrame,
  self-contained BARE sections, untouchable shared) and exclusive microdraw-ui usage with Tailwind v4 tokens.
  Use when creating or refactoring pages, colocated stories, and isolated section stories.
  Do not use for business logic, stateful forms, wireflow canvas, or architecture diagrams.
---

# Microstory Skill

Documentation and prototyping engine for Storybook with **modules → pages → sections → shared**.
Each page is a thin composer and each business block is an isolated BARE section,
both rendered exclusively with **`microdraw-ui`**.

## Architecture

- **Modules (`src/modules/<domain>/<page>/`)**: Page owners. `page.tsx` composes sections, `sections/*.tsx` holds business, `*.stories.tsx` colocated mirrors the folder in the sidebar.
- **Shared (`src/components/ui/*`, `src/components/layout/*`, `src/lib/*`)**: Vendored microdraw-ui primitives + own atoms (`panel`, `info-tile`, `note`, `section`) + `DesktopFrame`. No domain modifies it.
- **Colocated stories**: Each `modules/<domain>/<page>/` ships `*.stories.tsx` with `title: "<Group>/<Page>"`. `src/stories/` holds only `Bienvenida`, `Guia`, `Recorrido`.
- **Storybook shell (`.storybook/`)**: `storySort` ordering, `links + docs + a11y` addons, no numeric title prefixes.

---

## Storybook Construction Specification

> [!IMPORTANT]
> All pages, sections, and stories MUST follow the normative specification:
> 👉 **[rules/storybook-spec.md](rules/storybook-spec.md)**

### Invariant Rules for Agents

1. **Thin page** (spec R1): `page.tsx` composes `<*Section />` inside `<DesktopFrame>` only.
2. **BARE section** (spec R2): each `sections/*.tsx` opens with `Section > Container` (see `02`).
3. **Exclusive UI** (spec R3): only `@/components/ui/*` + `cn()`.
4. **Colocated story** (spec R5): `title "<Group>/<Page>"`, one `Page` export + one export per section in `Shell` (see `04`).
5. **Static docs-only** (spec R6): static `const` data, no fetch/router/app state.

---

## On-Demand Reference Modules

Load only the module needed for the current task:

| Capability | Reference File |
| :--- | :--- |
| **Folders and naming** | [references/01-folders-and-naming.md](references/01-folders-and-naming.md) |
| **Page vs Section vs Shared** | [references/02-page-vs-section-vs-shared.md](references/02-page-vs-section-vs-shared.md) |
| **Microdraw-ui and styling** | [references/03-microdraw-and-styling.md](references/03-microdraw-and-styling.md) |
| **Stories and Storybook shell** | [references/04-stories-and-storybook.md](references/04-stories-and-storybook.md) |
| **Canonical recipes and checklist** | [references/05-recipes-and-cheatsheet.md](references/05-recipes-and-cheatsheet.md) |
