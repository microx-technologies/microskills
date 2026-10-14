# Layout Structures & Dynamic Rendering

`microform` defines forms as typed structural graphs (`FieldStructure`), separating layout topology from HTML markup.

## 1. Declaring a `FieldStructure`

Layouts organize fields into **sections**, **rows**, and **columns** using a 12-column grid budget:

```tsx
import type { FieldStructure } from "microform";

export type UserFormValues = {
  fullName: string;
  age: number;
  email: string;
  bio?: string;
};

export const userStructure: FieldStructure<keyof UserFormValues> = [
  {
    kind: "section",
    key: "personal",
    title: "Personal Information",
    description: "Primary contact details",
    children: [
      {
        fields: [
          {
            type: "text",
            name: "fullName",
            label: "Full Name",
            colspan: 8, // 8 / 12 columns
          },
          {
            type: "number",
            name: "age",
            label: "Age",
            colspan: 4, // 4 / 12 columns (8 + 4 = 12)
          },
        ],
      },
      {
        type: "email",
        name: "email",
        label: "Email Address",
        // Unset colspan defaults to 12
      },
    ],
  },
  {
    type: "textarea",
    name: "bio",
    label: "Biography",
  },
];
```

---

## 2. Rendering with `MicroformProvider` and `FormFields`

Binds React Hook Form to the structure. `renderItem` maps each field definition into UI markup and calls `InferFieldInput`:

```tsx
import { useForm } from "react-hook-form";
import { createFormDefaults } from "microform";
import { FormFields, InferFieldInput, MicroformProvider } from "microform/react";
import { microform } from "./microform-setup";
import { userSchema, userStructure, type UserFormValues } from "./schema";

export function UserForm() {
  const form = useForm<UserFormValues>({
    defaultValues: createFormDefaults(userSchema),
  });

  return (
    <MicroformProvider value={microform}>
      <FormFields
        structure={userStructure}
        control={form.control}
        schema={userSchema}
        renderItem={(field, { rhf, fieldState, required }) => (
          <div className="space-y-1.5">
            <label htmlFor={rhf.name} className="text-sm font-medium">
              {field.label}
              {required && <span className="text-red-500 ml-1">*</span>}
            </label>

            <InferFieldInput
              type={field.type}
              name={rhf.name}
              value={rhf.value}
              onChange={rhf.onChange}
              onBlur={rhf.onBlur}
              disabled={field.disabled}
              fieldProps={field}
            />

            {field.description && (
              <p className="text-xs text-zinc-500">{field.description}</p>
            )}

            {fieldState.error && (
              <p className="text-xs text-red-500">{fieldState.error.message}</p>
            )}
          </div>
        )}
      />
    </MicroformProvider>
  );
}
```

---

## 3. High-Level Form Wrapper (`StructForm`)

`StructForm` encapsulates `<form>` element handling and standard submit buttons:

```tsx
import { StructForm } from "microform/react";

<MicroformProvider value={microform}>
  <StructForm
    structure={userStructure}
    form={form}
    schema={userSchema}
    onSubmit={onSubmit}
    submitLabel="Save Changes"
    renderItem={(field, { rhf, required }) => (
      <InferFieldInput
        type={field.type}
        name={rhf.name}
        value={rhf.value}
        onChange={rhf.onChange}
      />
    )}
  />
</MicroformProvider>
```
