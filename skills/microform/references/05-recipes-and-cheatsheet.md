# Recetas y Ejemplo Completo (CheatSheet)

Plantilla canónica completa para crear un formulario robusto con `microform`, `zod` y `react-hook-form`.

---

```tsx
import * as React from "react";
import { useForm } from "react-hook-form";
import { z } from "zod";
import {
  createFormDefaults,
  createMicroform,
  defineControl,
  type FieldStructure,
  cleanVars,
} from "microform";
import {
  FormFields,
  InferFieldInput,
  MicroformProvider,
} from "microform/react";

// ==========================================
// 1. ESQUEMA DE VALIDACIÓN CON ZOD
// ==========================================
export const projectSchema = z.object({
  title: z.string().min(3, "El título debe tener al menos 3 caracteres"),
  category: z.enum(["web", "mobile", "infra"]).default("web"),
  estimatedHours: z.number().min(1).default(10),
  description: z.string().optional(),
});

export type ProjectFormValues = z.infer<typeof projectSchema>;

// ==========================================
// 2. ESTRUCTURA VISUAL (GRID 12 COLUMNAS)
// ==========================================
export const projectStructure: FieldStructure<keyof ProjectFormValues> = [
  {
    kind: "section",
    key: "general",
    title: "Datos del Proyecto",
    children: [
      {
        fields: [
          {
            type: "text",
            name: "title",
            label: "Título",
            placeholder: "Ej: Plataforma de pagos",
            colspan: 8,
          },
          {
            type: "number",
            name: "estimatedHours",
            label: "Horas Estimadas",
            colspan: 4,
          },
        ],
      },
      {
        type: "select",
        name: "category",
        label: "Categoría",
        options: [
          { value: "web", label: "Web App" },
          { value: "mobile", label: "Mobile App" },
          { value: "infra", label: "Infraestructura" },
        ],
      },
      {
        type: "textarea",
        name: "description",
        label: "Descripción detallada",
        placeholder: "Describe los objetivos del proyecto...",
      },
    ],
  },
];

// ==========================================
// 3. REGISTRO DE CONTROLES
// ==========================================
const textControl = defineControl({
  type: "text",
  renderer: ({ value, onChange, onBlur, disabled, name, props }) => (
    <input
      id={name}
      name={name}
      type="text"
      value={value ?? ""}
      onChange={(e) => onChange?.(e.target.value)}
      onBlur={onBlur}
      disabled={disabled}
      placeholder={props.placeholder as string}
      className="w-full border rounded px-3 py-2 text-sm"
    />
  ),
});

const numberControl = defineControl({
  type: "number",
  renderer: ({ value, onChange, onBlur, disabled, name }) => (
    <input
      id={name}
      name={name}
      type="number"
      value={value ?? ""}
      onChange={(e) => onChange?.(e.target.valueAsNumber || undefined)}
      onBlur={onBlur}
      disabled={disabled}
      className="w-full border rounded px-3 py-2 text-sm"
    />
  ),
});

export const microform = createMicroform({
  controls: [textControl, numberControl],
});

// ==========================================
// 4. COMPONENTE DE FORMULARIO
// ==========================================
export function ProjectForm({ onSave }: { onSave: (data: ProjectFormValues) => void }) {
  const defaultValues = React.useMemo(() => createFormDefaults(projectSchema), []);

  const form = useForm<ProjectFormValues>({
    defaultValues,
  });

  const onSubmit = (rawValues: ProjectFormValues) => {
    const cleaned = cleanVars(rawValues, { empty: "null" });
    onSave(cleaned as ProjectFormValues);
  };

  return (
    <MicroformProvider value={microform}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        <FormFields
          structure={projectStructure}
          control={form.control}
          schema={projectSchema}
          renderItem={(field, { rhf, fieldState, required }) => (
            <div className="space-y-1">
              <label htmlFor={rhf.name} className="block text-sm font-semibold">
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
                <p className="text-xs text-red-500 font-medium">
                  {fieldState.error.message}
                </p>
              )}
            </div>
          )}
        />

        <div className="flex justify-end gap-2 pt-4">
          <button
            type="button"
            onClick={() => form.reset(defaultValues)}
            className="px-4 py-2 border rounded text-sm"
          >
            Restablecer
          </button>
          <button
            type="submit"
            className="px-4 py-2 bg-blue-600 text-white rounded text-sm font-medium"
          >
            Guardar Proyecto
          </button>
        </div>
      </form>
    </MicroformProvider>
  );
}
```
