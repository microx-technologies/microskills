# microform — Form Construction Specification

## 1. Intent

- **Outcome**: Deterministic, schema-first, headless form construction engine binding Zod, React Hook Form, and modular control registries. Decouples data validation and layout structure from UI components.
- **User**: AI agents and engineers implementing enterprise forms, backoffice data-entry suites, and dynamic configuration UIs.
- **Why now**: Traditional form setups tightly couple schema validation, field layout, and UI components, leading to duplicated default values, manual grid math, and fractured draft persistence.
- **Success**: A full 10-field form defined in <80 declarative lines. 100% type safety from Zod schema to control renderers. Zero hardcoded UI tags inside the engine.
- **Constraints**: React 19, `react-hook-form` ^7, `zod` ^4. 12-column layout grid. Ports and Adapters for side-effects (storage, notification, mutation).
- **Out of scope**: Visual styling/CSS, bundled UI kits, backend transport clients, server-action executors.

---

## 2. Invariant: Mechanism vs Policy

The framework provides mechanism; userland provides policy:

- **Framework MAY ship**: Typed registry, Zod defaults derivation (`createFormDefaults`), required map extraction (`buildRequiredMap`), variable cleaner (`cleanVars`), 12-column grid layout binding (`FieldGrid`, `FormFields`), React context (`MicroformProvider`), dynamic renderer dispatcher (`InferFieldInput`), draft persistence hook (`usePersistence`), debounced async query hook (`useAsyncQuery`), submit lifecycle orchestrator (`handleSubmitFlow`), memory storage adapter for testing.
- **Framework MUST NOT ship**: HTML tags, CSS styles, UI components (inputs, selects, buttons), global singleton registries, bundled HTTP clients, hardcoded toast libraries.
- Every visual primitive and side-effect adapter MUST be userland-injected.

---

## 3. Goals & Non-goals

### Goals
- Single source of truth: Schema declares types, constraints, and default values.
- Declarative layout: Sections, rows, and responsive spans declared via `FieldStructure`.
- Isolated registries: Multiple independent form registries coexisting without global state contamination.
- Pluggable ports: Storage, notifications, queries, and mutations injected as typed interfaces.

### Non-goals
- Form builder UI / drag-and-drop generator.
- Multi-framework support: Core is framework-agnostic; bindings target React 19 exclusively.
- Client-side validation replacement: Validation belongs to Zod and React Hook Form.

---

## 4. Core Concepts

| Concept | Meaning |
| :--- | :--- |
| **Schema** | Zod object schema (`z.object({...})`) defining data shape, constraints, and defaults. |
| **Defaults** | Scalar/fallback values derived automatically from schema via `createFormDefaults`. |
| **Structure** | Declarative array (`FieldStructure<T>`) defining sections, rows, and 12-column spans (`colspan`). |
| **Control** | Unit composed of a string `type` and a pure renderer function (`defineControl`). |
| **Registry** | Isolated container (`Microform`) mapping control types to renderers. Created via `createMicroform`. |
| **Provider** | React context (`MicroformProvider`) propagating a `Microform` instance to tree descendants. |
| **FormFields** | React component mapping `FieldStructure` into React Hook Form `Controller` slots. |
| **InferFieldInput** | Dynamic component resolving a field renderer from contextual or explicit registry. |
| **StorageAdapter** | Port for persistence (`get`, `set`, optional `remove`). Consumed by `usePersistence`. |
| **SubmitFlow** | Standardized mutation runner handling variable cleaning, execution, notifications, and cache invalidation. |

---

## 5. Normative Construction Rules (Binding on Agents)

Every form built using `microform` MUST comply with the following rules:

### Rule 1: Schema-First Authority
Forms MUST be anchored by a Zod schema. Form values type MUST be inferred directly:
```ts
const formSchema = z.object({ ... });
type FormValues = z.infer<typeof formSchema>;
```

### Rule 2: Automatic Defaults Derivation
Form state MUST NOT duplicate initial values manually. Initial state MUST be generated via `createFormDefaults`:
```ts
const defaultValues = React.useMemo(() => createFormDefaults(formSchema), []);
const form = useForm<FormValues>({ defaultValues });
```

### Rule 3: 12-Column Grid Budget
Layout MUST be declared via `FieldStructure<keyof FormValues>`. Every row containing multiple fields MUST satisfy `sum(colspan) <= 12`. Fields without explicit `colspan` default to 12 (full width).

### Rule 4: Registry Instantiation Scope
`createMicroform({ controls })` MUST be declared at module scope or memoized via `useMemo`. Instantiating registries inside component render bodies without memoization is FORBIDDEN.

### Rule 5: Contextual Provider Wrapping
Any view utilizing `<InferFieldInput />` MUST be enclosed within `<MicroformProvider value={microform}>` unless `microform={instance}` is passed as an explicit prop.

### Rule 6: Input Sanitization on Submit
- API mutations (REST / GraphQL) MUST sanitize payloads using `cleanVars(data, { empty: "null" })` to convert blank strings `""` to `null`.
- Search / filter queries MUST sanitize payloads using `cleanVars(data, { empty: "omit" })` to drop empty fields.
- Alternatively, use `handleSubmitFlow` which enforces `{ empty: "null" }` by default.

### Rule 7: Draft Persistence Lifecycle
When draft retention is required, fields MUST be bound using `usePersistence` against a typed `StorageAdapter`. Successful submission MUST explicitly purge persisted keys from storage before resetting the form.
