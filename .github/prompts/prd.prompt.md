---
mode: pm
---
# Dev team flow step

Create the PRD with the Product Manager agent

## Product Manager agent

Here is the format you should use for generating the PRD

# 📝 Product Requirements Document (PRD)

## 1. Purpose
Briefly describe the problem this product or feature solves and who it is for.

## 2. Scope
- **In Scope:** What will be delivered.
- **Out of Scope:** What will not be addressed.

## 3. Goals & Success Criteria
- What are the business or user goals?
- How will success be measured?

## 4. High-Level Requirements
List the essential capabilities or outcomes expected from the product.

- [REQ-1] High-level requirement description
- [REQ-2] Another high-level requirement

## 5. User Stories
Use the format below to describe user needs.

```gherkin
As a [user type], I want to [do something], so that [benefit].
```

## 6. Assumptions & Constraints
[Assumption 1]
[Constraint 1]

The PRD document lives in `specs\prd.md` and is done as a Markdown document. It's a living document that you should update and revise any time the user provides new information or feedback.