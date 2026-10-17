# Flow Definition & Edges

Flows in `microwireflow` bind screen nodes and transition connectors together with full compile-time and runtime integrity checks.

---

## 1. Defining a Flow (`defineFlow`)

Import `defineFlow` from the dedicated lightweight subpath `microwireflow/flow`:

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
      notes: "Password reset screen. Form recovers access token.",
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
    {
      from: "login",
      trigger: "forgot",
      to: "reset",
      lineStyle: "dotted",
    },
    {
      from: "home",
      trigger: "logout",
      to: "logout-confirm",
      label: "tap",
    },
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
    {
      from: "reset",
      trigger: "send",
      to: "login",
    },
  ],
});
```

---

## 2. `FlowNode` Contract

```ts
export type FlowNode = {
  id: string;               // Unique screen ID used as edge source/target
  title: string;            // Human-readable header title
  kind?: string;            // Badge tag (e.g. "<<screen>>", "<<modal>>", "<<drawer>>")
  component: React.ReactNode; // React element representing the screen UI
  notes?: string;           // Documentation rendered in annotations panel
};
```

---

## 3. `FlowEdge` Contract & Edge Styling

Each edge represents a directed transition from a trigger inside a source node to a target node:

```ts
export type FlowEdge = {
  from: string;                 // Source FlowNode.id
  trigger: string;              // Source screen <Trigger id="...">
  to: string;                   // Target FlowNode.id
  label?: string;               // Text pill badge shown along connector line
  lineStyle?: LineStyle;        // "solid" | "dashed" | "dotted"
  arrow?: ArrowStyle;           // boolean | "closed" | "open" | "none"
  routing?: EdgeRouting;        // "smoothstep" | "step" | "bezier" | "straight"
  targetSide?: TargetSide;      // "left" | "top" | "right" | "bottom"
  sourceSide?: SourceSide;      // "left" | "right"
  avoidCollisions?: boolean;    // Smart routing around intermediate nodes
  lineHops?: boolean;           // Hop over intersecting edge paths
};
```

### Visual Styling Options:
- **`lineStyle`**:
  - `"solid"` (default): Primary happy-path user flows.
  - `"dashed"`: Secondary or optional branches (e.g. forgot password, skip step).
  - `"dotted"`: Background/implicit triggers (e.g. session timeout, webhook refresh).
- **`arrow`**:
  - `"closed"` (default): Solid arrow pointing to target node.
  - `"open"`: Minimal open bracket arrowhead.
  - `"none"`: Plain connector stroke.

---

## 4. Loud Dev-Time Validation

`defineFlow` executes validation during initialization:
- **Dangling Targets**: If `to: "checkout"` is specified but no node with `id: "checkout"` exists, an error banner is shown with the exact line and node ID.
- **Unmatched Triggers**: If `from: "login"` has no `<Trigger id="submit">` in its JSX, `defineFlow` alerts in console and overlay.
- **Duplicate IDs**: Duplicate node IDs or duplicate trigger IDs within a single screen are flagged immediately.
