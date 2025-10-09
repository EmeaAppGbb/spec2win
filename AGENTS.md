# AGENTS.md

## Mission

Build agents and applications that turn specifications into production-ready experiences with high reliability, speed, and delight. Every implementation decision must optimize for developer ergonomics, observable quality, and rapid iteration.

## Guiding Principles

- **Specification driven:** Treat written product specs, OpenAPI contracts, JSON Schemas, and design tokens as the source of truth. Automate code generation and validation wherever possible.
- **Type-safe end to end:** Deliver full TypeScript coverage on the frontend and C# with nullable reference types, record types, and strict type checking on the backend. Fail builds on type regressions and nullable warnings.
- **Quality gates first:** Unit, integration, contract, and e2e tests run on every PR. Blocks merge when coverage or quality thresholds dip below agreed budgets.
- **Observability by default:** Emit structured logs, traces, and metrics for every service. Expose key health indicators through standardized dashboards and alerting.
- **Security & compliance everywhere:** Least privilege access, secret scanning, dependency hygiene, and threat modeling are non-negotiable steps in planning and delivery.
- **Always use latest packages:** Always fetch and use the latest stable versions of NuGet packages and npm dependencies unless there's a specific compatibility constraint. Update dependencies regularly.
- **Modular, self-contained tasks:** Design each feature task to be independently implementable with minimal cross-dependencies. Modules and classes should be self-contained and loosely coupled to enable safe refactoring and parallel development.
- **MCP tool usage for learning**: When researching services, frameworks, or libraries during development, always query Microsoft Docs MCP (`microsoft.docs.mcp`) first for official Microsoft documentation. Only use Context7 MCP if information cannot be found in Microsoft Docs.

## Canonical Stack

| Layer | Default | Notes |
| --- | --- | --- |
| Frontend | **Next.js 14 (App Router) + React 18 + TypeScript** | Embrace Server Components + Client Components split. Support alternative adapters (e.g., Nuxt/Vue) only with explicit approval. |
| Backend | **.NET 9 + ASP.NET Core (Minimal APIs)** | Cloud-native, high-performance APIs with built-in DI, middleware pipeline, and async-first patterns. Strong typing via record types and nullable reference types. |
| Orchestration | **.NET Aspire** | Local development orchestrator for distributed apps with service discovery, telemetry, and resource management. Not for production—deploy to Azure Container Apps or Kubernetes. |
| Agent Runtime | **Microsoft Agent Framework + Azure AI Foundry** | Central orchestration for LLM/agent workflows, prompt assets, safety routing, and evaluation. Use **Microsoft.Agents.AI** and **Microsoft.Extensions.AI** NuGet packages. |
| Realtime | **WebSocket + SignalR** | Native ASP.NET Core SignalR for real-time communication with automatic reconnection, backpressure handling, and protocol negotiation (WebSockets, Server-Sent Events, Long Polling). |
| Package Manager | **pnpm (workspace mode)** for frontend, **NuGet** for .NET | Deterministic installs for Node.js packages. NuGet package management with central package versioning for .NET dependencies. |
| Monorepo Tooling | **Nx** (preferred) or **Turborepo** | Enforce consistent build graphs, caching, and affected-only CI pipelines. |
| Testing | **Frontend:** Vitest + React Testing Library + Playwright. **Backend:** xUnit (preferred), NUnit, or MSTest + FluentAssertions. | Keep coverage ≥ 85%. Contract tests generated from OpenAPI schema. Integration tests with WebApplicationFactory. |
| Linting & Formatting | **Frontend:** ESLint (Flat config) + Prettier + Stylelint. **Backend:** Roslyn analyzers + EditorConfig + CSharpier or dotnet format | Automated via pre-commit hooks and IDE integration. |
| CI/CD | GitHub Actions with reusable workflows | Stages: lint → test → build artifacts → security scan → deploy. Use `dotnet` CLI for builds and tests. |
| Environment Secrets | Azure Key Vault or GitHub OIDC → Cloud secret store | `.env` only for local dev; never commit. Use User Secrets for local .NET development. |

## Architecture Blueprint

