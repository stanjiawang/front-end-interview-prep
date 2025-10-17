# Front-End System Design Deep Dive — Form Builder / Dynamic Schema

> Build a **schema-driven form engine**: render UI from JSON schema, validations, dependencies, conditional logic, and autosave. React + TypeScript.

---

## 1. Requirements
- Define forms via JSON schema (fields, types, validations).
- Conditional fields & visibility (depends on other values).
- Client and server validation; error messages.
- Autosave drafts; undo/retry.

---

## 2. Architecture
```
FormEngine
 ├─ SchemaParser → Field components map
 ├─ FormState (values, errors, touched, dirty)
 ├─ Validators (sync/async)
 ├─ DependencyEngine (show/hide/disable rules)
 └─ Persistence (autosave)
```

---

## 3. Tech Choices

| Area | Option | Why | Alternatives |
|---|---|---|---|
| State | **react-hook-form** | Minimal re-render, good perf | Formik, custom |
| Schema | **Zod / Yup** | Type-safe validation | JSON Schema + Ajv |
| UI | Headless fields + design system | Reusable, themeable | MUI, Chakra |
| Persistence | Autosave via debounce | Smooth UX | Manual save only |

---

## 4. Implementation

```tsx
type FieldSchema = { name: string; label: string; type: "text"|"number"|"select"; required?: boolean; when?: { name: string; equals: any } };

function Field({ s, register, watch }: any) {
  if (s.when && watch(s.when.name) !== s.when.equals) return null;
  switch (s.type) {
    case "text": return <input {...register(s.name)} aria-label={s.label} />;
    case "number": return <input type="number" {...register(s.name)} aria-label={s.label} />;
    case "select": return <select {...register(s.name)} aria-label={s.label} />;
  }
}

export function DynamicForm({ schema, onSubmit }: { schema: FieldSchema[]; onSubmit: (v:any)=>void }) {
  const { register, handleSubmit, watch } = require("react-hook-form").useForm();
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {schema.map(s => <Field key={s.name} s={s} register={register} watch={watch} />)}
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## 5. Performance & A11y
- Avoid re-render storms: uncontrolled inputs or RHF.
- Keyboard nav and labels; error region live announcements.

---

## 6. Testing
- Schema variations; conditional visibility.
- Validation rules; autosave debounce.

---

## 7. Interview Summary
> “A schema→UI engine with react-hook-form minimizes renders; Zod handles validation; dependency rules control visibility; autosave debounced to persist drafts.”
