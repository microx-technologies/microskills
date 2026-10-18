# 03 — Microdraw-ui and styling

Norm: see spec Rule 3. All screens MUST use `@/components/ui/*` exclusively.

## Setup (`microdraw-ui` reference)

```bash
bun x shadcn registry add @microdraw=https://microx-technologies.github.io/microdraw-ui/r/{name}.json
bun x shadcn add @microdraw/styling/neutral
bun x shadcn add @microdraw/button @microdraw/badge @microdraw/card @microdraw/native-select
```

```css
@import "tailwindcss";
@import "tw-animate-css";
@import "@fontsource/patrick-hand/latin-400.css";
```

## Primitives in use

| Need | Component |
| :--- | :--- |
| Title + rhythm | `Section, Container, SectionHeader, SectionTitle, SectionMainTitle, SectionDescription, SectionLink` |
| Sketchy box | `Panel tone="highlight"\|"plain" size="md"\|"sm"` |
| Compact tile | `InfoTile title titleClassName` |
| Notice | `Note` |
| CTA | `Button` (`default`/`neutral`, `sm`/`icon`) |
| Label | `Badge variant="pill"\|"neutral"` |
| Select | `NativeSelect` + inline `label` |
| Draft photo | `Panel tone="plain" aria-label="Draft photo: ..."` |

## Style patterns

- Rhythm: `Section` gives `flex flex-col py-14` + divider; `Container` sets `max-w-7xl px-6`. Pages add no gaps.
- Bands: `tint` on every second section (indices 1,3,5…).
- Grids: `grid-cols-2 sm:lg:6` (brands), `sm:3` (calculators), `lg:4` (branches/products).
- One `SectionMainTitle as="h1"` per page (hero); draft carousels use `tabIndex={-1}` + `aria-hidden` + `<p className="sr-only">`.

PROHIBITED: raw `<button>/<input>/<select>/<textarea>`, other kits, custom chrome CSS, `Button` outside a section.
