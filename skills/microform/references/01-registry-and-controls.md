# Control Registry & Extensibility (OCP)

Controls in `microform` are pure render functions decoupled from visual design systems. New controls extend the registry without altering the core package.

## 1. Defining a Control (`defineControl`)

```tsx
import { defineControl } from "microform";

export const textControl = defineControl({
  type: "text",
  renderer: ({ value, inputValue, onChange, onBlur, disabled, name, props }) => (
    <input
      id={name}
      name={name}
      type="text"
      value={value ?? ""}
      onChange={(e) => onChange?.(e.target.value)}
      onBlur={onBlur}
      disabled={disabled}
      placeholder={props.placeholder as string}
    />
  ),
});
```

### Renderer Context Properties:
- `value`: Raw value from form state.
- `inputValue`: String representation for scalar inputs.
- `onChange`: State update callback `(val: Value | undefined) => void`.
- `onBlur`: Touched state trigger `() => void`.
- `disabled`: Boolean field state.
- `name`: Target field name.
- `props`: Passthrough props declared on the field structure (`placeholder`, `options`, `min`, `max`, etc.).

---

## 2. Rich Custom Controls

```tsx
export const ratingControl = defineControl({
  type: "rating",
  renderer: ({ value, onChange, disabled }) => {
    const current = Number(value) || 0;
    return (
      <div className="flex gap-1">
        {[1, 2, 3, 4, 5].map((star) => (
          <button
            key={star}
            type="button"
            disabled={disabled}
            onClick={() => onChange?.(star)}
            className={star <= current ? "text-amber-400" : "text-zinc-300"}
          >
            ★
          </button>
        ))}
      </div>
    );
  },
});
```

---

## 3. Creating & Extending the Registry

```tsx
import { createMicroform } from "microform";
import { textControl } from "./controls/text";
import { ratingControl } from "./controls/rating";

// Static registry instantiation
export const microform = createMicroform({
  controls: [textControl, ratingControl],
});

// Dynamic registration at runtime
microform.register("color_picker", ({ value, onChange }) => (
  <input type="color" value={value} onChange={(e) => onChange?.(e.target.value)} />
));
```

### Registry API:
- `register(type, renderer)`: Adds or overrides a single control.
- `registerAll({ type: renderer })`: Batch registers controls.
- `resolve(type)`: Retrieves the renderer for a type identifier.
- `has(type)`: Boolean check for registered type.
- `types()`: Array of all registered type keys.
