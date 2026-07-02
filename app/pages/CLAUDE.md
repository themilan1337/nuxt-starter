# Pages Best Practices

Routing conventions (file names, params, groups) are covered by the `pages` skill. These rules are about what goes INSIDE a page component.

- Pages compose, they do not implement: fetch via queries from `app/queries/`, render via components. NO business logic, no manual `$fetch`
- Keep pages thin. When a page grows, extract sections into feature components
- ALWAYS call `useSeoMeta()` with at least `title` and `description` on every page
- Use `definePageMeta()` for `layout`, `middleware`, and route `meta`
- ALWAYS render ALL query states: loading (`asyncStatus === 'loading'`), error (`state.status === 'error'`), empty data, and success. No blank screens
- Read params through the typed route, `useRoute('/users/[userId]')`, and pass them to queries with a getter so they stay reactive
- Redirects and access control belong in `app/middleware/`, NOT inside page setup

## Example

```vue
<script setup lang="ts">
// app/pages/users/[userId].vue
import { userByIdQuery } from '@/queries/users'

definePageMeta({ middleware: ['auth'] })

const route = useRoute('/users/[userId]')
const { state, asyncStatus } = useQuery(() => userByIdQuery(route.params.userId))

useSeoMeta({
  title: () => state.value.data?.name ?? 'Profile',
  description: 'User profile page',
})
</script>

<template>
  <UserProfileSkeleton v-if="asyncStatus === 'loading'" />
  <UiErrorMessage v-else-if="state.status === 'error'" :error="state.error" />
  <UserProfile v-else-if="state.data" :user="state.data" />
  <UiEmptyState v-else message="User not found" />
</template>
```
