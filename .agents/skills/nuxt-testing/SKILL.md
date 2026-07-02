---
name: nuxt-testing
description: >
  Test Nuxt 4 apps with @nuxt/test-utils + Vitest.
  TRIGGER when: writing or editing *.spec.ts / *.test.ts in a Nuxt project, testing components or
  composables that use Nuxt auto-imports, mocking Nuxt imports or API endpoints, configuring Vitest
  for Nuxt.
---

# Nuxt Testing

## Rules

- Components and composables that touch Nuxt context run in the `nuxt` environment with helpers from `@nuxt/test-utils/runtime`
- ALWAYS `await mountSuspended(Component)`: it supports async setup, auto-imports, and plugins. Plain `mount()` from `@vue/test-utils` is ONLY for pure Vue components with zero Nuxt dependencies
- Mock server endpoints with `registerEndpoint()`, mock auto-imports with `mockNuxtImport()`. NEVER stub `$fetch` globally
- Test behavior through rendered output and emitted events, not implementation details
- Priorities: pure logic in `shared/utils/` and `server/utils/` first (cheap, plain environment), then queries and components with logic, thin UI last
- Pure functions do NOT need the nuxt environment: keep them fast with `// @vitest-environment node` at the top of the spec

## Setup

```bash
pnpm i -D @nuxt/test-utils vitest @vue/test-utils happy-dom
```

```ts
// vitest.config.ts
import { defineVitestConfig } from '@nuxt/test-utils/config'

export default defineVitestConfig({
  test: {
    environment: 'nuxt',
  },
})
```

## Component Test

```ts
import { mountSuspended } from '@nuxt/test-utils/runtime'
import UserCard from './UserCard.vue'

it('renders the user name', async () => {
  const wrapper = await mountSuspended(UserCard, {
    props: { userId: '1' },
  })
  expect(wrapper.text()).toContain('Alice')
})
```

`mountSuspended` also accepts `route: '/users/1'` to mount at a specific route and exposes `wrapper.setupState` for rare cases where output alone is not enough.

## Mock API Endpoints

```ts
import { registerEndpoint } from '@nuxt/test-utils/runtime'

registerEndpoint('/api/users/1', () => ({ id: '1', name: 'Alice' }))

// Match a specific HTTP method
registerEndpoint('/api/users', {
  method: 'POST',
  handler: () => ({ created: true }),
})
```

## Mock Nuxt Auto-Imports

`mockNuxtImport` is hoisted: one mock per import per test file. Use `vi.hoisted` when the mock must change between tests:

```ts
import { mockNuxtImport } from '@nuxt/test-utils/runtime'

const { useRouteMock } = vi.hoisted(() => ({
  useRouteMock: vi.fn(() => ({ params: { userId: '1' } })),
}))

mockNuxtImport('useRoute', () => useRouteMock)

it('reads the route param', async () => {
  useRouteMock.mockReturnValue({ params: { userId: '2' } })
  // ...
})
```

## What NOT to Test

- Nuxt and Vue internals (routing itself, reactivity)
- Markup details that change often. Assert on text, roles, and emitted events
- Third-party library behavior
