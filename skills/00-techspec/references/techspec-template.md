# TechSpec Template

Structure for `.sdd/<slug>/techspec.md`. Fill each section from the grilling outcomes and codebase exploration; omit sections that do not apply and note the reason. Reference PRD sections by name without duplicating business context.

## Executive Summary

One to two paragraphs: the key architectural decisions, the implementation strategy, and the primary trade-offs.

## System Architecture

Main components, their responsibilities, and their relationships:

- Component name, purpose, and boundaries
- Data flow between components
- External system interactions

Include an inline Mermaid diagram when it makes the architecture or a flow clearer than prose; use the `mermaid-diagrams` skill when available.

## Implementation Design

### Core Interfaces

The primary types other components depend on, in the project's language, with code examples of 20 lines or fewer:

- Interface definitions and contracts
- Method signatures with parameter and return types
- Error handling conventions

### Data Models

Core domain entities and their relationships:

- Entity definitions with field types
- Request and response types
- Storage schemas or structures

### API Endpoints

The API surface organized by resource, when the feature exposes one:

- Method, path, and description
- Request format and required fields
- Response format and status codes

## Integration Points

Only when the design integrates with systems outside the codebase:

- Service name and purpose of the integration
- Authentication and authorization approach
- Error handling and retry strategy

## Impact Analysis

| Component | Impact | Description and risk | Required action |
| --- | --- | --- | --- |
| [component] | [new/modified/deprecated] | [what changes, risk level] | [action needed] |

## Testing Approach

Strategy only — concrete test cases are assigned to tasks in `tasks.md`:

- Frameworks, harnesses, and fixture strategy; fakes sit at I/O boundaries only
- What each level covers for this feature (unit, integration, end-to-end) and how it runs
- Environment or data dependencies the integration and end-to-end suites need
- How the project's coverage floor applies to the new code

## Development Sequencing

1. [First component] — no dependencies
2. [Second component] — depends on step 1
3. [Continue along the dependency chain]

Blocking dependencies: infrastructure, external services, or shared deliverables that must exist before implementation.

## Decisions

Significant technical choices made during grilling:

- Decision: what was chosen
- Rationale: why this option
- Trade-offs: what was given up
- Alternatives rejected: what else was considered and why not

## Known Risks

- Risk description and likelihood
- Mitigation approach
- Areas requiring prototyping or further research

## Open Questions

Unresolved items, each with who can answer it or what would unblock it. Empty when the TechSpec is ready.
