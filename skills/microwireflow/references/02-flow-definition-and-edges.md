# Flow Definition & Edges

Flows in `microwireflow` bind screen nodes and transition connectors together with full compile-time and runtime integrity checks.

---

## 1. Defining a Flow (`defineFlow`)

Import `defineFlow` from the dedicated lightweight subpath `microwireflow/flow`:

```tsx
// flows/authFlow.tsx
import * as React from "react";
import { defineFlow } from "microwireflow/flow";
import { LoginScreen } from "../screens/LoginScreen";
import { DashboardScreen } from "../screens/DashboardScreen";
import { ResetPasswordScreen } from "../screens/ResetPasswordScreen";

export const authFlow = defineFlow({
  nodes: [
    {
      id: "login",
      title: "Sign In",
      kind: "<<screen>>",
      component: <LoginScreen />,
      notes: "Default entry point for unauthenticated visitors.",
    },
    {
      id: "dashboard",
      title: "Main Dashboard",
      kind: "<<screen>>",
      component: <DashboardScreen />,
      notes: "Requires active JWT session with role=user or role=admin.",
    },
    {
      id: "reset-password",
      title: "Password Recovery",
      kind: "<<modal>>",
      component: <ResetPasswordScreen />,
      notes: "Sends 6-digit OTP code to registered email address.",
    },
  ],
  edges: [
    {
      from: "login",
      trigger: "submit-login",
      to: "dashboard",
      label: "valid credentials",
      lineStyle: "solid",
      arrow: "closed",
    },
    {
      from: "login",
      trigger: "forgot-password",
      to: "reset-password",
      label: "recovery request",
      lineStyle: "dashed",
      arrow: "open",
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
- **Unmatched Triggers**: If `from: "login"` has no `<Trigger id="submit-login">` in its JSX, `defineFlow` alerts in console and overlay.
- **Duplicate IDs**: Duplicate node IDs or duplicate trigger IDs within a single screen are flagged immediately.