1. **Monorepo layout (.NET Aspire structure)**
    - `{appname}.AppHost`: .NET Aspire orchestration project defining the app model and service dependencies for local development.
    - `{appname}.Api`: ASP.NET Core Minimal API service hosting REST, WebSocket, SignalR, and agent endpoints.
    - `{appname}.Web`: Frontend web app (Next.js, Blazor, or other) consuming generated APIs and component libraries.
    - `{appname}.ServiceDefaults`: Shared .NET Aspire project providing service defaults (telemetry, service discovery, resilience) reused across all .NET services.
    - `{appname}.AgentHost`: Microsoft Agent Framework host for agent orchestration, skills, memory, and telemetry middleware.
    - `/infra`: IaC (Bicep/Terraform) and GitHub Actions workflows. Azure deployment manifests generated by Aspire.

2. **Domain boundaries**
    - Separate presentation, orchestration, and domain logic. No cross-layer imports without contracts.
    - Use CQRS-inspired patterns for read/write segregation when scaling event-driven workloads.
    - Follow Clean Architecture or Vertical Slice Architecture patterns within .NET projects.

3. **Data flow**
    - API contracts defined via ASP.NET Core Minimal API endpoints with record types → OpenAPI spec published via Swashbuckle/NSwag → clients generated into `/packages/sdk`.
    - Frontend consumes the SDK via React Query (TanStack Query) to ensure cache consistency and retries.
    - Realtime channels use SignalR hubs with automatic reconnection and backpressure handling.
    - Service-to-service communication uses Aspire service discovery with automatic endpoint resolution.

## .NET Aspire Orchestration Guidelines

.NET Aspire is the local development orchestrator for cloud-native distributed applications. It simplifies service dependencies, configuration, and observability during development.

1. **AppHost project structure**
    - Create an AppHost project using `dotnet new aspire-apphost`.
    - Define your app model in `Program.cs` using `DistributedApplication.CreateBuilder()`.
    - **All service hosts must be defined in the AppHost**: Add resources using `.AddProject<T>()`, `.AddContainer()`, `.AddRedis()`, `.AddAzureCosmosDB()`, `.AddAzureServiceBus()`, etc.
    - **Use Aspire client integrations in services**: Wire up Aspire clients (e.g., `builder.AddAzureCosmosClient()`, `builder.AddRedisClient()`) in consuming services to leverage Aspire's service discovery and configuration.
    - **Use local emulators for development**: Configure Aspire to use local emulators (Cosmos DB Emulator, Azurite, Redis locally) via `.RunAsEmulator()` or `.RunAsContainer()` for local development. Never require cloud resources for local dev.
    - For Node.js/Next.js frontends: Add `Aspire.Hosting.NodeJs` package and use `.AddNpmApp("webapp", "../{appname}.Web")` to integrate frontend apps.
    - Express dependencies with `.WithReference()` for automatic service discovery and connection string injection.
    - Use `.WaitFor()` to ensure startup ordering between services.

2. **ServiceDefaults project**
    - Create a shared ServiceDefaults project using `dotnet new aspire-servicedefaults`.
    - All .NET services should reference this project and call `builder.AddServiceDefaults()` in `Program.cs`.
    - ServiceDefaults configures: OpenTelemetry (logs, metrics, traces), service discovery, HTTP resilience (Polly), and health checks.
    - Customize telemetry, add global middleware, or configure default HTTP client behaviors here.

3. **Service discovery**
    - Services reference each other using logical names defined in the AppHost (e.g., `"apiservice"`).
    - Use `https+http://apiservice` as the base URL in HttpClient configurations—Aspire resolves to actual endpoints.
    - Environment variables are automatically injected: `services__apiservice__http__0`, `ConnectionStrings__cache`, etc.

4. **Deployment**
    - Aspire is for local development only. **Do not deploy the AppHost to production.**
    - Use `azd init` and `azd up` to deploy to Azure Container Apps with auto-generated infrastructure.
    - Run `aspire publish` to generate deployment manifests for custom targets (Kubernetes, Docker Compose).
    - For production, deploy services independently to Azure Container Apps, Azure App Service, or Kubernetes.

5. **Dashboard**
    - Aspire provides a built-in dashboard at runtime showing all resources, logs, traces, and metrics.
    - Access via the URL shown when running the AppHost project.
    - Use for debugging service communication, observing telemetry, and monitoring resource health.

