# Persistencia de Borradores y Puertos de Almacenamiento

`microform` implementa el patrón de Puertos y Adaptadores para desacoplar el motor de persistencia del navegador (`localStorage`, `sessionStorage`, `IndexedDB` o memorias de prueba).

---

## 1. El Puerto `StorageAdapter`

Todo adaptador de almacenamiento implementa la interfaz `StorageAdapter`:

```ts
import type { StorageAdapter } from "microform";

export function createLocalStorageAdapter(): StorageAdapter {
  return {
    get: (key: string) => {
      try {
        return localStorage.getItem(key);
      } catch {
        return null;
      }
    },
    set: (key: string, value: string) => {
      try {
        localStorage.setItem(key, value);
      } catch {
        // quota exceeded o navegación privada
      }
    },
    remove: (key: string) => {
      try {
        localStorage.removeItem(key);
      } catch {}
    },
  };
}
```

> **Tip**: En pruebas unitarias, `microform` incluye `createMemoryStorage({ "initial:key": "value" })` que no depende del DOM.

---

## 2. Autoguardado con el Hook `usePersistence`

Conecta campos individuales de React Hook Form al adaptador de almacenamiento. Restaura el valor al montar y sincroniza cada cambio:

```tsx
import { useForm } from "react-hook-form";
import { usePersistence } from "microform/react";
import { createLocalStorageAdapter } from "./adapters";

const storage = createLocalStorageAdapter();

export function UserEditor() {
  const form = useForm({ defaultValues: { name: "", notes: "" } });

  // Persistir campo "name"
  usePersistence({
    control: form.control,
    name: "name",
    storage,
    persistKey: "draft_user_name",
    onRestore: (val) => form.setValue("name", String(val)),
  });

  // Persistir campo "notes"
  usePersistence({
    control: form.control,
    name: "notes",
    storage,
    persistKey: "draft_user_notes",
    onRestore: (val) => form.setValue("notes", String(val)),
  });

  const onReset = () => {
    storage.remove?.("form-persist:draft_user_name");
    storage.remove?.("form-persist:draft_user_notes");
    form.reset();
  };

  return ( ... );
}
```

### Opciones de `usePersistence`:
- `control`: El objeto `control` retornado por `useForm()`.
- `name`: Nombre del campo en el formulario.
- `storage`: Instancia de `StorageAdapter`.
- `persistKey`: Clave única para identificar el dato en el almacenamiento.
- `prefix`: Prefijo de clave opcional (por defecto `form-persist:`).
- `onRestore`: Callback ejecutado cuando se encuentra un valor persistido previamente.
