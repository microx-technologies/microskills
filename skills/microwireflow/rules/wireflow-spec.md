# microwireflow — Wireflow Construction Specification

## 1. Intent

- **Outcome**: Deterministic, code-first wireflow documentation rendering real React TSX screen components on an interactive canvas, with transition connectors anchored directly to userland interactive elements.
- **User**: AI agents and design engineers creating UX navigation documentation, design handoffs, and flow maps.
- **Why now**: Traditional diagrams (Figma, Mermaid, generic React Flow) either detach screens from actual frontend code or anchor edges blindly to the outer bounding box of nodes rather than specific interactive buttons/links.
- **Success**: A complete 5-screen wireflow defined entirely in code with zero custom canvas CSS, where reviewers can trace every button directly to its destination screen.
- **Constraints**: React 18 or 19, `@xyflow/react` ^12. Screen interiors are author-owned; canvas chrome, layout, and connector anchoring are engine-owned.
- **Out of scope**: State-machine execution, router simulation, live form validation inside screens, drag-and-drop wireframe builder UIs.

---

## 2. Invariant: Docs-Only Canvas vs. App Simulation

The canvas is strictly a **documentation and communication tool**:

- **Triggers only navigate**: Clicking a `<Trigger>` on the canvas focuses and centers the destination screen node or displays a floating picker if multiple edges originate from that trigger.
- **Triggers MUST NOT simulate app logic**: Triggers do not submit forms, run API mutations, or transition app states.
- **Screens are self-contained snapshots**: Pass static props or mock data to represent state variants (e.g. `LoginForm state="error"` or separate `LoginScreen` and `LoginErrorScreen` components).

---

## 3. Goals & Non-goals

### Goals
- Code is single source of truth: Screens are real `.tsx` files importing shared design system components.
- Exact element anchoring: Connectors emerge from the precise button or link causing the action via `<Trigger id="...">`.
- Loud dev-time validation: Broken references (edges pointing to non-existent nodes or triggers) throw visible errors, never fail silently.
- Pluggable layouts: Automated ELK layered hierarchical layout with optional manual coordinate overrides (`positions`).

### Non-goals
- Application runtime simulation / functional prototyping.
- Schema-to-UI generators (screens are hand-coded TSX, not JSON trees).
- Bidirectional drag sync to remote databases (positions are exported to TS files).

---

## 4. Core Concepts

| Concept | Meaning |
| :--- | :--- |
| **Screen Component** | Pure TSX component representing a page, modal, or step. Wrapped with zero extra markup. |
| **Trigger** | Inline wrapper component (`<Trigger id="...">`) marking a clickable element as a connector source handle. |
| **FlowNode** | Object declaring a screen node: `id`, `title`, optional `kind` (`<<screen>>`, `<<modal>>`), `component`, and `notes`. |
| **FlowEdge** | Object declaring a transition: `from` (node ID), `trigger` (trigger ID), `to` (target node ID), optional `label`, `lineStyle`, `arrow`. |
| **defineFlow** | Type-safe factory and validator checking referential integrity of nodes, edges, and triggers. |
| **Wireflow** | Full canvas component (`<Wireflow flow={flow} />`) rendering nodes, minimap, search, controls, and annotations panel. |
| **Positions** | Coordinate map (`Record<string, { x: number, y: number }>`) overriding the automated ELK layout engine. |

---

## 5. Normative Construction Rules (Binding on Agents)

Every wireflow built using `microwireflow` MUST comply with the following rules:

### Rule 1: Single-Source TSX Screens
Screens MUST be declared as real React functional components. They should import real design system primitives (buttons, inputs, cards) rather than placeholder HTML strings.

### Rule 2: Explicit Trigger Element Anchoring
Interactive elements that cause transitions MUST be wrapped with `<Trigger id="...">`.
- The `id` MUST be a unique slug within that screen component (e.g. `submit-btn`, `forgot-password-link`).
- Never nest a `<Trigger>` inside another `<Trigger>`.
- `<Trigger>` preserves child layout and styling without injecting disruptive wrappers.

### Rule 3: Declarative Wiring via `defineFlow`
Flows MUST be created using `defineFlow({ nodes, edges, positions? })`:
```tsx
import { defineFlow } from "@microx-technologies/microwireflow/flow";

export const authFlow = defineFlow({
  nodes: [ ... ],
  edges: [ ... ],
});
```
Every `from` in `edges` must match a `node.id`. Every `trigger` must match an existing `<Trigger id>` inside that source node. Every `to` must match a target `node.id`.

### Rule 4: Descriptive Screen Metadata and Notes
Every `FlowNode` SHOULD include meaningful `title` and `notes`:
- `title`: Short human-readable title displayed in the node header and search panel.
- `kind`: Structural tag (e.g. `"<<screen>>"`, `"<<modal>>"`, `"<<drawer>>"`, `"<<error>>"`).
- `notes`: Markdown or text explaining permissions, API requirements, or edge-case behavior displayed in the annotations panel.

### Rule 5: Zero Author CSS for Canvas Chrome
Authors MUST NOT write CSS to style the canvas, node cards, headers, connector lines, or panels. All chrome is built-in and themed consistently using the microdraw neobrutalist aesthetic.

### Rule 6: Hand Positions Persistence
When manual layout tuning is performed on the canvas, authors export positions via the canvas export tool and paste the resulting `positions` map into `defineFlow({ ..., positions })`. The engine guarantees that explicit positions override automated ELK calculations.

### Rule 7: Stylesheet Inclusion
Any root application hosting `<Wireflow />` MUST import the package stylesheet:
```tsx
import "@microx-technologies/microwireflow/styles.css";
```
