# AGENTS.md

## Mission

Build agents and applications that turn specifications into production-ready experiences with high reliability, speed, and delight. Every implementation decision must optimize for developer ergonomics, observable quality, and rapid iteration.

## Guiding Principles

- **Specification driven:** Treat written product specs, OpenAPI contracts, JSON Schemas, and design tokens as the source of truth. Automate code generation and validation wherever possible.
- **Type-safe end to end:** Deliver full TypeScript coverage on the frontend and Python type annotations (via `pydantic`, `typing`, and `mypy`) on the backend. Fail builds on type regressions.
- **Quality gates first:** Unit, integration, contract, and e2e tests run on every PR. Blocks merge when coverage or quality thresholds dip below agreed budgets.
- **Observability by default:** Emit structured logs, traces, and metrics for every service. Expose key health indicators through standardized dashboards and alerting.
- **Security & compliance everywhere:** Least privilege access, secret scanning, dependency hygiene, and threat modeling are non-negotiable steps in planning and delivery.

## Canonical Stack

| Layer | Default | Notes |
| --- | --- | --- |
| Frontend | **Next.js 14 (App Router) + React 18 + TypeScript** | Embrace Server Components + Client Components split. Support alternative adapters (e.g., Nuxt/Vue) only with explicit approval. |
| Backend | **FastAPI + Uvicorn (ASGI)** | Async-first Python services with native WebSocket support, background tasks, and strong typing via Pydantic v2. |
| Agent Runtime | **Microsoft Agent Framework + Azure AI Foundry** | Central orchestration for LLM/agent workflows, prompt assets, safety routing, and evaluation. |
| Realtime | **WebSocket + WebRTC (aiortc)** | Dedicated ASGI lifespan tasks for signaling, TURN/STUN integration, and media relays for avatar/speech use cases. |
| Package Manager | **pnpm (workspace mode)** | Deterministic installs, isolated builds, shared lockfile. Use Poetry for Python packaging inside the monorepo. |
| Monorepo Tooling | **Nx** (preferred) or **Turborepo** | Enforce consistent build graphs, caching, and affected-only CI pipelines. |
| Testing | **Frontend:** Vitest + React Testing Library + Playwright. **Backend:** Pytest + httpx/pytest-asyncio. | Keep coverage ≥ 85%. Contract tests generated from OpenAPI schema. |
| Linting & Formatting | ESLint (Flat config) + Prettier + Stylelint, Ruff + Black for Python | Automated via pre-commit hooks. |
| CI/CD | GitHub Actions with reusable workflows | Stages: lint → test → build artifacts → security scan → deploy. |
| Environment Secrets | Azure Key Vault or GitHub OIDC → Cloud secret store | `.env` only for local dev; never commit. |

## Architecture Blueprint

1. **Monorepo layout**
    - `/apps/web`: Next.js app consuming generated APIs and component libraries.
    - `/apps/api`: FastAPI service hosting REST, WebSocket, and agent endpoints.
    - `/apps/agents`: Agent orchestration workers leveraging Microsoft Agent Framework connectors.
    - `/packages/ui`: Reusable component library (Storybook + design tokens).
    - `/packages/sdk`: Auto-generated TypeScript + Python clients from OpenAPI / AsyncAPI specs.
    - `/infra`: IaC (Bicep/Terraform) and GitHub Actions workflows.

2. **Domain boundaries**
    - Separate presentation, orchestration, and domain logic. No cross-layer imports without contracts.
    - Use CQRS-inspired patterns for read/write segregation when scaling event-driven workloads.

3. **Data flow**
    - API contracts defined via FastAPI routers with Pydantic models → OpenAPI spec published → clients generated into `/packages/sdk`.
    - Frontend consumes the SDK via React Query (TanStack Query) to ensure cache consistency and retries.
    - Realtime channels follow a publish/subscribe model through Redis Streams or Azure Event Hubs.

## Agent-First Delivery

- Host agent brains and tools within Azure AI Foundry, storing prompt assets, evaluation suites, and safety rules under version control.
- Wrap each agent in the Microsoft Agent Framework lifecycle: planners, skills/tools, memory, and telemetry middleware.
- Define clear SLAs for latency, accuracy, and guardrail enforcement. Build regression suites that run in CI using offline evaluation datasets.
- Provide SDK shims so frontend clients can call agent endpoints asynchronously (REST, streaming, or WebRTC) with graceful fallback modes.
- Record conversations, decisions, and tool invocations for review—pipe to data lake storage with retention policies.

## Frontend Playbook

1. **Scaffolding & folders**
    - `app/` for routes, `components/` for shared UI, `features/<domain>/` for spec-driven modules, `hooks/`, `lib/`, and `styles/`.
    - Keep server actions isolated under `app/(server-actions)/`.

2. **Spec-to-UI generation**
    - Consume OpenAPI/JSON Schema to auto-generate forms with libraries like React Hook Form + Zod.
    - Leverage design tokens (Style Dictionary) to ensure theme consistency. Document UI variants in Storybook with MDX stories.

3. **State & data**
    - Use React Server Components for data-fetch-heavy views; fall back to client components when interactivity is required.
    - Adopt TanStack Query for remote data caching and optimistic updates. Use Zustand or Jotai for local ephemeral state.

