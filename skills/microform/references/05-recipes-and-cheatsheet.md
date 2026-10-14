# Recipes & Canonical Cheatsheet

Complete, production-ready form binding `microform`, `zod`, and `react-hook-form`.

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

// 1. Zod Schema
export const projectSchema = z.object({
  title: z.string().min(3, "Title must have at least 3 characters"),
  category: z.enum(["web", "mobile", "infra"]).default("web"),
  estimatedHours: z.number().min(1).default(10),
  description: z.string().optional(),
});

export type ProjectFormValues = z.infer<typeof projectSchema>;

// 2. 12-Column Grid Layout Structure
export const projectStructure: FieldStructure<keyof ProjectFormValues> = [
  {
    kind: "section",
    key: "general",
    title: "Project Details",
    children: [
      {
        fields: [
          {
            type: "text",
            name: "title",
            label: "Title",
            placeholder: "e.g. Payment Gateway",
            colspan: 8,
          },
          {
            type: "number",
            name: "estimatedHours",
            label: "Estimated Hours",
            colspan: 4,
          },
        ],
      },
      {
        type: "select",
        name: "category",
        label: "Category",
        options: [
          { value: "web", label: "Web Application" },
          { value: "mobile", label: "Mobile Application" },
          { value: "infra", label: "Infrastructure" },
        ],
      },
      {
        type: "textarea",
        name: "description",
        label: "Description",
        placeholder: "Brief overview of deliverables...",
      },
    ],
  },
];

// 3. Control Registry
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

// 4. Form Component
export function ProjectForm({ onSave }: { onSave: (data: ProjectFormValues) => void }) {
  const defaultValues = React.useMemo(() => createFormDefaults(projectSchema), []);

  const form = useForm<ProjectFormValues>({ defaultValues });

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
                <p className="text-xs text-zinc-500">{field.description}</p>
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
            Reset
          </button>
          <button
            type="submit"
            className="px-4 py-2 bg-blue-600 text-white rounded text-sm font-medium"
          >
            Save Project
          </button>
        </div>
      </form>
    </MicroformProvider>
  );
}
```
