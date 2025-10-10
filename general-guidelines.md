# General Guidelines

## Mission

Build agents and applications that turn specifications into production-ready experiences with high reliability, speed, and delight. Every implementation decision must optimize for developer ergonomics, observable quality, and rapid iteration.

## Guiding Principles

- **Specification driven:** Treat written product specs, OpenAPI contracts, JSON Schemas, and design tokens as the source of truth. Automate code generation and validation wherever possible.
- **Type-safe end to end:** Enforce strong typing across the entire stack. Fail builds on type regressions and violations of type safety rules.
- **Quality gates first:** Unit, integration, contract, and e2e tests run on every PR. Blocks merge when coverage or quality thresholds dip below agreed budgets.
- **Observability by default:** Emit structured logs, traces, and metrics for every service. Expose key health indicators through standardized dashboards and alerting.
- **Security & compliance everywhere:** Least privilege access, secret scanning, dependency hygiene, and threat modeling are non-negotiable steps in planning and delivery.
- **Always use latest packages:** Always fetch and use the latest stable versions of packages and dependencies unless there's a specific compatibility constraint. Update dependencies regularly.
- **Modular, self-contained tasks:** Design each feature task to be independently implementable with minimal cross-dependencies. Modules and classes should be self-contained and loosely coupled to enable safe refactoring and parallel development.
- **MCP tool usage for learning**: When researching services, frameworks, or libraries during development, always query Microsoft Docs MCP (`microsoft.docs.mcp`) first for official Microsoft documentation. Only use Context7 MCP if information cannot be found in Microsoft Docs.

## Architecture Blueprint

1. **Monorepo layout**
    - `{appname}.AppHost`: Orchestration project defining the app model and service dependencies for local development.
    - `{appname}.Api`: Backend API service hosting REST, WebSocket, and agent endpoints.
    - `{appname}.Web`: Frontend web app consuming generated APIs and component libraries.
    - `{appname}.ServiceDefaults`: Shared project providing service defaults (telemetry, service discovery, resilience) reused across all services.
    - `{appname}.AgentHost`: Agent framework host for agent orchestration, skills, memory, and telemetry middleware.
    - `/infra`: IaC (Bicep/Terraform) and CI/CD workflows. Deployment manifests for cloud platforms.

2. **Domain boundaries**
    - Separate presentation, orchestration, and domain logic. No cross-layer imports without contracts.
    - Use CQRS-inspired patterns for read/write segregation when scaling event-driven workloads.
    - Follow Clean Architecture or Vertical Slice Architecture patterns within projects.

3. **Data flow**
    - API contracts defined via backend endpoints → OpenAPI spec published → clients generated into `/packages/sdk`.
    - Frontend consumes the SDK via appropriate data fetching library to ensure cache consistency and retries.
    - Realtime channels use WebSocket/SignalR hubs with automatic reconnection and backpressure handling.
    - Service-to-service communication uses service discovery with automatic endpoint resolution.

## Agent-First Delivery

- Host agent brains and tools within Azure AI Foundry, storing prompt assets, evaluation suites, and safety rules under version control.
- Wrap each agent in the Microsoft Agent Framework lifecycle: planners, skills/tools, memory, and telemetry middleware.
- **AgentHost architecture**: Create a dedicated `{appname}.AgentHost` project for distributed agent execution, orchestration, and background processing separate from the API layer.
- **Agent providers pattern**: Implement agents as provider classes (e.g., `CopywritingAgentProvider`, `AuditAgentProvider`) that encapsulate agent creation, tool registration, and configuration.
- **Orchestration services**: Create orchestrator services that coordinate multiple agents with sequential workflows (dependent execution) and concurrent workflows (parallel execution).
- Define clear SLAs for latency, accuracy, and guardrail enforcement. Build regression suites that run in CI using offline evaluation datasets.
- Provide SDK shims so frontend clients can call agent endpoints asynchronously (REST, streaming, or SignalR) with graceful fallback modes.
- Record conversations, decisions, and tool invocations for review—pipe to data lake storage with retention policies.

## Shared Engineering Systems

- **Code quality**: Pre-commit hooks running linters, formatters, static analyzers, and commit message lint (Conventional Commits).
- **Documentation**: Documentation site generated from `/docs` with appropriate theme. Keep ADRs (Architecture Decision Records) under `/specs/adr` with numbered titles.
- **Secrets**: Use secure secret management for local development and cloud environments. No plain-text secrets in configuration files beyond placeholders.
- **Version control**: Comprehensive `.gitignore` must cover build artifacts, dependencies, environment files, IDE files, and secrets. Never commit build outputs, dependencies, or sensitive data.
- **Dependency governance**: Automated dependency updates with grouped update strategy. Centralized package management. Weekly security scan using GitHub Dependabot + Snyk.
- **Release management**: Semantic versioning and automated changelog generation from Conventional Commits.

## CI/CD Pipeline Expectations

1. **Workflow stages**
    - `lint` → `type-check` → `restore` → `build` → `unit-tests` → `integration-tests` → `security-scan` → `publish` → `deploy`.
    - Use appropriate CLI tools for restore, build, test, and publish operations.
    - Reuse workflow templates from `/infra/github` and rely on monorepo tooling for caching.
    - Orchestration projects: Build the AppHost project which orchestrates all dependencies.

2. **Preview environments**
    - Frontend: Preview deployments or staging slots per PR.
    - Backend: Deploy ephemeral backend services to cloud platform using appropriate deployment tools per PR.
    - Use deployment manifests to generate infrastructure as code.

3. **Deployment**
    - **Recommended**: Deploy to serverless/container platforms with automatic manifest generation.
    - **Alternatives**: Deploy to managed app services, serverless functions, or container orchestration platforms.
    - Blue/green or canary deployments with automated health checks and rollback automation.
    - Capture SBOM (Software Bill of Materials) for each release.

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

- ✅ Linters and formatters appropriate to the technology stack
- ✅ Type checks with strict type safety enforcement
- ✅ Static analysis and code quality tools
- ✅ Tests (unit, integration, e2e, contract)
- ✅ Security scans (CodeQL, dependency review, language-specific security analyzers)
- ✅ Observability smoke tests (health endpoints, telemetry assertions)
- ✅ Documentation updates (README, changelog, ADRs, code documentation)
- ✅ MADR created for significant architectural or implementation decisions
- ✅ Latest stable package versions used (unless specific version constraints exist)

## Change Management

- Work on feature branches. PRs must reference Jira/Azure Boards tickets and include test evidence.
- Require two approvals for code touching security, auth, or agent orchestration logic.
- Keep PRs < 500 LOC; split larger features into incremental, reviewable units.
- Merge only when CI is green and quality gates are satisfied. Use squash merges by default.
