# Estructuras de Layout y Renderizado Dinámico

`microform` desacopla la definición del formulario del marcado HTML usando estructuras declarativas tipadas (`FieldStructure`) y componentes de renderizado de React.

---

## 1. Definir una `FieldStructure`

Una estructura organiza los campos en **secciones**, **filas** y **columnas** (basadas en un grid de 12 columnas):

```tsx
import type { FieldStructure } from "microform";

export type ProfileFormValues = {
  fullName: string;
  age: number;
  email: string;
  bio?: string;
};

export const profileStructure: FieldStructure<keyof ProfileFormValues> = [
  {
    kind: "section",
    key: "personal",
    title: "Información Personal",
    description: "Datos de contacto iniciales",
    children: [
      {
        fields: [
          {
            type: "text",
            name: "fullName",
            label: "Nombre Completo",
            placeholder: "Ada Lovelace",
            colspan: 8, // 8 de 12 columnas
          },
          {
            type: "number",
            name: "age",
            label: "Edad",
            colspan: 4, // 4 de 12 columnas (8 + 4 = 12)
          },
        ],
      },
      {
        type: "email",
        name: "email",
        label: "Correo Electrónico",
        // Sin colspan = 12 columnas (ancho completo)
      },
    ],
  },
  {
    type: "textarea",
    name: "bio",
    label: "Biografía",
  },
];
```

---

## 2. Inyección con `MicroformProvider` y `FormFields`

Conecta React Hook Form con la estructura mediante `FormFields` e `InferFieldInput`:

```tsx
import { useForm } from "react-hook-form";
import { FormFields, InferFieldInput, MicroformProvider } from "microform/react";
import { microform } from "./microform-setup";
import { profileSchema } from "./schema";

export function ProfileForm() {
  const form = useForm({
    defaultValues: createFormDefaults(profileSchema),
  });

  return (
    <MicroformProvider value={microform}>
      <FormFields
        structure={profileStructure}
        control={form.control}
        schema={profileSchema}
        renderItem={(field, { rhf, fieldState, required }) => (
          <div className="space-y-1.5">
            <label htmlFor={rhf.name} className="block text-sm font-medium">
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
              <p className="text-xs text-gray-500">{field.description}</p>
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

## 3. Formulario Completo de Alto Nivel con `StructForm`

Si deseas un componente contenedor estándar que incluya el `<form>` y el botón de submit:

```tsx
import { StructForm } from "microform/react";

<MicroformProvider value={microform}>
  <StructForm
    structure={profileStructure}
    form={form}
    schema={profileSchema}
    onSubmit={onSubmit}
    submitLabel="Guardar Cambios"
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
