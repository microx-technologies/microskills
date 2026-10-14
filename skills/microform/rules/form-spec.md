# Especificación de Construcción de Formularios (Form Spec)

> **Nota**: Esta es la especificación base y normativa para construir formularios consistentes utilizando **microform**. Cualquier agente que genere o modifique formularios debe adherirse a estas pautas.

---

## 1. Principio Schema-First con Zod
- **Obligatorio**: Todo formulario debe originarse a partir de un esquema de Zod (`z.object({...})`).
- **Inferencia de Tipos**: El tipo de datos del formulario se infiere del esquema:
  ```ts
  const userSchema = z.object({ ... });
  type UserFormValues = z.infer<typeof userSchema>;
  ```
- **Defaults automáticos**: Prohibido duplicar valores por defecto manualmente. Siempre usa:
  ```ts
  const defaultValues = React.useMemo(() => createFormDefaults(userSchema), []);
  ```

---

## 2. Definición Estructural (Layout y Grid)
- Los campos visuales y su orden se declaran mediante `FieldStructure<keyof FormValues>`.
- **Sistema de Grid de 12 Columnas**:
  - Toda fila con múltiples campos debe sumar un total de 12 columnas distribuidas con `colspan` (ej: `colspan: 6` y `colspan: 6`, o `colspan: 8` y `colspan: 4`).
  - Si un campo no define `colspan`, ocupa el ancho completo (12 columnas por defecto).
- **Secciones Semánticas**: Agrupa campos relacionados usando `{ kind: "section", title: "...", description: "...", children: [...] }`.

---

## 3. Inyección y Catálogo de Controles (Registry)
- **Instancia del Registro**: Agrupa los controles con `createMicroform({ controls })` a nivel de módulo o memoizado con `useMemo`. Nunca lo instancies dentro del cuerpo de la función del componente sin memoizar.
- **MicroformProvider**: Si el formulario utiliza renderizado dinámico (`FormFields` + `InferFieldInput`), envuelve el contenedor en `<MicroformProvider value={microform}>`.
- **Controles Personalizados**: Si se requiere un nuevo tipo de input (ej. `rating`, `color_picker`), créalo mediante `defineControl({ type, renderer })` y añádelo a la lista de controles.

---

## 4. Renderizado y Manejo de Errores
- Al usar `FormFields`, el callback `renderItem(field, { rhf, fieldState, required })` debe:
  1. Renderizar la etiqueta (`<Label>`) mostrando un indicador visual obligatorio (`*`) si `required` es `true`.
  2. Renderizar `<InferFieldInput />` conectando `value={rhf.value}`, `onChange={rhf.onChange}`, `onBlur={rhf.onBlur}` y `disabled={field.disabled}`.
  3. Renderizar la descripción del campo si `field.description` existe.
  4. Renderizar el mensaje de validación accesible si `fieldState.error` existe (`fieldState.error.message`).

---

## 5. Sanitización y Flujo de Envío (Submit)
- **Limpieza de Variables**:
  - En mutaciones (APIs REST / GraphQL), sanitiza los valores con `cleanVars(data, { empty: "null" })` para convertir cadenas vacías `""` en `null`.
  - En filtros y búsquedas, usa `cleanVars(data, { empty: "omit" })` para omitir propiedades vacías.
- **Orquestación de Envío**:
  - Se recomienda usar `handleSubmitFlow({ mutation, vars, notify, successMessage, resetForm, invalidate })` para encapsular carga, notificaciones de éxito/error y limpieza de borradores.

---

## 6. Persistencia de Borradores (Drafts)
- Para formularios extensos o propensos a recargas accidentales, conecta los campos sensibles a un `StorageAdapter` usando el hook `usePersistence`:
  ```tsx
  usePersistence({
    control: form.control,
    name: "fieldKey",
    storage,
    persistKey: "draft_<entity>_<fieldKey>",
    onRestore: (val) => form.setValue("fieldKey", val),
  });
  ```
- Al completar un submit exitoso, el almacenamiento de dicho borrador debe limpiarse (`storage.remove?.(...)`).
