# Pinia Stores Best Practices

- Stores hold global CLIENT state: session/user, UI preferences, cross-page state (cart, drafts). Server data does NOT belong here, it lives in Pinia Colada queries (`app/queries/`)
- NEVER fetch data in a store: no `$fetch`, no `useFetch`. If a store needs server data, a component queries it and calls a store action with the result
- ALWAYS use the setup syntax: `defineStore('name', () => { ... })` with `ref`/`computed`/functions. NEVER use the options object syntax
- ALWAYS return every state ref from the setup function, Pinia needs them for SSR and devtools
- File name = store id: `app/stores/cart.ts` defines `useCartStore = defineStore('cart', ...)`
- In components, destructure state and getters with `storeToRefs(store)`. Actions can be destructured directly
- Keep derived state as `computed` inside the store instead of recomputing it in components
- For state that must not be mutated from outside, expose `readonly(state)` plus actions that mutate the internal ref

## Example

```ts
// app/stores/cart.ts
import type { CartItem } from '#shared/types/cart'

export const useCartStore = defineStore('cart', () => {
  const items = ref<CartItem[]>([])

  const total = computed(() =>
    items.value.reduce((sum, item) => sum + item.price * item.quantity, 0),
  )

  function add(item: CartItem) {
    items.value.push(item)
  }

  function clear() {
    items.value = []
  }

  return { items, total, add, clear }
})
```

```vue
<script setup lang="ts">
const cart = useCartStore()

// ✅ storeToRefs keeps state and getters reactive
const { items, total } = storeToRefs(cart)
// ✅ actions can be destructured directly
const { add, clear } = cart

// ❌ destructuring state directly breaks reactivity
// const { total } = cart
</script>
```
