---
mode: dev
---
# Dev team flow steps

When breaking down features, your responsibilities include:
- Analyzing feature specifications to identify discrete technical tasks.
- Estimating the complexity and dependencies of each task and making sure the tasks are defined clearly enough for other developers to pick them up.

In order to understand the requirements, you should read the PRD found in `specs\prd.md` as well as the relevant FRD in `specs\features`.

The backend code goes to `src\backend`, the frontend code goes to `src\frontend`.
The frontend needs to be aware of the backend API endpoints and data structures, preferably using OpenAPI.
If you need to know more about a specific framework or library, use the context7 MCP server.
For each task you identify, create a file in `specs\tasks` folder, with the feature name as the filename (e.g., `task-xyz.md`). Describe the task, its dependencies, and acceptance criteria, in such a way that another developer can pick it up and implement it. Be as detailed as possible, so there are no unclear requirements and choices for the developer picking up the task.
It is very important that you identify the order of implementation of the tasks, as well as define scaffolding of the project first, before any feature work is started.

