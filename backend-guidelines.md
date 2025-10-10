# Backend Guidelines

## Guiding Principles

- **Type-safe C#**: Strong typing via record types and nullable reference types. Fail builds on nullable warnings and type violations.
- **Always use latest NuGet packages**: Always fetch and use the latest stable versions of NuGet packages unless there's a specific compatibility constraint. Update dependencies regularly.
- **Specification driven**: Define API contracts via Minimal API endpoints with record types → OpenAPI spec published via Swashbuckle/NSwag.

## Canonical Stack

| Layer | Technology | Notes |
| --- | --- | --- |
| **Framework** | .NET 9 + ASP.NET Core (Minimal APIs) | Cloud-native, high-performance APIs with built-in DI, middleware pipeline, and async-first patterns. Strong typing via record types and nullable reference types. |
| **Orchestration** | .NET Aspire | Local development orchestrator for distributed apps with service discovery, telemetry, and resource management. Not for production—deploy to Azure Container Apps or Kubernetes. |
| **Agent Runtime** | Microsoft Agent Framework + Azure AI Foundry | Central orchestration for LLM/agent workflows, prompt assets, safety routing, and evaluation. Use **Microsoft.Agents.AI** and **Microsoft.Extensions.AI** NuGet packages. |
| **Realtime** | WebSocket + SignalR | Native ASP.NET Core SignalR for real-time communication with automatic reconnection, backpressure handling, and protocol negotiation (WebSockets, Server-Sent Events, Long Polling). |
| **Testing** | xUnit (preferred), NUnit, or MSTest + FluentAssertions | Keep coverage ≥ 85%. Contract tests generated from OpenAPI schema. Integration tests with WebApplicationFactory. |
| **Linting & Formatting** | Roslyn analyzers + EditorConfig + CSharpier or dotnet format | Automated via pre-commit hooks and IDE integration. |
| **Package Manager** | NuGet with Central Package Management | NuGet package management with central package versioning for .NET dependencies via `Directory.Packages.props`. |
| **CI/CD** | GitHub Actions with reusable workflows | Stages: lint → test → build artifacts → security scan → deploy. Use `dotnet` CLI for builds and tests. |
| **Environment Secrets** | Azure Key Vault or GitHub OIDC → Cloud secret store | Use User Secrets for local .NET development. Never commit secrets. |

## Backend Playbook (ASP.NET Core + Aspire)

### 1. ASP.NET Core Project Layout

- **`Program.cs`**: Application startup, middleware pipeline, and Minimal API endpoint definitions
- **`Endpoints/`**: Endpoint groups and route handlers (Minimal APIs pattern)
- **`Services/`**: Domain logic, orchestrating agents and external services (registered via DI)
- **`Models/`**: DTOs, domain entities, and data models (use records for immutability)
- **`Data/`**: Cosmos DB repository implementations and database client configurations
  - **Prefer Cosmos DB for data persistence**: Use Azure Cosmos DB as the primary database for all data storage needs
  - **Do NOT use Entity Framework**: Use the native Azure Cosmos DB SDK (`Microsoft.Azure.Cosmos`) directly for all database operations. Avoid EF Core or other ORMs
  - Implement repository pattern with async methods for CRUD operations
  - **Auto-initialize database in development**: Create containers and databases automatically on startup in development mode using initialization service

### 2. Aspire Integration

- Reference the `ServiceDefaults` project from all .NET services for shared telemetry, service discovery, and resilience configuration
- Call `builder.AddServiceDefaults()` early in `Program.cs` to enable Aspire integrations
- Use `builder.AddAzureCosmosClient()`, `builder.AddRedisClient()`, `builder.AddAzureServiceBusClient()`, etc., from Aspire hosting integrations
- Define dependencies in `AppHost/Program.cs` using `WithReference()` for automatic service discovery and connection string injection

### 3. Type Safety & C# Best Practices

- **Nullable reference types**: Enable nullable reference types in all projects
  - Add `<Nullable>enable</Nullable>` to `.csproj` files
  - Treat nullable warnings as errors: `<WarningsAsErrors>nullable</WarningsAsErrors>`
- **Use record types**: Prefer `record` for DTOs and immutable data models
- **Use init-only properties**: Leverage `init` accessors for immutable objects
- **Pattern matching**: Use modern C# pattern matching for cleaner code
- **File-scoped namespaces**: Use file-scoped namespace declarations
- **Global usings**: Define common usings in `GlobalUsings.cs`
- **Required members**: Use `required` keyword for mandatory properties in C# 11+

### 4. Async Excellence & Best Practices

