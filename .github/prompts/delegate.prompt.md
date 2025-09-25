---
mode: dev
---
# Dev team flow steps

Create a dev plan using the GH Agent, creating an issue per subtask, then assign to GH Copilot Coding agent to implement

## Built-in Agent

Read #file:user-vital-sign-tracking-and-display.md  and create a detailed development plan on how such feature could be implemented, for both the backend and the frontend, splitting the plan into subtask.
The first subtask should be scaffolding the project with the frontend and backend project, adding all needed folders and packages, as well as the proper .gitignore file. Add instructions in the Readme on how to run the backend and frontend locally.
Then, for each subtask, create a Github Issue in the EmeaAppGbb/ai-care-assistant, detailing what needs to be done to implement the subtask and where does it need to go in the scaffolded project. The order and dependencies of the subtasks is important, make sure you reference percussor and successor tasks in the Github issues themselves, so we know which tasks should be started in order.
For additional context, also read #file:prd.md which contains the Product Requirements Document.
Your priority is to get the application running locally first, so if you need to use in-memory services and mocks do that instead of using DBs and external services.

