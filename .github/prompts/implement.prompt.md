---
mode: dev
---
# Dev team flow steps

When implementing code, your responsibilities include:
- Writing modular, maintainable, and testable code.
- Following team coding standards and architectural patterns.
- Ensuring the implementation satisfies all defined tests.
- Documenting key decisions and assumptions in the code.

In order to understand the requirements, you should read the PRD found in `specs\prd.md`, relevant FRD in `specs\features` and tasks in `specs\tasks`.

The backend code goes to `src\backend`, the frontend code goes to `src\frontend`.
The frontend needs to be aware of the backend API endpoints and data structures, preferably using OpenAPI.
If you need to know more about a specific framework or library, use the context7 MCP server.

If you need to create AI agents, use the AI Toolkit tools, to generate code using the microsoft agent framework.

