# microstory — Storybook Construction Specification

## 1. Intent

- **Outcome**: Every `modules/<domain>/<page>/` folder mirrors 1:1 in the sidebar via its colocated `*.stories.tsx`: full page with frame plus each section isolated.
- **User**: AI agents and engineers documenting catalogs, calculators, content, and closings with React + Storybook.
- **Why now**: Pages accumulate layout `div`s and inline heroes, and flat per-domain stories mix sections from unrelated folders.
- **Success**: A new page is <30 lines of `page.tsx`, its colocated story appears under `<Group>/<Page>`, `tsc` + Biome pass with zero warnings in own code.
- **Constraints**: React 19, Tailwind v4, `@/*` → `./src/*`, Biome. No colocated tests; verification is `check` + visual review.
- **Out of scope**: Business logic, backends, stateful forms, `microwireflow` canvas, app simulation.

---

## 2. Invariant: Thin Page vs BARE Section vs Untouchable Shared

- **BARE** = no `DesktopFrame`, no `Card` window, no extra padding; MUST open with `Section > Container` from `@/components/ui/section`. In stories, wrapped only in `Shell <div className="w-full">`.
- **Page MUST**: import `DesktopFrame` + `*Section` from the same folder, order them, pass `page`/`slug`/`cartCount`. Limit 30 lines. PROHIBITED: layout, `.map`, `const` data, helpers.
- **Section MUST**: encapsulate `Section > Container`, its static `const` data, local sub-blocks, and its actions (`SectionLink` / `Button`). PROHIBITED: `DesktopFrame`, cross-`modules/*` imports, `Trigger`/canvas providers.
- **Shared**: `ui/*`, `layout/*`, `lib/*` are used, never edited by a domain. Missing atoms go to `ui/*` (see `02`).

---

## 3. Goals & Non-goals

### Goals

- Single source per block: a section lives in `sections/*.tsx`, reused by page + story.
- Dual visibility: every block renders bare and composed without edits.
- Vertical rhythm: `Section py-14` + `Container max-w-7xl`, `tint` on every second section (indices 1,3,5…; hero index 0 without `tint`). PROHIBITED: two consecutive `tint`s.
- Sidebar traceability: new folder + `*.stories.tsx` creates the entry; no manual mirror in `src/stories/`.

### Non-goals

- Drag-and-drop page builder. Real router/state (navigation is story → story via `linkTo`). Form validation (see `microform`).

---

## 4. Core Concepts

| Concept | Meaning |
| :--- | :--- |
| **Domain** | Folder under `src/modules/` grouping pages of one flow. |
| **Page** | `page.tsx`, exports `*Page`, sole `DesktopFrame` owner. |
| **Section** | `sections/*.tsx`, exports `*Section`, BARE block. Kind MUST be `hero`\|`list/grid`\|`closing` (see `02`); compose two sections instead of inventing a fourth. |
| **Story** | Colocated `*.stories.tsx`, `title "<Group>/<Page>"`. One `Page` export + one export per section in `Shell`. |
| **Frame/Shared** | `DesktopFrame` + `frame/*`; `ui/*` atoms; `cn()` from `lib/utils`. |

---

## 5. Normative Construction Rules (Binding on Agents)

### Rule 1: Thin page with single frame

`page.tsx` contains only `DesktopFrame` + `<*Section />` list. PROHIBITED: `div flex/grid/gap`, `Button`, `Badge`, data, heroes. Reorder by moving lines (see `02` example).

### Rule 2: BARE section with `Section > Container`

Every section opens with `Section` + `Container`. Kebab-case English files (`hero.tsx`), `PascalCase *Section` exports (see `01` naming).

### Rule 3: Exclusive `microdraw-ui` + `cn()`

Only `@/components/ui/*` + `cn()`. Forms use inline `label` + `div` with `NativeSelect`/`Input`. PROHIBITED: raw `<button>/<input>/<select>/<textarea>`, MUI/Chakra/Ant/standard shadcn/Radix Themes, custom chrome CSS. Tokens: `rounded-sketchy*`, `shadow-shadow`, Patrick Hand (see `03`).

### Rule 4: Static data and local sub-blocks

Typed `const` lists with stable `key`. Single-section sub-blocks stay local. Promote to `ui/*` only with `rounded-sketchy*` tokens; PROHIBITED: duplicating the same JSX in two sections (see `02`).

### Rule 5: Mandatory colocated story per page

`title "<Group>/<Page>"` mirroring the folder, relative imports (`./page`, `./sections/*`), no cross-`modules/*` imports. `src/stories/` holds only `Bienvenida`, `Guia`, `Recorrido` (see `04`).

### Rule 6: Docs-only honesty

No fetch/router/app state. Variants are static props or separate sections. Placeholders: `Panel tone="plain"` + `aria-label`; draft carousels use `tabIndex={-1}`, `aria-hidden` dots, `<p className="sr-only">`.

### Rule 7: Shell ordering and navigation

No numeric prefixes. Global order via `storySort.order` by group (see `04` config). Navigation only via `linkTo(kind, story)` / `<LinkTo>` with colocated kinds; anchored canvas connectors exist only in `microwireflow`.
