# Frontend Guidelines

## Guiding Principles

- **Type-safe TypeScript**: Deliver full TypeScript coverage across the frontend. Fail builds on type regressions.
- **Always use latest npm packages**: Always fetch and use the latest stable versions of npm dependencies unless there's a specific compatibility constraint. Update dependencies regularly.
- **Specification driven**: Consume OpenAPI/JSON Schema to auto-generate types and forms.

## Canonical Stack

| Layer | Technology | Notes |
| --- | --- | --- |
| **Framework** | Next.js 14 (App Router) + React 18 + TypeScript | Embrace Server Components + Client Components split. Support alternative adapters (e.g., Nuxt/Vue) only with explicit approval. |
| **State Management** | TanStack Query for remote data, Zustand or Jotai for local state | Cache consistency and optimistic updates with TanStack Query. |
| **Testing** | Vitest + React Testing Library + Playwright | Keep coverage ≥ 85%. E2E tests for critical flows. |
| **Linting & Formatting** | ESLint (Flat config) + Prettier + Stylelint | Automated via pre-commit hooks and IDE integration. |
| **Package Manager** | pnpm (workspace mode) | Deterministic installs for Node.js packages. |
| **Monorepo Tooling** | Nx (preferred) or Turborepo | Enforce consistent build graphs, caching, and affected-only CI pipelines. |
| **Environment Secrets** | Azure Key Vault or GitHub OIDC → Cloud secret store | `.env` only for local dev; never commit. |

## Frontend Playbook

### 1. Scaffolding & Folders

- `app/` for routes
- `components/` for shared UI
- `features/<domain>/` for spec-driven modules
- `hooks/` for custom React hooks
- `lib/` for utility functions and shared logic
- `styles/` for global styles and themes
- Keep server actions isolated under `app/(server-actions)/`

### 2. Spec-to-UI Generation

- Consume OpenAPI/JSON Schema to auto-generate forms with libraries like React Hook Form + Zod
- Leverage design tokens (Style Dictionary) to ensure theme consistency
- Document UI variants in Storybook with MDX stories

### 3. State & Data Management

- Use React Server Components for data-fetch-heavy views
- Fall back to client components when interactivity is required
- Adopt TanStack Query for remote data caching and optimistic updates
- Use Zustand or Jotai for local ephemeral state
- Consume backend SDK via React Query to ensure cache consistency and retries

### 4. Performance & Accessibility

- **Performance Budgets:**
  - Largest Contentful Paint (LCP) < 2.5s
  - Cumulative Layout Shift (CLS) < 0.1
  - Enforce via Lighthouse CI
- **Accessibility:**
  - Mandate WCAG 2.2 AA compliance
  - Integrate axe-core tests in CI

### 5. Testing Strategy

- **Unit tests:** Vitest per component/hook
- **Interaction tests:** React Testing Library + user-event
- **Visual regression:** Storybook stories double as visual regression baselines (Chromatic or Loki)
- **E2E tests:** Playwright suite tied to critical flows:
  - Authentication
  - Agent interaction
  - Realtime session setup

### 6. Type Safety

- **Full TypeScript coverage**: Every file must have proper TypeScript types
- **Strict configuration**: Enable strict mode in `tsconfig.json`
  - `strict: true`
  - `noUncheckedIndexedAccess: true`
  - `noImplicitOverride: true`
- **Fail builds on type regressions**: Run `tsc --noEmit` in CI pipeline
- **Generated types**: Use generated types from OpenAPI schemas for API contracts
- **No `any` types**: Avoid using `any`; use `unknown` with proper type guards instead
- **Type-safe API clients**: Generate TypeScript SDK from OpenAPI spec into `/packages/sdk`

### 7. Code Quality

- **Pre-commit hooks** running:
  - **ESLint (Flat config)**: Use modern flat config format (`eslint.config.js`)
  - **Prettier**: Consistent code formatting across the codebase
  - **Stylelint**: CSS/SCSS linting for style files
  - **Commit message lint**: Enforce Conventional Commits format
- **Automated via IDE integration**: Configure VS Code, WebStorm, or other IDEs
- **CI enforcement**: Run all linters in CI pipeline before tests
- **TypeScript strict checks**: Enable `tsc --noEmit` in pre-commit and CI

### 8. Architecture Patterns

- Embrace Server Components + Client Components split
- Server Components for data fetching and SEO-critical content
- Client Components for interactive features and real-time updates
- Support alternative adapters (e.g., Nuxt/Vue) only with explicit approval

### 9. Realtime Communication

- Consume SignalR endpoints for real-time bidirectional communication
- Implement graceful degradation and reconnection logic
- Handle WebSocket fallbacks appropriately

### 10. Best Practices

- **Latest npm packages**: Always use latest stable npm packages unless there's a specific compatibility constraint
- **Package manager**: Use pnpm with workspace mode for deterministic installs
- **Dependency updates**: Update dependencies regularly via Renovate bot with grouped update strategy
- **Lock files**: Always commit `pnpm-lock.yaml` for reproducible builds
- Keep components modular and self-contained
- Follow specification-driven development
- Generate SDK clients from OpenAPI specs into `/packages/sdk`
- Use design tokens for consistent theming