## Agent-First Delivery

- Host agent brains and tools within Azure AI Foundry, storing prompt assets, evaluation suites, and safety rules under version control.
- Wrap each agent in the Microsoft Agent Framework lifecycle: planners, skills/tools, memory, and telemetry middleware.
- **AgentHost architecture**: Create a dedicated `{appname}.AgentHost` project for distributed agent execution, orchestration, and background processing separate from the API layer.
- **Agent providers pattern**: Implement agents as provider classes (e.g., `CopywritingAgentProvider`, `AuditAgentProvider`) that encapsulate agent creation, tool registration, and configuration.
- **Orchestration services**: Create orchestrator services that coordinate multiple agents with sequential workflows (dependent execution) and concurrent workflows (parallel execution).
- Define clear SLAs for latency, accuracy, and guardrail enforcement. Build regression suites that run in CI using offline evaluation datasets.
- Provide SDK shims so frontend clients can call agent endpoints asynchronously (REST, streaming, or SignalR) with graceful fallback modes.
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

## Backend Playbook (ASP.NET Core + Aspire)

1. **ASP.NET Core project layout**
    - `Program.cs` for application startup, middleware pipeline, and Minimal API endpoint definitions.
    - `Endpoints/` for endpoint groups and route handlers (Minimal APIs pattern).
    - `Services/` for domain logic, orchestrating agents and external services (registered via DI).
    - `Models/` for DTOs, domain entities, and data models (use records for immutability).
    - `Data/` for Cosmos DB repository implementations and database client configurations.
        - **Prefer Cosmos DB for data persistence**: Use Azure Cosmos DB as the primary database for all data storage needs.
        - **Do NOT use Entity Framework**: Use the native Azure Cosmos DB SDK (`Microsoft.Azure.Cosmos`) directly for all database operations. Avoid EF Core or other ORMs.
        - Implement repository pattern with async methods for CRUD operations.
        - **Auto-initialize database in development**: Create containers and databases automatically on startup in development mode using initialization service.

2. **Aspire integration**
    - Reference the `ServiceDefaults` project from all .NET services for shared telemetry, service discovery, and resilience configuration.
    - Call `builder.AddServiceDefaults()` early in `Program.cs` to enable Aspire integrations.
    - Use `builder.AddAzureCosmosClient()`, `builder.AddRedisClient()`, `builder.AddAzureServiceBusClient()`, etc., from Aspire hosting integrations.
    - Define dependencies in `AppHost/Program.cs` using `WithReference()` for automatic service discovery and connection string injection.

3. **Async excellence & best practices**
    - All I/O operations must be async (`async`/`await`). Never block with `.Result` or `.Wait()`.
    - Use ASP.NET Core's built-in dependency injection with scoped, singleton, and transient lifetimes.
    - Minimize middleware complexity. Keep hot code paths fast and avoid long-running tasks in the request pipeline.
    - Use `IAsyncEnumerable<T>` for streaming responses and pagination to avoid loading large datasets into memory.

4. **Realtime & media**
    - SignalR hubs for real-time bidirectional communication (WebSockets with fallback). Define strongly-typed hub contracts.
    - Ensure proper backpressure handling in SignalR streaming methods using `IAsyncEnumerable<T>` or `ChannelReader<T>`.
    - For WebRTC signaling, use SignalR or dedicated WebSocket endpoints. Offload TURN/STUN to Azure Communication Services.
    - Configure CORS policies for SignalR cross-origin connections.

