# Frontend Architecture Playbook

Opinionated guide to scalable React architecture, folder strategy, state boundaries, testing, and DX.

> *If I were setting a React standard for a team today, I would not chase the most academic folder structure. I would optimize for discoverability, predictable boundaries, route-level performance, semantic UI, and tests that protect real product behavior.*

---

## Table of Contents

- [Frontend Architecture Playbook](#frontend-architecture-playbook)
  - [Table of Contents](#table-of-contents)
  - [Why this exists](#why-this-exists)
  - [Companion playbooks](#companion-playbooks)
  - [The defaults I'd reach for first](#the-defaults-id-reach-for-first)
  - [The two structures worth knowing](#the-two-structures-worth-knowing)
    - [1. Group by shared module type](#1-group-by-shared-module-type)
    - [2. Group by module, domain, or feature](#2-group-by-module-domain-or-feature)
  - [My recommended default: the hybrid model](#my-recommended-default-the-hybrid-model)
    - [Use feature folders for:](#use-feature-folders-for)
    - [Use shared folders for:](#use-shared-folders-for)
  - [Shared modules structure](#shared-modules-structure)
    - [Where it shines](#where-it-shines)
    - [Where it breaks](#where-it-breaks)
  - [Feature or domain structure](#feature-or-domain-structure)
    - [Why I like it](#why-i-like-it)
  - [Component and hook anatomy](#component-and-hook-anatomy)
    - [Good component folder shape](#good-component-folder-shape)
    - [Good hook folder shape](#good-hook-folder-shape)
    - [What belongs together](#what-belongs-together)
    - [What I would avoid](#what-i-would-avoid)
  - [Performance defaults](#performance-defaults)
    - [Practical performance habits](#practical-performance-habits)
  - [API contract, types, and errors](#api-contract-types-and-errors)
  - [Authentication and sessions (Node API)](#authentication-and-sessions-node-api)
  - [Testing defaults](#testing-defaults)
    - [Baseline rules I would publish](#baseline-rules-i-would-publish)
    - [Analytics deserves a special note](#analytics-deserves-a-special-note)
  - [Semantic HTML and accessibility habits](#semantic-html-and-accessibility-habits)
  - [Environment and secrets](#environment-and-secrets)
    - [Rules worth stating plainly](#rules-worth-stating-plainly)
  - [Suggested repository structure](#suggested-repository-structure)
  - [References and inspiration](#references-and-inspiration)
    - [Official and high-signal references](#official-and-high-signal-references)
    - [Strong GitHub examples and adjacent repositories](#strong-github-examples-and-adjacent-repositories)
  - [License](#license)

---

## Why this exists

React projects usually become messy in a very predictable way.

They begin tidy, then slowly turn into:

- one `components/` folder nobody wants to open;
- pages that know too much;
- utilities that quietly became infrastructure;
- hooks with half the app inside them;
- tests that lag behind the product;
- performance fixes added late, one route at a time.

The raw notes in this repository already point in the right direction. This README turns them into a clearer standard you can publish, share, and actually enforce.

---

## Companion playbooks

These repositories form one playbook suite:

- [Auth & Identity Playbook](https://github.com/khasky/auth-identity-playbook) — sessions, tokens, OAuth, and identity boundaries across the stack
- [Backend Architecture Playbook](https://github.com/khasky/backend-architecture-playbook) — APIs, boundaries, OpenAPI, persistence, and errors
- [Best of JavaScript](https://github.com/khasky/best-of-javascript) — curated JS/TS tooling and stack defaults
- [Caching Playbook](https://github.com/khasky/caching-playbook) — HTTP, CDN, and application caches; consistency and invalidation
- [Code Review Playbook](https://github.com/khasky/code-review-playbook) — PR quality, ownership, and review culture
- [DevOps Delivery Playbook](https://github.com/khasky/devops-delivery-playbook) — CI/CD, environments, rollout safety, and observability
- [Engineering Lead Playbook](https://github.com/khasky/engineering-lead-playbook) — standards, RFCs, and technical leadership habits
- **Frontend Architecture Playbook** — React structure, performance, and consuming API contracts
- [Marketing and SEO Playbook](https://github.com/khasky/marketing-and-seo-playbook) — growth, SEO, experimentation, and marketing surfaces
- [Monorepo Architecture Playbook](https://github.com/khasky/monorepo-architecture-playbook) — workspaces, package boundaries, and shared code at scale
- [Node.js Runtime & Performance Playbook](https://github.com/khasky/nodejs-runtime-performance-playbook) — event loop, streams, memory, and production Node performance
- [Testing Strategy Playbook](https://github.com/khasky/testing-strategy-playbook) — unit, integration, contract, E2E, and CI-friendly test layers

---

## The defaults I'd reach for first

If I were starting a modern React application today, I would usually default to this:

- **Structure:** feature-first for product code, shared folders for true cross-app primitives
- **Routing:** code-split at the route level by default
- **Components:** colocate component, test, story, styles, and re-export index when the component matters enough
- **Hooks:** keep reusable hooks small, named, and colocated with their tests if they contain real logic
- **State:** colocate feature state with the feature whenever possible; use **TanStack Query** for server state so caching, retries, and error boundaries stay consistent with your API
- **Types:** prefer **types generated from the same OpenAPI** the backend owns (`openapi-typescript`, Hey API, Orval — see [Best of JS](https://github.com/khasky/best-of-javascript)); avoid duplicating request/response shapes by hand
- **Errors:** parse API error bodies with the **same JSON fields** the backend documents (`code`, `message`, `request_id`); surface `request_id` in toasts or error UIs when users report bugs
- **Testing:** **Vitest** + **Testing Library** for unit and component tests; **Jest** only where legacy already dictates it
- **Analytics:** important instrumentation should be testable, not "fire and pray"
- **Markup:** prefer semantic HTML before div soup
- **Secrets:** `.env` files are for runtime wiring, not for committing secrets into Git history

---

## The two structures worth knowing

The source notes highlight two major ways to organize a React codebase.

### 1. Group by shared module type

This is the familiar style:

- `components/`
- `hooks/`
- `utils/`
- `config/`
- `pages/`
- `routes/`
- `store/`

It is easy to start with and easy for new developers to recognize.

### 2. Group by module, domain, or feature

This style pushes product code into feature areas like:

- `modules/dashboard`
- `modules/details`
- `modules/common`

It scales better when the application grows sideways and multiple product areas evolve independently.

Both structures are valid. The real question is not "which one is correct?" The question is "which one fails later?"

---

## My recommended default: the hybrid model

This is the model I would recommend to most teams.

### Use feature folders for:

- pages with real business behavior;
- feature-specific components;
- state slices;
- API adapters tied to a single product area;
- selectors, hooks, and tests that mostly serve one domain.

### Use shared folders for:

- design-system primitives;
- app shell concerns;
- routing bootstrap;
- global config;
- localization;
- truly reusable hooks and utilities;
- cross-feature types and constants.

That gives you a healthier balance:

- not every button becomes its own "domain";
- not every domain gets flattened into a giant global `components/` folder.

---

## Shared modules structure

This structure works well for smaller apps, design-system repos, or codebases that are still mostly UI composition.

```txt
src/
  assets/
  components/
    Button/
      Button.tsx
      Button.test.tsx
      Button.styles.ts
      Button.stories.tsx
      index.ts
  pages/
  routes/
  store/
    rootReducer.ts
  hooks/
    useSomeStuff/
      useSomeStuff.ts
      useSomeStuff.test.ts
      index.ts
  utils/
    validation/
  config/
  locales/
  types/
  constants/
  App.tsx
  main.tsx
```

### Where it shines

- clear at the beginning;
- low conceptual overhead;
- easy for shared UI work.

### Where it breaks

- feature logic gets scattered across many top-level folders;
- product changes require editing files in five places;
- ownership becomes fuzzy.

---

## Feature or domain structure

This is usually the stronger long-term choice for application code.

```txt
src/
  app/
    router/
    providers/
    store/
  modules/
    common/
      components/
        Button/
        Input/
    dashboard/
      api/
      components/
        Table/
        Sidebar/
      hooks/
      routes/
      state/
      tests/
    details/
      api/
      components/
      hooks/
      routes/
      state/
      tests/
  shared/
    assets/
    config/
    hooks/
    lib/
    locales/
    types/
  main.tsx
```

### Why I like it

- feature code stays together;
- refactors become local instead of repo-wide;
- testing becomes more natural;
- onboarding improves because the code mirrors the product.

This direction is also aligned with public guidance around feature grouping and colocating logic near the code that owns it.

---

## Component and hook anatomy

The source notes already contain a useful instinct here: when a component or hook is important enough, give it a little home.

### Good component folder shape

```txt
Button/
  Button.tsx
  Button.test.tsx
  Button.styles.ts
  Button.stories.tsx
  index.ts
```

### Good hook folder shape

```txt
useSomeStuff/
  useSomeStuff.ts
  useSomeStuff.test.ts
  index.ts
```

### What belongs together

- implementation;
- tests;
- storybook story if it is part of a reusable UI layer;
- styles if the styling approach benefits from colocation;
- `index.ts` re-export to keep imports clean.

### What I would avoid

- component folders for trivial one-file throwaways;
- giant barrel exports that hide ownership;
- generic hooks that quietly depend on half the app.

---

## Performance defaults

One of the smartest notes in the source material is also one of the simplest:

> code-split pages at the router level to improve load time

That should be the default in most product apps.

### Practical performance habits

- lazy-load route chunks;
- keep dashboard-sized dependencies out of the initial bundle if the landing page does not need them;
- avoid dragging feature-only code into shared modules;
- profile before "optimizing" every component manually.

Route-level code splitting gives you one of the highest signal-to-effort wins in React architecture.

---

## API contract, types, and errors

The [backend playbook](https://github.com/khasky/backend-architecture-playbook) treats **OpenAPI** as the contract. The React app should consume that contract instead of guessing.

- **Codegen:** generate TypeScript types (and optionally clients) from the shared spec; wire outputs through a workspace package in a monorepo or a published `@your-scope/api-types` package.
- **Data fetching:** implement feature hooks with **TanStack Query** (`useQuery` / `useMutation`) so loading and error states stay uniform; map **HTTP status** and **parsed error JSON** in one place (for example a shared `parseApiError` helper used by `QueryClient`'s global handlers).
- **Stability:** agree with the API on validation status codes (**400** vs **422**) and on the **error envelope**; UI code should not special-case different shapes per endpoint.
- **Supportability:** when the API returns **`request_id`**, show it (or copy it) on error screens so logs and user reports match.

---

## Authentication and sessions (Node API)

Browser auth should match how your **Express / Fastify / Hono** service issues and checks credentials (see the backend playbook's pipeline section).

- **Cookies vs bearer tokens:** if the API uses **httpOnly** cookies, prefer **same-site** patterns and a **BFF** or first-party proxy when the SPA and API are on different origins; if you use **Authorization: Bearer**, never store tokens in places that sync to Git or public bundles — treat refresh flows as part of the architecture, not a one-off fetch.
- **CORS:** configure allowed origins and credentials explicitly; "allow everything" is a common source of subtle production bugs.
- **Alignment:** the same OpenAPI document should describe **security schemes** (cookie, bearer, OAuth2) so generated clients and manual `fetch` wrappers stay honest.

Secrets for third-party auth (OAuth client secrets, API keys) stay in **vaults / CI / platform config**, not in committed `.env` files — the [environment section](#environment-and-secrets) below still applies.

---

## Testing defaults

The README should make testing expectations obvious, not optional folklore.

### Baseline rules I would publish

- all new or modified logic should have unit test coverage using **Vitest** and **Testing Library** as the default stack for new work;
- **Jest** is acceptable when the repo already standardizes on it; do not mix two runners in the same package without a migration plan;
- important custom hooks should be tested if they contain branching or side effects;
- analytics events should be testable, ideally with explicit spies or mocks;
- unit tests should run on feature branches and again when merged to the main branch (see the [DevOps playbook](https://github.com/khasky/devops-delivery-playbook) for lane layout).

### Analytics deserves a special note

Instrumentation code is easy to ignore because it rarely blocks local development. That is exactly why it drifts.

If analytics matters to the business, test it like product behavior.

Example idea:

```ts
import { vi } from "vitest"

vi.spyOn(analytics, "track")
```

Then assert the event payload from the actual interaction path, not from a detached helper test.

---

## Semantic HTML and accessibility habits

Prefer semantic elements when they match the job.

- `<header>`
- `<nav>`
- `<main>`
- `<section>`
- `<article>`
- `<aside>`
- `<footer>`
- `<figure>`
- `<figcaption>`
- `<ul><li>`
- `<time datetime="...">`

That does not make the app accessible by magic, but it gives the UI better structure for users, assistive tech, maintainers, and search engines.

A React codebase with strong architecture and weak semantics is still incomplete.

---

## Environment and secrets

The `.env` note in the source material is exactly the right instinct.

### Rules worth stating plainly

- environment files are configuration, not a secret-management strategy;
- sensitive values should come from vaults, platform secrets, or CI-managed injection;
- `.env` should not become a graveyard of production secrets committed by accident.

If a repo needs runtime configuration, document the expected variables. If it needs secrets, use proper secret management.

---

## Suggested repository structure

This is the version I would use for a serious app:

```txt
src/
  app/
    providers/
    router/
    store/
  modules/
    auth/
    dashboard/
    details/
  shared/
    assets/
    components/
    config/
    hooks/
    lib/
    locales/
    types/
  tests/
  main.tsx
```

And inside a real feature:

```txt
modules/
  dashboard/
    api/
    components/
      DashboardTable/
      DashboardSidebar/
    hooks/
    routes/
    state/
    tests/
    index.ts
```

Simple enough to scan. Structured enough to scale.

---

## References and inspiration

### Official and high-signal references

- [React: File Structure FAQ](https://legacy.reactjs.org/docs/faq-structure.html)
- [Redux: Code Structure FAQ](https://redux.js.org/faq/code-structure)

### Strong GitHub examples and adjacent repositories

- [bulletproof-react: project structure](https://github.com/alan2207/bulletproof-react/blob/master/docs/project-structure.md)
- [Awesome React](https://github.com/enaqx/awesome-react)
- [React + TypeScript clean architecture boilerplate](https://github.com/TECHS-Technological-Solutions/react-typescript-clean-architecture)

---

## License

MIT is a sensible default for a guide repository like this, but choose the license that matches how open you want reuse to be.
