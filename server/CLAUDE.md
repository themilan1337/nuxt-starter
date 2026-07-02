# Server Routes Best Practices

The Nuxt server runs on Nitro (h3). Files in `server/` define routes by name.

- File name defines route and HTTP method: `server/api/users.get.ts` -> `GET /api/users`, `users.post.ts` -> `POST /api/users`, `users/[userId].get.ts` -> `GET /api/users/:userId`
- ALWAYS add the method suffix: `.get.ts`, `.post.ts`, `.patch.ts`, `.delete.ts`
- ALWAYS use descriptive param names: `[userId].get.ts`, NOT `[id].get.ts`
- ALWAYS wrap handlers in `defineEventHandler()` and return plain data (object, array). Nuxt serializes it, there is no `res.json()`
- ALWAYS validate user input at the route boundary with `readValidatedBody()` / `getValidatedQuery()` and a schema. NEVER use raw `readBody()`/`getQuery()` results without validation
- Read route params with `getRouterParam(event, 'userId')`
- Errors: `throw createError({ statusCode, statusMessage })`. NEVER leak internals (stack traces, SQL, secrets) in error messages
- Keep routes thin: business logic, DB access, and external API calls live in `server/utils/` (auto-imported in all handlers)
- Access config with `useRuntimeConfig(event)`, NEVER `process.env`
- Cache expensive work: `defineCachedEventHandler` for routes, `defineCachedFunction` for utils (set `maxAge`, add `swr: true`)
- Fire-and-forget work (analytics, logging) goes in `event.waitUntil(promise)` so it does not block the response
- Response types shared with the client live in `shared/types/`, then `$fetch('/api/users')` infers the response type end to end
- Nuxt 4 uses h3 v1 and Nitro v2: patterns from h3 v2 / Nitro v3 docs DO NOT apply. Docs: <https://v1.h3.dev> and <https://nitro.build>

## Example

```ts
// server/api/users.post.ts
import { z } from 'zod'

const CreateUserSchema = z.object({
  name: z.string().min(1),
  email: z.email(),
})

export default defineEventHandler(async (event) => {
  const body = await readValidatedBody(event, CreateUserSchema.parse)
  // createUser comes from server/utils/users.ts (auto-imported)
  const user = await createUser(body)
  setResponseStatus(event, 201)
  return user
})
```

## Server Middleware

`server/middleware/` runs on EVERY request (pages and API alike). Use it only for cross-cutting concerns like auth context or logging. Attach data via `event.context`, NEVER return a response from server middleware.

```ts
// server/middleware/auth.ts
export default defineEventHandler(async (event) => {
  const session = await getUserSession(event) // server/utils/session.ts
  event.context.user = session?.user ?? null
})
```
