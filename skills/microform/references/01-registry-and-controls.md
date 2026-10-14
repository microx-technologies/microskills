# Registro de Controles y Extensibilidad (OCP)

En `microform`, los controles son funciones de renderizado desacopladas de cualquier librería visual fija. El principio Open/Closed permite crear cualquier input personalizado sin modificar el núcleo.

---

## 1. Definir un Control con `defineControl`

Cada control declara su identificador `type` y su función renderizadora:

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
      className="input-class"
    />
  ),
});
```

### Argumentos del Renderer Context:
- `value`: Valor actual del campo.
- `inputValue`: Representación en string escalar inferida automáticamente.
- `onChange`: Callback para notificar cambios a React Hook Form.
- `onBlur`: Callback para marcar el campo como tocado (*touched*).
- `disabled`: Booleano opcional si el campo está inhabilitado.
- `name`: Nombre del campo en el formulario.
- `props`: Atributos adicionales definidos en la estructura (ej: `placeholder`, `options`, `min`, `max`).

---

## 2. Controles Personalizados Ricos

Para controles avanzados (ejemplo: un selector de calificación con estrellas):

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
            className={star <= current ? "text-yellow-400" : "text-gray-300"}
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

## 3. Crear la Instancia del Registry

Agrupa los controles usando `createMicroform`:

```tsx
import { createMicroform } from "microform";
import { textControl } from "./controls/text";
import { ratingControl } from "./controls/rating";

export const microform = createMicroform({
  controls: [textControl, ratingControl],
});
```

### Métodos de la Instancia `microform`:
- `register(type, renderer)`: Registra un control adicional dinámicamente.
- `registerAll(renderersMap)`: Registra múltiples controles en bloque.
- `resolve(type)`: Obtiene el renderer correspondiente a un tipo.
- `has(type)`: Verifica si un tipo de control está registrado.
- `types()`: Retorna un array con todos los tipos disponibles.
