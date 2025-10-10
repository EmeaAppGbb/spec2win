---
mode: devlead
---
# Generate AGENTS.md from Guidelines

Your task is to read the project guidelines from the `/standards/` folder and create comprehensive AGENTS.md files that summarize all relevant rules and best practices for either backend or frontend development.

## For Backend AGENTS.md
When creating a backend AGENTS.md file, you must:
1. Read and synthesize content from:
   - `/standards/general/general-guidelines.md` (general principles)
   - `/standards/backend/backend-guidelines.md` (backend-specific guidelines)
2. Create a comprehensive AGENTS.md file in the target backend folder
3. Organize the content logically based on the structure and topics defined in the guidelines

## For Frontend AGENTS.md
When creating a frontend AGENTS.md file, you must:
1. Read and synthesize content from:
   - `/standards/general/general-guidelines.md` (general principles)
   - `/standards/frontend/frontend-guidelines.md` (frontend-specific guidelines)
2. Create a comprehensive AGENTS.md file in the target frontend folder
3. Organize the content logically based on the structure and topics defined in the guidelines

## Guidelines for Content Synthesis

1. **Be Comprehensive**: Include ALL relevant guidelines from both general and domain-specific files
2. **Maintain Clarity**: Organize content in a logical, hierarchical structure
3. **Preserve Details**: Keep specific technical details, version numbers, package names, and configuration examples
4. **Use Markdown Formatting**: Proper headers, lists, code blocks, and tables for readability
5. **Cross-reference**: When general and specific guidelines overlap, merge them coherently
6. **Highlight Critical Rules**: Emphasize non-negotiable requirements (e.g., type safety, security, testing coverage)
7. **Include Examples**: Keep code samples and configuration snippets where provided
8. **Maintain Consistency**: Use consistent terminology and formatting throughout

## Output Format

The generated AGENTS.md should follow this structure:

```markdown
# AGENTS.md

## Mission
[Mission statement from general guidelines]

## Guiding Principles
[Combined principles from general and domain-specific guidelines]

## Canonical Stack
[Technology stack table with details]

## Architecture Blueprint
[Architecture patterns and structure]

## [Domain]-Specific Playbook
[Detailed implementation guidelines for backend or frontend]

## Agent-First Delivery
[Agent integration patterns if applicable]

## Shared Engineering Systems
[Code quality, documentation, secrets management, etc.]

## CI/CD Pipeline Expectations
[Pipeline stages, preview environments, deployment]

## Security & Compliance
[Security best practices and compliance requirements]

## Documentation & Knowledge Sharing
[Documentation standards and practices]

## Quality Gates Checklist
[Comprehensive quality checklist]

## Change Management
[PR and merge policies]
```

## Execution Steps

When invoked:
1. **Clarify the target**: Ask whether they want backend or frontend AGENTS.md (if not specified)
2. **Read the guidelines**: Always read the latest content from the `/standards/` folder
3. **Synthesize content**: Combine general and domain-specific guidelines
4. **Generate the file**: Create a comprehensive, well-structured AGENTS.md file
5. **Confirm completion**: Let the user know the file has been created and where

## Important Notes

- Always read the LATEST content from the guidelines files - never use cached or outdated information
- Ensure no guidelines are omitted - completeness is critical
- Maintain the technical accuracy of all specifications, package names, and version requirements
- Keep the document maintainable - use clear sections and proper formatting
- If guidelines conflict, prefer the domain-specific guidelines over general ones
