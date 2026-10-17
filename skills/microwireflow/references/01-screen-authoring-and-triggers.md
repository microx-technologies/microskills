# Screen Authoring & Trigger Anchors

Screens in `microwireflow` are standard, unadulterated React components. Interaction points are marked using the `<Trigger>` wrapper.

---

## 1. Authoring a Screen Component

Screens live as individual `.tsx` files. They should look and feel like production UI components, using your design system:

```tsx
// screens/LoginScreen.tsx
import * as React from "react";
import { Trigger } from "@microx-technologies/microwireflow";

export function LoginScreen() {
  return (
    <div className="w-[320px] p-6 bg-white border border-neutral-200 rounded-lg shadow-sm space-y-4">
      <div className="space-y-1">
        <h2 className="text-xl font-bold tracking-tight">Sign In</h2>
        <p className="text-sm text-neutral-500">Access your workspace</p>
      </div>

      <div className="space-y-3">
        <div>
          <label className="text-xs font-medium text-neutral-700">Email</label>
          <input
            type="email"
            placeholder="alice@company.com"
            className="w-full px-3 py-1.5 text-sm border rounded mt-1"
            readOnly
          />
        </div>
        <div>
          <label className="text-xs font-medium text-neutral-700">Password</label>
          <input
            type="password"
            value="••••••••"
            className="w-full px-3 py-1.5 text-sm border rounded mt-1"
            readOnly
          />
        </div>
      </div>

      {/* Primary Transition Trigger */}
      <Trigger id="submit-login">
        <button
          type="button"
          className="w-full bg-neutral-900 text-white font-medium py-2 rounded text-sm hover:bg-neutral-800"
        >
          Sign In
        </button>
      </Trigger>

      {/* Secondary Transition Trigger */}
      <div className="text-center pt-2">
        <Trigger id="forgot-password">
          <button type="button" className="text-xs text-neutral-500 hover:underline">
            Forgot your password?
          </button>
        </Trigger>
      </div>
    </div>
  );
}
```

---

## 2. The `<Trigger>` Component

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
1. **Uniqueness**: Every `id` inside a screen MUST be distinct. If `LoginScreen` has two triggers named `"submit"`, `defineFlow` will log a validation error.
2. **Interactive Elements Only**: Wrap the actual action trigger (button, link, tab, card click area), not the entire screen or paragraph.
3. **No Event Interception**: `<Trigger>` only coordinates canvas diagram navigation. It does not stop click events or prevent native button focus.

---

## 3. Single vs. Branching Triggers

### Single Destination:
If an edge connects `from: "login", trigger: "submit-login", to: "dashboard"`, clicking the trigger button in the live canvas smoothly scrolls and pans the viewport to center `dashboard`.

### Multiple Destinations (Branching):
If multiple edges share the same trigger ID:
```tsx
edges: [
  { from: "login", trigger: "submit-login", to: "dashboard", label: "success" },
  { from: "login", trigger: "submit-login", to: "mfa-verify", label: "2FA enabled" },
  { from: "login", trigger: "submit-login", to: "login-error", label: "invalid credentials" },
]
```
Clicking the trigger opens a floating **Destination Picker** menu on the canvas, allowing the reviewer to select which path to inspect.
