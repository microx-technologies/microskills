# 05 — Canonical recipes and cheatsheet

Single canonical E2E (other files reference it; do not duplicate this code elsewhere).

```tsx
// modules/sitio/nueva/sections/hero.tsx
export function NuevaHeroSection() {
  return (
    <Section>
      <Container>
        <Badge variant="pill">Nueva</Badge>
        <SectionMainTitle as="h1">Page headline</SectionMainTitle>
        <SectionDescription>One line of context.</SectionDescription>
        <div className="flex flex-wrap gap-2 pt-1">
          <Button>View catalog</Button>
          <Button variant="neutral">Estimate per m²</Button>
        </div>
      </Container>
    </Section>
  );
}

// modules/sitio/nueva/sections/closing.tsx
export function NuevaClosingSection() {
  return (
    <Section tint>
      <Container>
        <div className="flex flex-wrap items-center justify-end gap-2">
          <Button>Start estimate</Button>
        </div>
      </Container>
    </Section>
  );
}

// modules/sitio/nueva/page.tsx — compose only
export function NuevaPage() {
  return (
    <DesktopFrame page="Nueva" slug="nueva">
      <NuevaHeroSection />
      <NuevaClosingSection />
    </DesktopFrame>
  );
}

// modules/sitio/nueva/nueva.stories.tsx — colocated (see 04)
export const Nueva: Story = { render: () => <NuevaPage /> };
export const Hero: Story = { render: () => (<Shell><NuevaHeroSection /></Shell>) };
export const Cierre: Story = { render: () => (<Shell><NuevaClosingSection /></Shell>) };
```

## Checklist before a page is done

- [ ] `page.tsx` <30 lines, `DesktopFrame` + sections only.
- [ ] Each section opens `Section > Container`, English file, `*Section` export, `hero|list/grid|closing` kind.
- [ ] Only `@/components/ui/*` + `cn()`; static `const` data with stable `key`.
- [ ] Colocated `<page>.stories.tsx` (`title "<Group>/<Page>"`, relative imports); `Guia`/`Recorrido` point at the colocated kind.
- [ ] One `as="h1"` per page, alternating `tint`, placeholders with `aria-label` + `sr-only`.
- [ ] `check` + lint clean (own code zero warnings); 360px + `a11y` review.

## Migrating a legacy domain

1. Extract each visual block from `page.tsx` to `sections/<block>.tsx` (`Section > Container`).
2. Move loose page buttons into their section (`closing` or originator).
3. Leave the page as `DesktopFrame` + section list.
4. Create/move colocated `<page>.stories.tsx` (`title "<Group>/<Page>"`, relative imports); delete the `src/stories/<Group>*` entry.
5. Repoint `Guia.mdx` and `Recorrido` to the colocated kind.
6. Run `check` + lint + 360px + `a11y`.