- All I/O operations must be async (`async`/`await`)
- Never block with `.Result` or `.Wait()`
- Use ASP.NET Core's built-in dependency injection with scoped, singleton, and transient lifetimes
- Minimize middleware complexity
- Keep hot code paths fast and avoid long-running tasks in the request pipeline
- Use `IAsyncEnumerable<T>` for streaming responses and pagination to avoid loading large datasets into memory

### 5. Realtime & Media

- SignalR hubs for real-time bidirectional communication (WebSockets with fallback)
- Define strongly-typed hub contracts
- Ensure proper backpressure handling in SignalR streaming methods using `IAsyncEnumerable<T>` or `ChannelReader<T>`
- For WebRTC signaling, use SignalR or dedicated WebSocket endpoints
- Offload TURN/STUN to Azure Communication Services
- Configure CORS policies for SignalR cross-origin connections

### 6. Agents Integration

- **Always use AI Toolkit best practices**: Before implementing agent code, use `#aitk-get_agent_code_gen_best_practices` to retrieve the latest best practices and guidance for Microsoft Agent Framework code generation
- **Required NuGet packages**: Install `Microsoft.Agents.AI`, `Microsoft.Agents.AI.OpenAI`, `Microsoft.Extensions.AI`, and `Microsoft.Extensions.AI.OpenAI` packages
- **Use `ChatClientAgent` for all agents**: Wrap all agents using `ChatClientAgent` from `Microsoft.Agents.AI` with proper agent descriptions, names, and instructions
- **Agent construction pattern**: Create agents with `new ChatClientAgent(chatClient, instructions: "...", name: "AgentName")` in service constructors
- **Use `AIAgent.RunAsync()` pattern**: Call agents using `await _agent.RunAsync(prompt)` instead of direct `IChatClient` calls. Never use `IChatClient` directly for agent operations
- **Configuration chain**: `Azure OpenAI Client → ChatClient → .AsIChatClient() → ChatClientAgent → AIAgent` is the proper initialization chain
- **Implement tools using `AIFunctionFactory`**: Define agent capabilities as tools using `AIFunctionFactory.Create()` with clear descriptions and parameter documentation
- **Tool registration pattern**: Register tools in agent constructor using `tools: [AIFunctionFactory.Create(Method1), AIFunctionFactory.Create(Method2)]` parameter
- **Equip agents with domain-specific tools**: Each agent should have 3+ relevant tools that enable specific capabilities (e.g., validation, optimization, analysis)
- **Use `IChatClient` abstraction**: All agent services should depend on `IChatClient` in constructors for flexibility and testability, but wrap it in `ChatClientAgent`
- **Implement orchestration patterns**: Support both sequential (one-after-another) and concurrent (parallel) agent execution patterns via orchestrator services. Use pattern matching for agent routing
- **Retry logic with compliance feedback**: Implement retry loops (5+ attempts) with exponential backoff. Pass rejection feedback from compliance agents to the next retry attempt
- **Campaign brief construction**: Build comprehensive context strings that include theme, product details, target audience, and optional revision feedback from previous iterations
- **Prefer AI-powered over rule-based logic**: Use LLM intelligence with tools for decision-making rather than simple string matching or rule engines
- **DI registration**: Register `IChatClient` as singleton via `.AsIChatClient()`. Register agent services as scoped for per-request isolation
- Implement Microsoft Agent Framework skills as injectable services (singleton or scoped)
- Use `IHostedService` or `BackgroundService` for long-running agent workers and background processing
- Persist agent memory/state in Cosmos DB using the native Azure Cosmos DB SDK with encryption at rest

### 7. Observability & Resilience

- OpenTelemetry is configured automatically via Aspire `ServiceDefaults` (logs, metrics, traces exported via OTLP)
- Use ASP.NET Core health checks (`MapHealthChecks()`) and expose via Aspire dashboard
- Implement resilience with Polly (circuit breakers, retries, timeouts) via `Microsoft.Extensions.Http.Resilience`
- Return Problem Details (RFC 7807) for errors using `Results.Problem()` in Minimal APIs
- Use structured logging with `ILogger<T>` and log scopes for correlation

### 8. Testing Strategy

- **Test Framework**: xUnit as preferred test framework. NUnit and MSTest are acceptable alternatives
- **Integration Tests**: Use `WebApplicationFactory<TProgram>` for integration tests, testing full HTTP pipelines
- **Assertions**: FluentAssertions for readable test assertions
- **Coverage Areas**:
  - Unit tests (services, domain logic)
  - Integration tests (API endpoints, database)
  - Contract tests (OpenAPI validation)
