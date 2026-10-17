# Draft Persistence & Storage Ports

`microform` uses a Ports and Adapters pattern for persistence, decoupling storage drivers (`localStorage`, `sessionStorage`, `IndexedDB`, or in-memory test mocks) from form components.

## 1. StorageAdapter Port

Any storage engine implements `StorageAdapter`:

```ts
import type { StorageAdapter } from "@microx-technologies/microform";

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
      } catch {}
    },
    remove: (key: string) => {
      try {
        localStorage.removeItem(key);
      } catch {}
    },
  };
}
```

> **Testing**: `microform` exports `createMemoryStorage({ [key]: value })` for headless or unit test suites without DOM APIs.

---

## 2. Automatic Field Sync via `usePersistence`

Binds a specific React Hook Form field to storage. Restores persisted value on mount and writes updates on change:

```tsx
import { useForm } from "react-hook-form";
import { usePersistence } from "@microx-technologies/microform/react";
import { createLocalStorageAdapter } from "./adapters";

const storage = createLocalStorageAdapter();

export function DraftUserEditor() {
  const form = useForm({ defaultValues: { name: "", notes: "" } });

  // Sync field "name"
  usePersistence({
    control: form.control,
    name: "name",
    storage,
    persistKey: "draft_user_name",
    onRestore: (val) => form.setValue("name", String(val)),
  });

  // Sync field "notes"
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

### `usePersistence` Parameters:
- `control`: React Hook Form `control` instance.
- `name`: Target field name.
- `storage`: `StorageAdapter` implementation.
- `persistKey`: Unique storage identifier.
- `prefix`: Key prefix (defaults to `"form-persist:"`).
- `onRestore`: Invoked on mount if a persisted value is found.
