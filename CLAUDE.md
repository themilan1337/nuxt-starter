# PROJECT NAME

**THE USER MUST REPLACE THIS SECTION WITH THEIR OWN PROJECT NAME AND DESCRIPTION.**

EXAMPLE: A full-stack Nuxt 4 application with TypeScript, type safe file-based routing, server API routes, data fetching with Pinia Colada, state management, and comprehensive tooling.

## Standards

MUST FOLLOW THESE RULES, NO EXCEPTIONS

- Stack: Nuxt 4 (Vue 3 + Nitro), TypeScript, TailwindCSS v4, Pinia (`@pinia/nuxt`), Pinia Colada (`@pinia/colada-nuxt`)
- Patterns: ALWAYS use Composition API + `<script setup lang="ts">`, NEVER use Options API
- ALWAYS keep types alongside your code, use TypeScript for type safety, prefer `interface` over `type` for defining types
- Types used by both `app/` and `server/` MUST live in `shared/types/`
- Keep unit and integration tests alongside the file they test: `app/components/ui/UiButton.vue` + `app/components/ui/UiButton.spec.ts`
- ALWAYS use TailwindCSS classes rather than manual CSS
- DO NOT hard code colors, use Tailwind's color system
- ONLY add meaningful comments that explain why something is done, not what it does
- Dev server is already running on `http://localhost:3000` with HMR enabled. NEVER launch it yourself
- ALWAYS use named functions when declaring methods, use arrow functions only for callbacks
- ALWAYS prefer named exports over default exports. EXCEPTION: pages, layouts, plugins, middleware, and server handlers, where Nuxt requires `export default`
- NEVER manually import auto-imported APIs (`ref`, `computed`, `useRoute`, components, composables, utils). Nuxt handles it

## Architecture

Data flows one way: `server/api` -> `app/queries` -> `app/pages` -> `app/components`.

- Pages compose, they do not implement: data comes from queries, markup from components. NO business logic in pages
- Components receive data via props and communicate up via emits. Reach for a store ONLY when state is truly global (session, theme, cart)
- Extract logic into `app/composables/` when it is stateful and reused, into `app/utils/` when it is a pure function, into `shared/utils/` when the server needs it too
- NEVER pass props through more than 2 component levels: use typed `provide`/`inject` (with an `InjectionKey`) or restructure with slots
- NEVER import between feature component folders (`components/home/` must not import from `components/checkout/`). Shared pieces move to `components/ui/` or `components/layout/`
- The server is the trust boundary: every input is validated in `server/api` handlers, and responses return only the fields the client needs, NEVER raw DB rows

## Nuxt Rules

- ALWAYS use `<NuxtLink>` for internal links, NEVER `<a>`. Use `navigateTo()` for programmatic navigation, NEVER `window.location`
- ALWAYS use typed route names for navigation: `navigateTo({ name: '/users/[userId]', params: { userId } })` rather than string paths. Read the `pages` skill when working on routes
- SSR safety: NEVER access `window`/`document` during setup. Guard client-only code with `onMounted()` or `import.meta.client`, use `useRequestURL()` instead of `window.location`
- NEVER declare reactive state at module scope: it leaks between requests on the server. Shared state belongs in Pinia stores
- Wrap client-only UI in `<ClientOnly>` with a `#fallback` slot
- ALWAYS set page title and meta with `useSeoMeta()` (use `useHead()` for non-SEO tags like scripts)
- Errors: throw `createError({ statusCode, statusMessage })`. Full-screen errors render `app/error.vue`
- Config: secrets go in `runtimeConfig` (overridden via `NUXT_*` env vars), client-visible values in `runtimeConfig.public`, non-sensitive reactive app settings in `app.config.ts`. NEVER read `process.env` in runtime code
- Route middleware is ONLY for navigation guards (auth, redirects): `app/middleware/auth.ts` applied via `definePageMeta({ middleware: ['auth'] })`, or `*.global.ts` to run on every navigation

## Data Fetching

- ALWAYS fetch API data through Pinia Colada queries in `app/queries/`, mutations in `app/mutations/`. Read the `pinia-colada` skill before writing any
- Query and mutation functions MUST use `$fetch('/api/...')`: it runs on server and client, and `@pinia/colada-nuxt` transfers SSR state automatically
- NEVER call bare `$fetch` in component setup to load data (it fetches twice on SSR) and NEVER fetch data in Pinia stores
- Direct `$fetch` calls are ONLY for imperative actions inside mutations or event handlers
- Reserve `useFetch()`/`useAsyncData()` for the rare cases Pinia Colada does not cover
- In `defineQuery()` files, import `useRoute` from `vue-router` explicitly: the Nuxt auto-imported `useRoute` MUST NOT be used inside `defineQuery`

