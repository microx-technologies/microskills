---
name: microform
description: >-
  Headless form engine and data-entry toolkit with typed control registry, Zod defaults,
  12-column grid structures, draft persistence, async sources, and submit orchestration.
  Use when creating or refactoring forms, generating form fields from schemas,
  handling draft auto-save, building custom form controls, or managing async queries in microform.
  Do not use for raw HTML forms without React or simple single-input forms without microform.
---

# Microform Skill

`microform` es un motor de formularios headless desacoplado basado en arquitectura de **Puertos y Adaptadores** y el principio **Open/Closed (OCP)**. Separa la lógica de negocio, validaciones y diseño estructural del renderizado visual.

---

## 🏛 Arquitectura en 2 Paquetes

- **Core (`microform`)**: Agnóstico al framework. Maneja el registro de controles (`createMicroform`), valores por defecto automáticos desde Zod (`createFormDefaults`), mapas de campos requeridos (`buildRequiredMap`), limpieza de variables (`cleanVars`) y puertos.
- **React (`microform/react`)**: Integración con React 19 y React Hook Form (`MicroformProvider`, `FormFields`, `InferFieldInput`, `usePersistence`, `useAsyncQuery`).

---

## 📜 Especificación Obligatoria (Form Spec)

> [!IMPORTANT]
> Todo formulario creado o modificado con `microform` debe seguir las reglas normativas de la especificación:
> 👉 **Lee la especificación completa en [rules/form-spec.md](rules/form-spec.md)**.

### Resumen de Reglas Clave
1. **Schema-First**: Define el esquema con `zod` y extrae defaults con `createFormDefaults(schema)`.
2. **Layout Grid de 12 Columnas**: Usa `FieldStructure<keyof FormValues>` asegurando que las filas con múltiples campos sumen 12 columnas (`colspan`).
3. **Provider Contextual**: Toda vista que use `<InferFieldInput />` debe estar envuelta en `<MicroformProvider value={microform}>`.
4. **Instanciación Fuera del Render**: Define `createMicroform({ controls })` a nivel de módulo o con `useMemo`.
5. **Limpieza en Submit**: Transforma strings vacíos con `cleanVars(data, { empty: "null" })` al enviar a APIs.

---

## 📚 Guías Detalladas de Referencia (Carga bajo demanda)

El agente debe consultar únicamente el archivo relevante para la tarea que esté resolviendo:

| Tarea / Caso de Uso | Documento de Referencia |
| :--- | :--- |
| **Definir Controles y Registry** | [references/01-registry-and-controls.md](references/01-registry-and-controls.md) |
| **Estructuras, Grids y FormFields** | [references/02-structure-and-rendering.md](references/02-structure-and-rendering.md) |
| **Persistencia de Borradores (Drafts)** | [references/03-ports-and-persistence.md](references/03-ports-and-persistence.md) |
| **Búsqueda Asíncrona y Flujo de Submit** | [references/04-async-and-submit-flow.md](references/04-async-and-submit-flow.md) |
| **Recetas y Ejemplo Completo (CheatSheet)** | [references/05-recipes-and-cheatsheet.md](references/05-recipes-and-cheatsheet.md) |
