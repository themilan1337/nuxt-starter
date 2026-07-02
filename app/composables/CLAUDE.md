# Composables Best Practices

- ALWAYS check [VueUse](https://vueuse.org) first (installed via `@vueuse/nuxt`, auto-imported). Most patterns already exist, do not rewrite them
- Composables are for reusable stateful logic. Pure functions go in `app/utils/`, API data fetching goes in `app/queries/` (Pinia Colada), global state goes in `app/stores/` (Pinia)
- Only top-level files of `app/composables/` are auto-imported: `composables/useFoo.ts` yes, `composables/auth/useFoo.ts` no
- ALWAYS prefix with `use`, file name = function name: `useCounter.ts` exports `useCounter()`
- ALWAYS call composables at the top level of `<script setup>` or another composable, NEVER inside conditionals, loops, or event handlers
- NEVER make a composable `async`: an `await` before it loses the component context. Expose a sync composable with an async `execute()` action instead
- NEVER create reactive state at module scope: on the server it is shared by ALL requests

```ts
// ❌ Module scope: leaks between SSR requests
const user = ref<User | null>(null)
export function useUser() {
  return { user }
}

// ✅ State created per call (or put it in a Pinia store)
export function useUser() {
  const user = ref<User | null>(null)
  return { user }
}
```

- Accept `MaybeRefOrGetter` arguments and unwrap with `toValue()` so callers can pass values, refs, or getters
- Return a plain object of refs and functions. Wrap state the caller must not mutate in `readonly()`
- ALWAYS clean up side effects: `onScopeDispose()` for listeners and timers, `onWatcherCleanup()` inside watchers
- Guard browser-only APIs (`localStorage`, geolocation, observers) with `import.meta.client` so the composable stays SSR-safe

## Example

```ts
// app/composables/useClock.ts
export function useClock(intervalMs: MaybeRefOrGetter<number> = 1000) {
  const now = ref(new Date())

  if (import.meta.client) {
    const timer = setInterval(() => {
      now.value = new Date()
    }, toValue(intervalMs))
    onScopeDispose(() => clearInterval(timer))
  }

  return { now: readonly(now) }
}
```