## Forms & Validation

- ONE zod schema per form or entity, defined in `shared/schemas/`, validates BOTH sides: the client (`schema.safeParse` on submit) and the server (`readValidatedBody(event, schema.parse)`)
- Derive types from schemas: `type CreateUser = z.infer<typeof CreateUserSchema>`. NEVER write an interface by hand next to a schema
- Show field errors from the `safeParse` result and disable submit while the mutation is loading (`asyncStatus === 'loading'`)

## Project Structure

Keep this section up to date with the project structure. Use it as a reference to find files and directories.

EXAMPLES are there to illustrate the structure, not to be implemented as-is.

```
app/                     # Vue application (Nuxt srcDir, `~`/`@` alias)
├── assets/              # Assets processed by the build (CSS, fonts, icons)
├── components/          # Auto-imported components, named by their path
│   ├── ui/              # Base UI components: ui/UiButton.vue -> <UiButton>
│   ├── layout/          # App chrome: layout/LayoutHeader.vue -> <LayoutHeader>
│   └── home/            # EXAMPLE feature folder: home/HomeHero.vue -> <HomeHero>
├── composables/         # Auto-imported composables (top-level files only)
├── layouts/             # default.vue + named layouts, applied via definePageMeta
├── middleware/          # Route middleware (navigation guards)
├── pages/               # File-based routes, see the `pages` skill
│   ├── (home).vue       # EXAMPLE index page using a group for a better name, renders at /
│   ├── users.vue        # EXAMPLE layout for nested routes, must contain <NuxtPage />
│   └── users/
│       └── [userId].vue # EXAMPLE that renders at /users/:userId
├── plugins/             # Nuxt plugins, auto-registered
├── queries/             # Pinia Colada queries: key factories + defineQueryOptions
│   └── users.ts         # EXAMPLE file for user-related queries
├── mutations/           # Pinia Colada mutations: defineMutation
├── stores/              # Pinia stores for global client state (NOT data fetching)
├── utils/               # Auto-imported pure utility functions
├── app.config.ts        # Reactive non-sensitive app settings
├── app.vue              # Root component
└── error.vue            # Full-screen error page
server/
├── api/                 # API endpoints: users.get.ts -> GET /api/users
├── middleware/          # Server middleware, runs on EVERY request
├── routes/              # Server routes outside /api: healthz.get.ts -> GET /healthz
└── utils/               # Auto-imported server utilities (DB, external APIs)
shared/
├── schemas/             # Zod schemas shared by app forms and server validation
├── types/               # Types shared between app/ and server/
└── utils/               # Pure functions shared between app/ and server/
public/                  # Static files served as-is (favicon, robots.txt)
nuxt.config.ts           # Modules, runtimeConfig, routeRules
```

## Project Commands

Frequently used commands:

- `pnpm run build`: bundles the project for production
- `pnpm run typecheck`: type checks the project with vue-tsc
- `pnpm run test`: runs all tests
- `pnpm vitest run <test-files>`: runs one or multiple specific test files
  - add `--coverage` to check missing test coverage

## Development Workflow

ALWAYS follow the workflow when implementing a new feature or fixing a bug. This ensures consistency, quality, and maintainability of the codebase.

1. Plan your tasks, review them with user. Include tests when possible
2. Write code, following the [project structure](#project-structure) and [conventions](#standards)
3. **ALWAYS test implementations work**:
   - Write tests for logic and components
   - Use the agent-browser to test like a real user
4. Stage your changes with `git add` once a feature works
5. Review changes and analyze the need of refactoring

## Testing Workflow

### Unit and Integration Tests

- Test critical logic first
- Split the code if needed to make it testable
- Server routes: test validation and error paths, not just the happy path

### Browser Testing

1. Navigate to the relevant page
2. Wait for content to load completely
3. Test primary user interactions
4. Test secondary functionality (error states, edge cases)
5. Check the JS console for errors or warnings
   - If you see errors, investigate and fix them immediately
   - If you see warnings, document them and consider fixing if they affect user experience
6. Check the terminal output of the dev server for SSR errors and hydration mismatch warnings
7. Document any bugs found and fix them immediately

## Research & Documentation

- **NEVER hallucinate or guess URLs**
- ALWAYS try accessing the `llms.txt` file first to find relevant documentation. Verified indexes:
  - <https://nuxt.com/llms.txt>
  - <https://vuejs.org/llms.txt>
  - <https://router.vuejs.org/llms.txt>
  - <https://pinia-colada.esm.dev/llms.txt>
  - <https://vueuse.org/llms.txt>
  - <https://nitro.build/llms.txt> (server engine)
- ALWAYS follow existing links in table of contents or documentation indices
- Verify examples and patterns from documentation before using
