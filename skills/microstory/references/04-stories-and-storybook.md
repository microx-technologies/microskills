# 04 — Stories and Storybook shell

## Colocated story per page (1:1 folder mirror)

```tsx
// modules/sitio/home/home.stories.tsx
import type { Meta, StoryObj } from "@storybook/react-vite";
import { HomePage } from "./page";
import { HomeHeroSection } from "./sections/hero";

const meta = { title: "Sitio/Home" } satisfies Meta;

export default meta;
type Story = StoryObj;

function Shell({ children }: { children: React.ReactNode }) {
  return <div className="w-full">{children}</div>;
}

export const Portada: Story = { render: () => <HomePage /> };
export const Hero: Story = {
  render: () => (
    <Shell>
      <HomeHeroSection />
    </Shell>
  ),
};
```

Sidebar: `Sitio → Home (Portada + Hero + …), Quienes-Somos (…), Sucursales (…)`. Export names stay client-friendly; file/component names stay English.

## Global shell (`.storybook/`)

```ts
// main.ts — glob already covers modules/**
{ stories: ["../src/**/*.mdx", "../src/**/*.stories.@(ts|tsx)"],
  addons: ["@storybook/addon-links", "@storybook/addon-docs", "@storybook/addon-a11y"] }
```

```tsx
// preview.tsx
import "../src/index.css";
{ controls: { expanded: true }, docs: { toc: true }, a11y: { test: "todo" }, // raise to "error" to gate
  options: { storySort: { order: ["Bienvenida","Guia","Sitio","Calculadoras","Catalogo","Contenido","Cierre","Recorrido"] } } }
```

Trigger-free: no `ReactFlowProvider`, no `Trigger`, no `microwireflow` dependency.

## Story → story navigation

Anchored canvas connectors exist only in `microwireflow`. Here navigate with colocated kinds:

```tsx
<Button size="sm" onClick={linkTo("Sitio/Home", "Portada")}>View home</Button>
```

`Guia.mdx` is the sitemap (`<LinkTo kind="Sitio/Home" story="Portada" />`); `Bienvenida.mdx` the landing; `Recorrido` the guided click-through. Audit at 360px viewport + `a11y`.
