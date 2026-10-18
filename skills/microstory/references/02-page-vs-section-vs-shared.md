# 02 — Page vs Section vs Shared

## Page: thin composer (sole frame owner)

```tsx
// modules/sitio/home/page.tsx
import { DesktopFrame } from "@/components/layout/DesktopFrame";
import { HomeBrandsSection } from "./sections/brands";
import { HomeCalculatorsSection } from "./sections/calculators";
import { HomeHeroSection } from "./sections/hero";

export function HomePage() {
  return (
    <DesktopFrame page="Portada / Inicio" slug="" cartCount={3}>
      <HomeHeroSection />
      <HomeBrandsSection />
      <HomeCalculatorsSection />
    </DesktopFrame>
  );
}
```

`DesktopFrame` only wraps: `ScreenShell > BrowserBar + UtilityBar + HeaderVisual + main(children + FabVisual) + FooterVisual`. PROHIBITED in page: `div flex/grid`, `Button`, `Badge`, heroes, data, `.map`.

- Anti-pattern (1 line): heroes/buttons inside `page.tsx` → move to `sections/` (`closing` or originating section).

## Section: BARE business block

Signature (full example in `05`):

```tsx
export function StoresFilterSection() {
  return (
    <Section>
      <Container>{/* SectionHeader + business + actions */}</Container>
    </Section>
  );
}
```

- Closed taxonomy: every section MUST be `hero` (`as="h1"` + `Badge` + CTAs), `list/grid` (`.map` over `const` + `Panel`/`InfoTile`), or `closing` (single right-aligned action). Compose two sections instead of a fourth pattern.
- Local sub-blocks stay local (one-section use). To add a shared atom: create `src/components/ui/<atom>.tsx` with `rounded-sketchy*` tokens; PROHIBITED: duplicating the same JSX in two sections.

## Shared: used, never edited by domains

- `ui/section.tsx`: `Section` (`py-14` + divider), `tint` band, `Container` (`max-w-7xl px-6`), `SectionHeader/Title/MainTitle/Description/Link`.
- `ui/panel.tsx`: `Panel tone="highlight"|"plain" size="md"|"sm"` + `EntryCard`; `ui/info-tile.tsx`: `InfoTile`; `ui/note.tsx`: `Note`.
- `layout/DesktopFrame.tsx` + `frame/*`, `lib/utils.ts` (`cn()`).
