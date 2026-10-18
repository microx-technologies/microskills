# 01 — Folders and naming

Canonical layout (glob `../src/**/*.stories.tsx` in `.storybook/main.ts` already covers `modules/**`; no config change):

```text
apps/storybook/src/
├── modules/<domain>/<page>/page.tsx
├── modules/<domain>/<page>/<page>.stories.tsx   # colocated, title "<Group>/<Page>"
├── modules/<domain>/<page>/sections/<block>.tsx
├── components/ui/*          # vendored microdraw-ui + own atoms
├── components/layout/DesktopFrame.tsx
├── components/layout/frame/*
├── lib/utils.ts             # cn()
├── stories/                 # Bienvenida.mdx, Guia.mdx, Recorrido.stories.tsx only
└── index.css                # Tailwind v4 tokens + Patrick Hand
```

## Rules

1. **Domain and page in kebab-case**: `sitio/home`, `sitio/quienes-somos`. No uppercase, no level labels in paths.
2. **Section files in kebab-case English**: `hero.tsx`, `featured-products.tsx`, `list.tsx`. Never `HeroSection.tsx`.
3. **Story file named after the page**: `home.stories.tsx`. `title: "<Group>/<Page>"` mirrors the folder (`Sitio/Home`). Relative imports (`./page`, `./sections/hero`).
4. **`PascalCase` exports with suffix**: `*Page`, `*Section`. One component per file; a story file may hold N exports.
5. **Alias `@/*` → `./src/*`** for `components/*` and `lib/*`. Relative imports inside the module.
6. **Zero cross-domain imports**: no `sitio/*` importing from `catalogo/*`. Shared code lives in `components/` or `lib/`.
7. **No global `components/sections/`**: page heroes, fields, and cards inline or live in their section.

## Anti-patterns

- `sections/ui/` subfolders. If a section needs >2 files, use `sections/<block>/index.tsx` + local parts; PROHIBITED: importing that subfolder from another section.
- Fat `page.tsx`: move helpers to `sections/` or to `ui/*` when shared (limit 30 lines).
- Numeric story prefixes (`01-Sitio`): ordering comes from `storySort`, not titles.