5. **Agents integration**
    - **Always use AI Toolkit best practices**: Before implementing agent code, use `#aitk-get_agent_code_gen_best_practices` to retrieve the latest best practices and guidance for Microsoft Agent Framework code generation.
    - **Required NuGet packages**: Install `Microsoft.Agents.AI`, `Microsoft.Agents.AI.OpenAI`, `Microsoft.Extensions.AI`, and `Microsoft.Extensions.AI.OpenAI` packages.
    - **Use `ChatClientAgent` for all agents**: Wrap all agents using `ChatClientAgent` from `Microsoft.Agents.AI` with proper agent descriptions, names, and instructions.
    - **Agent construction pattern**: Create agents with `new ChatClientAgent(chatClient, instructions: "...", name: "AgentName")` in service constructors.
    - **Use `AIAgent.RunAsync()` pattern**: Call agents using `await _agent.RunAsync(prompt)` instead of direct `IChatClient` calls. Never use `IChatClient` directly for agent operations.
    - **Configuration chain**: `Azure OpenAI Client → ChatClient → .AsIChatClient() → ChatClientAgent → AIAgent` is the proper initialization chain.
    - **Implement tools using `AIFunctionFactory`**: Define agent capabilities as tools using `AIFunctionFactory.Create()` with clear descriptions and parameter documentation.
    - **Tool registration pattern**: Register tools in agent constructor using `tools: [AIFunctionFactory.Create(Method1), AIFunctionFactory.Create(Method2)]` parameter.
    - **Equip agents with domain-specific tools**: Each agent should have 3+ relevant tools that enable specific capabilities (e.g., validation, optimization, analysis).
    - **Use `IChatClient` abstraction**: All agent services should depend on `IChatClient` in constructors for flexibility and testability, but wrap it in `ChatClientAgent`.
    - **Implement orchestration patterns**: Support both sequential (one-after-another) and concurrent (parallel) agent execution patterns via orchestrator services. Use pattern matching for agent routing.
    - **Retry logic with compliance feedback**: Implement retry loops (5+ attempts) with exponential backoff. Pass rejection feedback from compliance agents to the next retry attempt.
    - **Campaign brief construction**: Build comprehensive context strings that include theme, product details, target audience, and optional revision feedback from previous iterations.
    - **Prefer AI-powered over rule-based logic**: Use LLM intelligence with tools for decision-making rather than simple string matching or rule engines.
    - **DI registration**: Register `IChatClient` as singleton via `.AsIChatClient()`. Register agent services as scoped for per-request isolation.
    - Implement Microsoft Agent Framework skills as injectable services (singleton or scoped).
    - Use `IHostedService` or `BackgroundService` for long-running agent workers and background processing.
    - Persist agent memory/state in Cosmos DB using the native Azure Cosmos DB SDK with encryption at rest.

6. **Observability & resilience**
    - OpenTelemetry is configured automatically via Aspire `ServiceDefaults` (logs, metrics, traces exported via OTLP).
    - Use ASP.NET Core health checks (`MapHealthChecks()`) and expose via Aspire dashboard.
    - Implement resilience with Polly (circuit breakers, retries, timeouts) via `Microsoft.Extensions.Http.Resilience`.
    - Return Problem Details (RFC 7807) for errors using `Results.Problem()` in Minimal APIs.
    - Use structured logging with `ILogger<T>` and log scopes for correlation.

7. **Testing**
    - xUnit as preferred test framework. NUnit and MSTest are acceptable alternatives.
    - Use `WebApplicationFactory<TProgram>` for integration tests, testing full HTTP pipelines.
    - FluentAssertions for readable test assertions.
    - Cover: unit tests (services, domain logic), integration tests (API endpoints, database), contract tests (OpenAPI validation).
    - Use Testcontainers for integration tests requiring databases or message brokers.
    - Load testing with k6, JMeter, or NBomber for high-throughput endpoints and SignalR hubs.

## Shared Engineering Systems

- **Code quality**: Pre-commit hooks running ESLint, Prettier, Stylelint (frontend), dotnet format/CSharpier, Roslyn analyzers (backend), and commit message lint (Conventional Commits).
- **Documentation**: MkDocs site generated from `/docs` with Material theme. Keep ADRs (Architecture Decision Records) under `/specs/adr` with numbered titles.
- **Secrets**: Use .NET User Secrets (`dotnet user-secrets`) for local development, Azure Key Vault for cloud environments. No plain-text secrets in `.env.sample` or `appsettings.json` beyond placeholders.
- **Version control**: Comprehensive `.gitignore` must cover .NET artifacts (`bin/`, `obj/`, `*.user`), Node.js dependencies (`node_modules/`, `.next/`), environment files (`.env.local`), IDE files (`.vs/`, `.vscode/`, `.idea/`), and secrets (`*.key`, `*.pfx`, `secrets.json`). Never commit build outputs, dependencies, or sensitive data.
- **Dependency governance**: Renovate bot with grouped update strategy. Central Package Management for .NET dependencies via `Directory.Packages.props`. Weekly security scan using GitHub Dependabot + Snyk.
- **Release management**: Semantic-release (apps/web) and MinVer or GitVersion for .NET projects. Tag releases with changelog generated from Conventional Commits.

