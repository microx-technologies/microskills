# Recipes & Canonical Cheatsheet

Complete, production-ready wireflow binding screens, transition triggers, and interactive canvas.

---

## 1. Screen Components (`screens/`)

```tsx
// screens/AuthScreens.tsx
import * as React from "react";
import { Trigger } from "@microx-technologies/microwireflow";

export function LoginScreen() {
  return (
    <div style={{ width: 280, padding: 16, background: "#fff", borderRadius: 8, border: "1px solid #e5e5e5" }}>
      <h3 style={{ margin: "0 0 12px", fontSize: 16, fontWeight: "bold" }}>Sign In</h3>
      <input
        type="email"
        placeholder="user@example.com"
        style={{ width: "100%", padding: "6px 8px", marginBottom: 8, border: "1px solid #ccc", borderRadius: 4 }}
        readOnly
      />
      <input
        type="password"
        value="••••••••"
        style={{ width: "100%", padding: "6px 8px", marginBottom: 12, border: "1px solid #ccc", borderRadius: 4 }}
        readOnly
      />
      <Trigger id="submit-login">
        <button
          type="button"
          style={{ width: "100%", padding: "8px 0", background: "#171717", color: "#fff", border: "none", borderRadius: 4, cursor: "pointer" }}
        >
          Continue
        </button>
      </Trigger>
      <div style={{ marginTop: 8, textAlign: "center" }}>
        <Trigger id="forgot-password">
          <button
            type="button"
            style={{ background: "none", border: "none", color: "#737373", fontSize: 12, cursor: "pointer", textDecoration: "underline" }}
          >
            Forgot password?
          </button>
        </Trigger>
      </div>
    </div>
  );
}

export function DashboardScreen() {
  return (
    <div style={{ width: 340, padding: 16, background: "#fff", borderRadius: 8, border: "1px solid #e5e5e5" }}>
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 16 }}>
        <h3 style={{ margin: 0, fontSize: 16, fontWeight: "bold" }}>Project Dashboard</h3>
        <Trigger id="logout-btn">
          <button type="button" style={{ fontSize: 12, padding: "4px 8px" }}>Sign Out</button>
        </Trigger>
      </div>
      <div style={{ height: 100, background: "#f5f5f5", borderRadius: 4, display: "grid", placeItems: "center", color: "#737373" }}>
        Activity Overview
      </div>
    </div>
  );
}

export function ForgotPasswordScreen() {
  return (
    <div style={{ width: 280, padding: 16, background: "#fff", borderRadius: 8, border: "1px solid #e5e5e5" }}>
      <h3 style={{ margin: "0 0 12px", fontSize: 16, fontWeight: "bold" }}>Password Recovery</h3>
      <p style={{ fontSize: 12, color: "#666", marginBottom: 12 }}>
        Enter your email to receive recovery instructions.
      </p>
      <input
        type="email"
        placeholder="user@example.com"
        style={{ width: "100%", padding: "6px 8px", marginBottom: 12, border: "1px solid #ccc", borderRadius: 4 }}
        readOnly
      />
      <Trigger id="send-recovery-link">
        <button
          type="button"
          style={{ width: "100%", padding: "8px 0", background: "#171717", color: "#fff", border: "none", borderRadius: 4, cursor: "pointer" }}
        >
          Send Link
        </button>
      </Trigger>
    </div>
  );
}
```

---

## 2. Flow Definition (`flows/authFlow.tsx`)

```tsx
// flows/authFlow.tsx
import * as React from "react";
import { defineFlow } from "@microx-technologies/microwireflow/flow";
import { LoginScreen, DashboardScreen, ForgotPasswordScreen } from "../screens/AuthScreens";

export const authFlow = defineFlow({
  nodes: [
    {
      id: "login",
      title: "Authentication",
      kind: "<<screen>>",
      component: <LoginScreen />,
      notes: "Main entry screen for unauthenticated sessions.",
    },
    {
      id: "dashboard",
      title: "Workspace Dashboard",
      kind: "<<screen>>",
      component: <DashboardScreen />,
      notes: "Authenticated workspace overview.",
    },
    {
      id: "forgot-password",
      title: "Recovery Modal",
      kind: "<<modal>>",
      component: <ForgotPasswordScreen />,
      notes: "Dispatches email reset token.",
    },
  ],
  edges: [
    {
      from: "login",
      trigger: "submit-login",
      to: "dashboard",
      label: "success",
      lineStyle: "solid",
      arrow: "closed",
    },
    {
      from: "login",
      trigger: "forgot-password",
      to: "forgot-password",
      label: "forgot",
      lineStyle: "dashed",
      arrow: "open",
    },
    {
      from: "dashboard",
      trigger: "logout-btn",
      to: "login",
      label: "sign out",
      lineStyle: "dotted",
    },
  ],
  positions: {
    "login": { x: 50, y: 100 },
    "dashboard": { x: 480, y: 100 },
    "forgot-password": { x: 50, y: 500 },
  },
});
```

---

## 3. Wireflow Canvas Hosting (`App.tsx`)

```tsx
// App.tsx
import * as React from "react";
import { Wireflow } from "@microx-technologies/microwireflow";
import "@microx-technologies/microwireflow/styles.css";
import { authFlow } from "./flows/authFlow";

export function App() {
  return (
    <div style={{ width: "100vw", height: "100vh" }}>
      <Wireflow
        flow={authFlow}
        showMinimap
        showSearch
        showControls
        nodeWidth={340}
        onPositionsExport={(positions, snippet) => {
          console.log("Exported positions:", snippet);
        }}
      />
    </div>
  );
}
```
