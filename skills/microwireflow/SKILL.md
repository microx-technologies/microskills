---
name: microwireflow
description: >-
  Code-first wireflow documentation engine for React. Renders real TSX screen components
  onto an interactive React Flow canvas with connectors anchored directly to trigger elements.
  Use when building or documenting UI flows, wireflows, screen navigation maps, or interactive flow documentation.
  Do not use for general architecture diagrams without screens, backend flowcharts, or full app functional simulation.
---

# Microwireflow Skill

Code-first wireflow documentation engine for React applications. Renders real TSX screen components onto an interactive React Flow canvas with transition edges anchored directly to specific `<Trigger>` elements.

## Architecture

- **Canvas & Primitives (`@microx-technologies/microwireflow`)**: Full canvas renderer (`Wireflow`), trigger anchor wrapper (`Trigger`), image export utilities (`exportAsPng`, `exportAsSvg`), and stylesheet (`@microx-technologies/microwireflow/styles.css`).
- **Flow Engine (`@microx-technologies/microwireflow/flow`)**: Lightweight flow definition and validation helper (`defineFlow`), node/edge type contracts (`FlowNode`, `FlowEdge`, `Flow`, `LineStyle`, `FlowPositions`).

---

## Wireflow Construction Specification

> [!IMPORTANT]
> All wireflows and screens MUST adhere to the normative specification:
> 👉 **[rules/wireflow-spec.md](rules/wireflow-spec.md)**

### Invariant Rules for Agents
1. **Real TSX Screens**: Screens MUST be plain, single-source React TSX components. Never author screen contents as mock JSON or abstract schemas.
2. **Trigger Anchoring**: Transitions MUST anchor to interactive elements wrapped with `<Trigger id="...">`. Trigger IDs MUST be unique within their screen.
3. **Loud Validation via `defineFlow`**: Declare flows using `defineFlow({ nodes, edges, positions? })`. Never bypass validation; invalid edge triggers or dangling node IDs surface dev-time errors.
4. **Docs-Only Honesty**: The canvas is strictly documentation. Trigger clicks navigate/focus target screens or open destination pickers; triggers NEVER execute application business logic or simulate state machines.
5. **Stylesheet Requirement**: Any application hosting `<Wireflow />` MUST import `@microx-technologies/microwireflow/styles.css`.

---

## On-Demand Reference Modules

Load the specific reference module required for the current task:

| Capability | Reference File |
| :--- | :--- |
| **Screen Authoring & Trigger Anchors** | [references/01-screen-authoring-and-triggers.md](references/01-screen-authoring-and-triggers.md) |
| **Flow Definitions, Edges & Routing** | [references/02-flow-definition-and-edges.md](references/02-flow-definition-and-edges.md) |
| **Canvas Hosting, Controls & Chrome** | [references/03-canvas-rendering-and-chrome.md](references/03-canvas-rendering-and-chrome.md) |
| **Canonical End-to-End Cheatsheet** | [references/04-recipes-and-cheatsheet.md](references/04-recipes-and-cheatsheet.md) |
