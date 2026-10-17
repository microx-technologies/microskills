# Async Queries, Variable Cleaning, and Submit Flow

`microform` supplies architectural primitives for remote paginated data sources, recursive payload sanitization, and standardized mutation lifecycles.

## 1. Variable Sanitization (`cleanVars`)

Recursively cleans objects and arrays prior to mutation or query dispatch:

```ts
import { cleanVars } from "microform";

const raw = {
  search: "  Jane Doe  ",
  emptyField: "   ",
  score: 100,
};

// "null" mode: Blank strings become null (GraphQL / SQL mutations)
const forMutation = cleanVars(raw, { empty: "null" });
// { search: "Jane Doe", emptyField: null, score: 100 }

// "omit" mode (default): Blank strings are stripped (search params / URL filters)
const forQuery = cleanVars(raw);
// { search: "Jane Doe", score: 100 }
```

---

## 2. Paginated Remote Source with `useAsyncQuery`

Handles built-in debouncing, stale response discard (race condition protection), and loading state:

```tsx
import { useState } from "react";
import { useAsyncQuery } from "microform/react";
import type { QuerySource } from "microform";

interface UserItem {
  id: string;
  name: string;
}

// 1. Declare query port adapter
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

// 2. Consume in component
export function UserSearchSelect() {
  const [search, setSearch] = useState("");
  const [page, setPage] = useState(1);

  const { data, total, isLoading, refetch } = useAsyncQuery({
    source: userSource,
    search,
    page,
    pageSize: 10,
    debounceMs: 250,
  });

  return ( ... );
}
```

---

## 3. Standardized Mutation Orchestration (`handleSubmitFlow`)

Coordinates sanitization (`cleanVars({ empty: "null" })`), execution, toast notifications, form reset, and cache invalidation:

```ts
import { handleSubmitFlow } from "microform";
import type { NotifyAdapter } from "microform";
import { toast } from "sonner";

const notify: NotifyAdapter = {
  success: (msg) => toast.success(msg),
  error: (msg, detail) => toast.error(msg, { description: detail }),
};

const onSubmit = async (values: FormValues) => {
  const ok = await handleSubmitFlow({
    mutation: async (cleanedVars) => {
      return await api.updateUser(cleanedVars);
    },
    vars: values, // Sanitized automatically
    notify,
    successMessage: "User profile updated",
    resetForm: () => {
      localStorage.removeItem("draft_user");
      form.reset();
    },
    invalidate: () => {
      refetchUserList();
    },
  });

  if (ok) {
    // Navigate or close modal
  }
};
```
