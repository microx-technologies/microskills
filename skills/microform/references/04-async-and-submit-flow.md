# Consultas Asíncronas, Limpieza y Submit Flow

`microform` proporciona utilidades de arquitectura limpia para el manejo de fuentes remotas con paginación/debounce, sanitización de datos y orquestación de mutaciones.

---

## 1. Limpieza de Variables con `cleanVars`

Normaliza objetos y arrays recursivamente antes de enviarlos a mutaciones o filtros:

```ts
import { cleanVars } from "microform";

const raw = {
  search: "  hola mundo  ",
  emptyField: "   ",
  age: 25,
};

// Modo "null" (ideal para GraphQL o APIs REST que esperan null en campos vacíos):
const forMutation = cleanVars(raw, { empty: "null" });
// { search: "hola mundo", emptyField: null, age: 25 }

// Modo "omit" (por defecto, ideal para queries, filtros y URLs):
const forQuery = cleanVars(raw);
// { search: "hola mundo", age: 25 } — omite 'emptyField'
```

---

## 2. Búsqueda y Paginación Asíncrona con `useAsyncQuery`

Maneja debouncing automático, protección contra respuestas obsoletas (*race conditions*) y estado de carga:

```tsx
import { useState } from "react";
import { useAsyncQuery } from "microform/react";
import type { QuerySource } from "microform";

interface UserItem {
  id: string;
  name: string;
}

// 1. Definir la fuente (QuerySource)
const userSource: QuerySource<UserItem> = {
  fetch: async ({ search = "", page = 1, pageSize = 10 }) => {
    const res = await fetch(`/api/users?q=${search}&page=${page}&limit=${pageSize}`);
    const json = await res.json();
    return {
      items: json.data,
      total: json.totalCount,
    };
  },
};

// 2. Consumir en componente
export function UserPicker() {
  const [search, setSearch] = useState("");
  const [page, setPage] = useState(1);

  const { data, total, isLoading, refetch } = useAsyncQuery({
    source: userSource,
    search,
    page,
    pageSize: 10,
    debounceMs: 300,
  });

  return ( ... );
}
```

---

## 3. Orquestación del Envío con `handleSubmitFlow`

Ejecuta el ciclo de vida completo de un submit: sanitización con `cleanVars({ empty: "null" })`, ejecución, notificación de éxito o error, reseteo del formulario e invalidación de caché:

```ts
import { handleSubmitFlow } from "microform";
import type { NotifyAdapter } from "microform";
import { toast } from "sonner"; // o cualquier UI de toast

const notify: NotifyAdapter = {
  success: (msg) => toast.success(msg),
  error: (msg, detail) => toast.error(msg, { description: detail }),
};

const onSubmit = async (values: FormValues) => {
  const ok = await handleSubmitFlow({
    mutation: async (cleanedVars) => {
      return await api.updateUser(cleanedVars);
    },
    vars: values, // Se limpia automáticamente
    notify,
    successMessage: "Usuario actualizado con éxito",
    resetForm: () => {
      localStorage.removeItem("draft_user");
      form.reset();
    },
    invalidate: () => {
      refetchUsersList();
    },
  });

  if (ok) {
    console.log("Operación completada exitosamente");
  }
};
```
