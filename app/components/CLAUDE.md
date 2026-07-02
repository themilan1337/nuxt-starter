# Vue Components Best Practices

- Components in `app/components/` are auto-imported, NEVER import them manually
- Component name = full directory path: `ui/UiButton.vue` -> `<UiButton>`. Nuxt deduplicates repeated segments, so the filename MUST spell the full component name: `ui/UiButton.vue`, NOT `ui/Button.vue`
- ALWAYS use PascalCase for component names in source code
- Compose names from the most general to the most specific: `SearchButtonClear.vue` not `ClearSearchButton.vue`
- ALWAYS define props with `defineProps<{ propOne: number }>()` and TypeScript types, WITHOUT `const props =`
- Use `const props =` ONLY if you need the whole props object in the script block
- Destructure props to declare default values. Destructured props stay reactive, but wrap them in a getter when passing to watchers or composables: `watch(() => count, ...)`
- ALWAYS define emits with `const emit = defineEmits<{ eventName: [argOne: type]; otherEvent: [] }>()` for type safety
- ALWAYS use camelCase in JS for props and emits, even if they are kebab-case in templates
- ALWAYS use kebab-case in templates for props and emits
- ALWAYS use the prop shorthand if possible: `<MyComponent :count />` instead of `<MyComponent :count="count" />` (value has the same name as the prop)
- ALWAYS Use the shorthand for slots: `<template #default>` instead of `<template v-slot:default>`
- ALWAYS use explicit `<template>` tags for ALL used slots
- ALWAYS use `defineModel<type>({ required, get, set, default })` to define allowed v-model bindings in components. This avoids defining `modelValue` prop and `update:modelValue` event manually
- ALWAYS use `useTemplateRef('name')` for template refs, not a bare `ref()`
- Wrap browser-only components in `<ClientOnly>` with a `#fallback` slot for the server render
- Prefix heavy conditional components with `Lazy` (`<LazyUserModal v-if="open" />`) and use lazy hydration for below-the-fold content: `<LazyCommentList hydrate-on-visible />`
- Keep components small and focused. Extract reusable logic into composables

## Examples

### Props with defaults

```vue
<script setup lang="ts">
// ✅ Reactive destructure with defaults (Vue 3.5+)
const { count = 0, label = 'Total' } = defineProps<{
  count?: number
  label?: string
}>()

// ⚠️ Wrap destructured props in a getter for watch sources
watch(
  () => count,
  (value) => console.log(value),
)
</script>
```

### defineModel()

```vue
<script setup lang="ts">
// ✅ Simple two-way binding for modelValue
const title = defineModel<string>()

// ✅ With options and modifiers
const [title, modifiers] = defineModel<string>({
  default: 'default value',
  required: true,
  get: (value) => value.trim(), // transform value before binding
  set: (value) => {
    if (modifiers.capitalize) {
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
    return value
  },
})
</script>
```

### Multiple Models

By default `defineModel()` assumes a prop named `modelValue` but if we want to define multiple v-model bindings, we need to give them explicit names:

```vue
<script setup lang="ts">
// ✅ Multiple v-model bindings
const firstName = defineModel<string>('firstName')
const age = defineModel<number>('age')
</script>
```

They can be used in the template like this:

```html
<UserForm v-model:first-name="user.firstName" v-model:age="user.age" />
```

### Modifiers & Transformations

Native elements `v-model` has built-in modifiers like `.lazy`, `.number`, and `.trim`. We can implement similar functionality in components, fetch and read <https://vuejs.org/guide/components/v-model.md#handling-v-model-modifiers> if the user needs that.

### useTemplateRef()

```vue
<script setup lang="ts">
const searchInput = useTemplateRef('search-input')

onMounted(() => {
  searchInput.value?.focus()
})
</script>

<template>
  <input ref="search-input" />
</template>
```