## CI/CD Pipeline Expectations

1. **Workflow stages**
    - `lint` → `type-check` → `restore` → `build` → `unit-tests` → `integration-tests` → `security-scan` → `publish` → `deploy`.
    - Use `dotnet restore`, `dotnet build`, `dotnet test`, and `dotnet publish` for .NET projects.
    - Reuse workflow templates from `/infra/github` and rely on Nx/Turborepo caching for frontend, .NET build caching for backend.
    - Aspire projects: Build the AppHost project which orchestrates all dependencies.

2. **Preview environments**
    - Frontend: Vercel preview or Azure Static Web Apps staging slot per PR.
    - Backend: Deploy ephemeral .NET Aspire apps to Azure Container Apps using `azd provision` and `azd deploy` per PR.
    - Use Aspire's deployment manifest (`aspire publish`) to generate Azure infrastructure as code.

3. **Deployment**
    - **Azure Container Apps** (recommended): Deploy .NET Aspire projects using Azure Developer CLI (`azd`) with automatic manifest generation.
    - **Azure App Service**: Deploy individual .NET projects as containerized Linux Web Apps (preview in Aspire 9.3+).
    - **Kubernetes**: Use Aspire manifest generation with custom publishing extensions for Kubernetes YAML.
    - Blue/green or canary deployments with automated health checks (ASP.NET Core health checks endpoint) and rollback automation.
    - Capture SBOM (Software Bill of Materials) for each release using Syft or `dotnet sbom-tool`.

## Security & Compliance

- Adopt Zero Trust: enforce MFA, least privilege, and just-in-time access for all cloud resources.
- Perform secure code scans (CodeQL, Trivy, Bandit) each PR.
- Threat models accompany new features with STRIDE checklist.
- Data classification drives storage, encryption, and retention policies. PII must remain within approved regions.
- Handle WebRTC media via SRTP with DTLS; sanitize TURN credentials and rotate frequently.

## Documentation & Knowledge Sharing

- Keep living product specs in `/specs`. Synchronize with design docs (Figma) and API definitions.
- **Update documentation in place**: Do NOT create separate summary documents. Always update existing documentation files in `/docs` directly with new information, decisions, and learnings.
- **Create MADRs for task implementations**: When implementing a task, create a Markdown Architectural Decision Record (MADR) in `/specs/adr` documenting the architectural decisions, alternatives considered, and rationale. Use sequential numbering (e.g., `0001-use-cosmos-db.md`).
- Provide runbooks for on-call, including agent triage, incident response, and WebRTC signaling debugging.
- Share learning through weekly engineering journal entries collected in `/specs/journal`.

## Quality Gates Checklist

- ✅ Linters (ESLint, Stylelint, Prettier for frontend; dotnet format/CSharpier for backend)
- ✅ Type checks (tsc --noEmit for TypeScript; C# compiler with nullable reference types enabled)
- ✅ Static analysis (Roslyn analyzers, StyleCop, SonarAnalyzer for .NET)
- ✅ Tests (unit, integration, e2e, contract)
- ✅ Security scans (CodeQL, dependency review, .NET security analyzers)
- ✅ Observability smoke tests (health endpoints, telemetry assertions via Aspire dashboard)
- ✅ Documentation updates (README, changelog, ADRs, XML documentation comments in C#)
- ✅ MADR created for significant architectural or implementation decisions
- ✅ Latest stable package versions used (unless specific version constraints exist)

## Change Management

- Work on feature branches. PRs must reference Jira/Azure Boards tickets and include test evidence.
- Require two approvals for code touching security, auth, or agent orchestration logic.
- Keep PRs < 500 LOC; split larger features into incremental, reviewable units.
- Merge only when CI is green and quality gates are satisfied. Use squash merges by default.

---