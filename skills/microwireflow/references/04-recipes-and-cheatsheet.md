# Recipes & Canonical Cheatsheet

Complete, production-ready wireflow architecture binding real screen components built **exclusively with `microdraw-ui`**, transition triggers, and the interactive canvas.

> [!IMPORTANT]
> **Strict Rule**:
> Screens MUST be authored exclusively with components from **`microdraw-ui`** (`@/components/ui/button`, `@/components/ui/card`, `@/components/ui/input`, `@/components/ui/dialog`, etc.).
> Never use raw unstyled HTML interactive controls or other component libraries (MUI, Chakra, standard shadcn, Radix Themes).

---

## 1. Screen Components (`screens/`)

### `screens/Login.tsx`
```tsx
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

### `screens/Home.tsx`
```tsx
import * as React from "react";
import { Trigger } from "microwireflow";
import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";

export function Home() {
  return (
    <Card size="sm" className="gap-3">
      <CardHeader>
        <div className="flex items-center justify-between">
          <div>
            <CardTitle>Welcome back, Ada</CardTitle>
            <CardDescription>
              3 flows active · last session saved 12m ago.
            </CardDescription>
          </div>
          <span className="rounded-sketchy-pill border-2 border-border bg-secondary-background px-2.5 py-0.5 text-xs">
            Lead Architect
          </span>
        </div>
      </CardHeader>

      <CardContent className="flex flex-col gap-3">
        <div className="grid grid-cols-3 gap-2">
          <div className="rounded-sketchy-card border-2 border-border bg-secondary-background p-2 text-center shadow-shadow">
            <div className="text-lg font-bold leading-none">14</div>
            <div className="text-xs text-muted-foreground">Wireflows</div>
          </div>
          <div className="rounded-sketchy-card border-2 border-border bg-secondary-background p-2 text-center shadow-shadow">
            <div className="text-lg font-bold leading-none">42</div>
            <div className="text-xs text-muted-foreground">Live screens</div>
          </div>
          <div className="rounded-sketchy-card border-2 border-border bg-secondary-background p-2 text-center shadow-shadow">
            <div className="text-lg font-bold leading-none">8</div>
            <div className="text-xs text-muted-foreground">Reviewers</div>
          </div>
        </div>
      </CardContent>

      <CardFooter className="flex items-center gap-2 pt-1">
        <Trigger id="logout" fullWidth>
          <Button type="button" variant="neutral" className="flex-1">
            Log out of workspace
          </Button>
        </Trigger>
      </CardFooter>
    </Card>
  );
}
```

---

## 2. Flow Definition (`flows/auth.tsx`)

```tsx
// flows/auth.tsx
import * as React from "react";
import { defineFlow } from "microwireflow/flow";
import { Home } from "@/screens/Home";
import { Login } from "@/screens/Login";
import { LogoutConfirm } from "@/screens/LogoutConfirm";
import { Reset } from "@/screens/Reset";

export const authFlow = defineFlow({
  nodes: [
    {
      id: "login",
      title: "Login",
      kind: "<<screen>>",
      component: <Login />,
      notes: "Entry point. Submit branches on validity; both leaves stay visible.",
    },
    {
      id: "login-error",
      title: "Login",
      kind: "<<error>>",
      component: <Login error="Bad credentials — try again." />,
      notes: "Error state, same file. Red-inked to read as a correction.",
    },
    {
      id: "home",
      title: "Home",
      kind: "<<screen>>",
      component: <Home />,
      notes: "Signed-in landing. Logout asks first via a modal.",
    },
    {
      id: "logout-confirm",
      title: "Confirm logout",
      kind: "<<modal>>",
      component: <LogoutConfirm />,
      notes: "Modal kind proof: double-ruled badge. Confirm exits, cancel returns.",
    },
    {
      id: "reset",
      title: "Reset",
      kind: "<<screen>>",
      component: <Reset />,
      notes: "Password reset. Added: forgot edge needs a target.",
    },
  ],
  edges: [
    {
      from: "login",
      trigger: "submit",
      to: "home",
      label: "if valid",
      lineStyle: "solid",
    },
    {
      from: "login",
      trigger: "submit",
      to: "login-error",
      label: "if invalid",
      lineStyle: "dashed",
    },
    { from: "login", trigger: "forgot", to: "reset", lineStyle: "dotted" },
    { from: "home", trigger: "logout", to: "logout-confirm", label: "tap" },
    {
      from: "logout-confirm",
      trigger: "confirm",
      to: "login",
      label: "yes, out",
    },
    {
      from: "logout-confirm",
      trigger: "cancel",
      to: "home",
      label: "stay",
      lineStyle: "dashed",
    },
    { from: "reset", trigger: "send", to: "login" },
  ],
  positions: {
    "login": { x: 50, y: 100 },
    "login-error": { x: 50, y: 550 },
    "home": { x: 600, y: 100 },
    "logout-confirm": { x: 600, y: 550 },
    "reset": { x: -450, y: 320 },
  },
});
```

---

## 3. Wireflow Canvas Hosting (`AuthDocs.tsx`)

```tsx
// AuthDocs.tsx
import * as React from "react";
import { Wireflow } from "microwireflow";
import "microwireflow/styles.css";
import { authFlow } from "./flows/auth";

export function AuthDocs() {
  return (
    <div style={{ width: "100vw", height: "100vh" }}>
      <Wireflow
        flow={authFlow}
        showMinimap
        showSearch
        showControls
        nodeWidth={460}
        onPositionsExport={(positions, snippet) => {
          console.log("Exported positions:", snippet);
        }}
      />
    </div>
  );
}
```