- Use Testcontainers for integration tests requiring databases or message brokers
- Load testing with k6, JMeter, or NBomber for high-throughput endpoints and SignalR hubs
- Keep coverage ≥ 85%

## .NET Aspire Orchestration Guidelines

.NET Aspire is the local development orchestrator for cloud-native distributed applications. It simplifies service dependencies, configuration, and observability during development.

### 1. AppHost Project Structure

- Create an AppHost project using `dotnet new aspire-apphost`
- Define your app model in `Program.cs` using `DistributedApplication.CreateBuilder()`
- **All service hosts must be defined in the AppHost**: Add resources using `.AddProject<T>()`, `.AddContainer()`, `.AddRedis()`, `.AddAzureCosmosDB()`, `.AddAzureServiceBus()`, etc.
- **Use Aspire client integrations in services**: Wire up Aspire clients (e.g., `builder.AddAzureCosmosClient()`, `builder.AddRedisClient()`) in consuming services to leverage Aspire's service discovery and configuration
- **Use local emulators for development**: Configure Aspire to use local emulators (Cosmos DB Emulator, Azurite, Redis locally) via `.RunAsEmulator()` or `.RunAsContainer()` for local development. Never require cloud resources for local dev
- For Node.js/Next.js frontends: Add `Aspire.Hosting.NodeJs` package and use `.AddNpmApp("webapp", "../{appname}.Web")` to integrate frontend apps
- Express dependencies with `.WithReference()` for automatic service discovery and connection string injection
- Use `.WaitFor()` to ensure startup ordering between services

### 2. ServiceDefaults Project

- Create a shared ServiceDefaults project using `dotnet new aspire-servicedefaults`
- All .NET services should reference this project and call `builder.AddServiceDefaults()` in `Program.cs`
- ServiceDefaults configures: OpenTelemetry (logs, metrics, traces), service discovery, HTTP resilience (Polly), and health checks
- Customize telemetry, add global middleware, or configure default HTTP client behaviors here

### 3. Service Discovery

- Services reference each other using logical names defined in the AppHost (e.g., `"apiservice"`)
- Use `https+http://apiservice` as the base URL in HttpClient configurations—Aspire resolves to actual endpoints
- Environment variables are automatically injected: `services__apiservice__http__0`, `ConnectionStrings__cache`, etc.

### 4. Deployment

- Aspire is for local development only. **Do not deploy the AppHost to production**
- Use `azd init` and `azd up` to deploy to Azure Container Apps with auto-generated infrastructure
- Run `aspire publish` to generate deployment manifests for custom targets (Kubernetes, Docker Compose)
- For production, deploy services independently to Azure Container Apps, Azure App Service, or Kubernetes

### 5. Dashboard

- Aspire provides a built-in dashboard at runtime showing all resources, logs, traces, and metrics
- Access via the URL shown when running the AppHost project
- Use for debugging service communication, observing telemetry, and monitoring resource health

## Code Quality

- **Pre-commit hooks** running:
  - **dotnet format or CSharpier**: Consistent code formatting
  - **Roslyn analyzers**: Enable StyleCop, SonarAnalyzer.CSharp, Microsoft.CodeAnalysis.NetAnalyzers
  - **Commit message lint**: Enforce Conventional Commits format
- **EditorConfig**: Use `.editorconfig` for consistent coding standards across IDEs
- **Automated via IDE integration**: Configure Visual Studio, VS Code, or Rider
- **CI enforcement**: Run `dotnet format --verify-no-changes` in CI pipeline
- **Static analysis**: Configure analyzers in `Directory.Build.props`:
  ```xml
  <PropertyGroup>
    <AnalysisLevel>latest</AnalysisLevel>
    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  </PropertyGroup>
  ```

## Best Practices

- **Latest NuGet packages**: Always use latest stable NuGet packages unless there's a specific compatibility constraint
- **Central Package Management**: Use `Directory.Packages.props` for centralized NuGet package version management
- **Dependency updates**: Update dependencies regularly via Renovate bot with grouped update strategy
- **Cloud-native APIs**: High-performance APIs with built-in DI, middleware pipeline, and async-first patterns
- **Strong typing**: Record types and nullable reference types throughout
- **Build enforcement**: Fail builds on nullable warnings with `<WarningsAsErrors>nullable</WarningsAsErrors>`
- **Secrets management**: .NET User Secrets (`dotnet user-secrets`) for local development, Azure Key Vault for cloud environments
- **API documentation**: Generate OpenAPI/Swagger docs via Swashbuckle or NSwag with XML documentation comments