4. **Performance & accessibility**
    - Budget Largest Contentful Paint (LCP) < 2.5s, CLS < 0.1. Enforce via Lighthouse CI.
    - Mandate WCAG 2.2 AA compliance. Integrate axe-core tests in CI.

5. **Testing**
    - Unit tests with Vitest per component/hook.
    - Interaction tests with React Testing Library + user-event.
    - Storybook stories double as visual regression baselines (Chromatic or Loki).
    - Playwright e2e suite tied to critical flows (auth, agent interaction, realtime session setup).

## Backend Playbook

1. **FastAPI project layout**
    - `app/api/routers` for REST/WebSocket routers.
    - `app/services` for domain logic, orchestrating agents and external services.
    - `app/models` for Pydantic schemas and SQLAlchemy models.
    - `app/core` for configuration, security, logging, and task scheduling.

2. **Async excellence**
    - Run under Uvicorn with `--http h11 --ws websockets`. Prefer `asyncio` tasks, `anyio`, and `asyncpg` for DB access.
    - Use dependency injection (`fastapi.Depends`) for composable services.

3. **Realtime & media**
    - WebSocket endpoints for streaming agent responses and telemetry. Ensure backpressure handling and heartbeat pings.
    - WebRTC signaling service using FastAPI routes + `aiortc` for media negotiation. Offload TURN to Azure Communication Services or coturn.

4. **Agents integration**
    - Implement adapters to Microsoft Agent Framework skills as FastAPI dependency providers.
    - Use background tasks (Celery, Dramatiq, or Azure Functions) for long-running tool executions.
    - Persist agent memory/state in Cosmos DB or PostgreSQL with encryption at rest.

5. **Observability & resilience**
    - Structured logging with `structlog`, tracing with OpenTelemetry (OTLP exporter), metrics via Prometheus + Grafana.
    - Circuit breakers and retries through `tenacity`. Global exception handlers returning Problem Details (`RFC 7807`).

6. **Testing**
    - Pytest as default. Cover: unit (`pytest -m unit`), integration (`pytest -m integration` hitting local services), contract (OpenAPI schema snapshots), and load tests (Locust or k6) for realtime endpoints.

## Shared Engineering Systems

- **Code quality**: Pre-commit hooks running ESLint, Prettier, Stylelint, Ruff, Black, and commit message lint (Conventional Commits).
- **Documentation**: Docusaurus site generated from `/docs`. Keep ADRs (Architecture Decision Records) under `/docs/adr` with numbered titles.
- **Secrets**: Use Doppler or Azure Key Vault for local injection; no plain-text secrets in `.env.sample` beyond placeholders.
- **Dependency governance**: Renovate bot with grouped update strategy. Weekly security scan using GitHub Dependabot + Snyk.
- **Release management**: Semantic-release (apps/web) and Poetry version bump (apps/api). Tag releases with changelog generated from Conventional Commits.

## CI/CD Pipeline Expectations

1. **Workflow stages**
    - `lint` → `type-check` → `unit-tests` → `integration-tests` → `build` → `security-scan` → `deploy`.
    - Reuse workflow templates from `/infra/github` and rely on Nx/Turborepo caching to minimize runtimes.

2. **Preview environments**
    - Frontend: Vercel preview or Azure Static Web Apps staging slot per PR.
    - Backend: Spin ephemeral environments via Azure Container Apps or Kubernetes namespaces.

3. **Deployment**
    - Blue/green or canary deployments with automated health checks and rollback automation.
    - Capture SBOM (Software Bill of Materials) for each release using Syft.

## Security & Compliance

- Adopt Zero Trust: enforce MFA, least privilege, and just-in-time access for all cloud resources.
- Perform secure code scans (CodeQL, Trivy, Bandit) each PR.
- Threat models accompany new features with STRIDE checklist.
- Data classification drives storage, encryption, and retention policies. PII must remain within approved regions.
- Handle WebRTC media via SRTP with DTLS; sanitize TURN credentials and rotate frequently.

## Documentation & Knowledge Sharing

- Keep living product specs in `/docs/specs`. Synchronize with design docs (Figma) and API definitions.
- Provide runbooks for on-call, including agent triage, incident response, and WebRTC signaling debugging.
- Share learning through weekly engineering journal entries collected in `/docs/journal`.

## Quality Gates Checklist

- ✅ Linters (ESLint, Stylelint, Prettier, Ruff, Black)
- ✅ Type checks (tsc --noEmit, mypy)
- ✅ Tests (unit, integration, e2e, contract)
- ✅ Security scans (CodeQL, dependency review)
- ✅ Observability smoke tests (health endpoints, telemetry assertions)
- ✅ Documentation updates (README, changelog, ADRs)

## Change Management

- Work on feature branches. PRs must reference Jira/Azure Boards tickets and include test evidence.
- Require two approvals for code touching security, auth, or agent orchestration logic.
- Keep PRs < 500 LOC; split larger features into incremental, reviewable units.
- Merge only when CI is green and quality gates are satisfied. Use squash merges by default.

---