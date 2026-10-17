---
name: microform
description: >-
  Headless form engine and data-entry toolkit with typed control registry, Zod defaults,
  12-column grid structures, draft persistence, async sources, and submit orchestration.
  Use when creating or refactoring forms, generating form fields from schemas,
  handling draft auto-save, building custom form controls, or managing async queries in microform.
  Do not use for raw HTML forms without React or simple single-input forms without microform.
---

# Microform Skill

Headless, decoupled form engine adhering to **Ports and Adapters** and the **Open/Closed Principle (OCP)**. Decouples business logic, schema validation, and layout grids from presentation UI.

## Architecture

- **Core (`@microx-technologies/microform`)**: Framework-agnostic. Registry (`createMicroform`), schema defaults (`createFormDefaults`), required field maps (`buildRequiredMap`), variable cleaner (`cleanVars`), memory adapter, and ports.
- **React (`@microx-technologies/microform/react`)**: React 19 bindings. Context (`MicroformProvider`), dynamic dispatcher (`InferFieldInput`), layout coordinator (`FormFields`), persistence (`usePersistence`), and async search (`useAsyncQuery`).

---

## Form Construction Specification

> [!IMPORTANT]
> All forms MUST adhere to the normative specification:
> 👉 **[rules/form-spec.md](rules/form-spec.md)**

### Invariant Rules for Agents
1. **Schema-first**: Always declare a Zod schema (`z.object({...})`). Derive defaults via `createFormDefaults(schema)`. Never hand-write default objects.
2. **12-Column Grid**: Form rows must sum to ≤ 12 columns via `colspan`. Unset `colspan` defaults to 12.
3. **Provider Requirement**: `<InferFieldInput />` requires an ancestor `<MicroformProvider value={microform}>`.
4. **Registry Scope**: Instantiate `createMicroform({ controls })` at module level or memoize with `useMemo`. Never instantiate naked inside component bodies.
5. **Sanitization**: Mutate with `cleanVars(data, { empty: "null" })`. Query with `cleanVars(data, { empty: "omit" })`.

---

## On-Demand Reference Modules

Load the specific reference module required for the current task:

| Capability | Reference File |
| :--- | :--- |
| **Custom Controls & Registry** | [references/01-registry-and-controls.md](references/01-registry-and-controls.md) |
| **Layout Grid & Dynamic Rendering** | [references/02-structure-and-rendering.md](references/02-structure-and-rendering.md) |
| **Draft Persistence & Storage** | [references/03-ports-and-persistence.md](references/03-ports-and-persistence.md) |
| **Async Queries & Submit Flow** | [references/04-async-and-submit-flow.md](references/04-async-and-submit-flow.md) |
| **Canonical End-to-End Cheatsheet** | [references/05-recipes-and-cheatsheet.md](references/05-recipes-and-cheatsheet.md) |
