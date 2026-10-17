# Screen Authoring & Trigger Anchors

Screens in `microwireflow` are plain React TSX components authored **exclusively using `microdraw-ui`**. Interaction points are marked using the `<Trigger>` wrapper.

---

## 1. Mandatory Component Library: `microdraw-ui`

> [!IMPORTANT]
> **Strict Requirement**:
> All screen components MUST use primitives from **`microdraw-ui`** (`@/components/ui/*`).
> - **Prohibited**: Never use raw HTML controls (`<button>`, `<input>`, `<dialog>`, etc.) or any other third-party component libraries (standard shadcn/ui, Radix Themes, Material UI, Chakra UI, Ant Design, Mantine).
> - **Architecture & Aesthetics**: Screens match the hand-drawn, sketchy neobrutalist aesthetic of the canvas, powered by `@fontsource/patrick-hand` and Tailwind CSS v4 sketchy tokens (`rounded-sketchy`, `rounded-sketchy-card`, `rounded-sketchy-pill`, `shadow-shadow`).

### Installing & Setting Up `microdraw-ui`

Follow the configuration in `microwireflow/apps/demo`:

```bash
# 1. Register the @microdraw registry in your components.json
bun x shadcn registry add @microdraw=https://microx-technologies.github.io/microdraw-ui/r/{name}.json

# 2. Add base theme styling
bun x shadcn add @microdraw/styling/neutral

# 3. Add needed microdraw-ui components
bun x shadcn add @microdraw/button @microdraw/card @microdraw/input @microdraw/dialog
```

Ensure your stylesheet (`src/index.css`) imports the font and animation primitives:
```css
@import "tailwindcss";
@import "tw-animate-css";
@import "@fontsource/patrick-hand/latin-400.css";
```

---

## 2. Canonical Screen Component (`screens/Login.tsx`)

Screens live as individual `.tsx` files importing components from `@/components/ui/*`:

```tsx
// screens/Login.tsx
import * as React from "react";
import { Trigger } from "microwireflow";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

export function Login({ error }: { error?: string }) {
  return (
    <div className="flex flex-col gap-3">
      <div className="flex items-start justify-between gap-2">
        <div>
          <h2 className="font-heading text-xl leading-none">Welcome back</h2>
          <p className="mt-1 text-xs text-muted-foreground">
            Sign in to sketch and inspect wireflow architectures.
          </p>
        </div>
        <span className="rounded-sketchy-pill border-2 border-border bg-secondary-background px-2.5 py-0.5 text-xs">
          Studio v0.2
        </span>
      </div>

      {error ? (
        <p className="rounded-sketchy-card border-2 border-border bg-secondary-background px-3 py-2 text-sm text-foreground shadow-shadow">
          {error}
        </p>
      ) : null}

      <div className="grid grid-cols-2 gap-3">
        <label className="flex flex-col gap-1 text-sm" htmlFor="login-email">
          Email address
          <Input id="login-email" placeholder="ada@sketch.shop" />
        </label>
        <label className="flex flex-col gap-1 text-sm" htmlFor="login-pass">
          Access token / password
          <Input id="login-pass" type="password" placeholder="••••••••••••" />
        </label>
      </div>

      <div className="flex items-center justify-between text-xs text-muted-foreground">
        <span className="flex items-center gap-1.5">
          <input
            type="checkbox"
            id="remember-me"
            defaultChecked
            className="accent-foreground"
          />
          <label htmlFor="remember-me" className="cursor-pointer">
            Remember session on this canvas
          </label>
        </span>
        <Trigger id="forgot">
          <Button type="button" variant="neutral" size="sm">
            Forgot password?
          </Button>
        </Trigger>
      </div>

      <div className="pt-1">
        <Trigger id="submit" fullWidth>
          <Button type="button" className="flex-1">
            Sign in to board
          </Button>
        </Trigger>
      </div>
    </div>
  );
}
```

---

## 3. The `<Trigger>` Component

The `<Trigger>` component attaches a React Flow invisible source handle to the wrapped DOM node.

### Props:
| Prop | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `id` | `string` | **required** | Unique identifier within this screen. Used by `FlowEdge.trigger`. |
| `children` | `React.ReactNode` | **required** | Single element or markup to wrap. |
| `fullWidth` | `boolean` | `false` | Sets `width: 100%` on the wrapper if needed for block buttons. |
| `className` | `string` | `undefined` | Optional utility classes applied to the trigger container. |
| `style` | `React.CSSProperties` | `undefined` | Optional inline styles. |

### Rules for Triggers:
1. **Uniqueness**: Every `id` inside a screen MUST be distinct. If `Login` has two triggers named `"submit"`, `defineFlow` will log a validation error.
2. **Interactive Elements Only**: Wrap the actual action trigger (button, link, tab, card click area), not the entire screen or paragraph.
3. **No Event Interception**: `<Trigger>` only coordinates canvas diagram navigation. It does not stop click events or prevent native button focus.

---

## 4. Single vs. Branching Triggers

### Single Destination:
If an edge connects `from: "login", trigger: "submit", to: "home"`, clicking the trigger button in the live canvas smoothly scrolls and pans the viewport to center `home`.

### Multiple Destinations (Branching):
If multiple edges share the same trigger ID:
```tsx
edges: [
  { from: "login", trigger: "submit", to: "home", label: "if valid" },
  { from: "login", trigger: "submit", to: "login-error", label: "if invalid" },
  { from: "login", trigger: "forgot", to: "reset", lineStyle: "dotted" },
]
```
Clicking the trigger opens a floating **Destination Picker** menu on the canvas, allowing the reviewer to select which path to inspect.
