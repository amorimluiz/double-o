# PRD Template

Structure for `.sdd/<slug>/prd.md`, consumed by `00-techspec`, `00-tasks`, and `00-loop`. It supplies business rules, domain behavior, and product intent. Fill each section per its rules, write in the project's artifact language, and leave unknown material in Open Questions rather than guessing.

## Overview

What the feature is, the problem it solves, who it is for, and why it is valuable. Two to four paragraphs.

## Goals

Product outcomes stated as observable behavior, not metrics:

- What users can do after this ships that they could not do before
- What the system guarantees or enforces once the feature exists
- What becomes unnecessary, automatic, or impossible for users

## User Stories

The canonical story catalog lives in this section. Cover every persona, secondary ones included, and every core feature.

| ID | Persona | Story | Acceptance criteria | Edge cases |
| --- | --- | --- | --- | --- |
| US-01 | [persona] | As a [persona] I want [capability] so that [outcome] | [verifiable statements] | [expected behavior per case] |

Rules:

- Every story has verifiable acceptance criteria — no adjectives without a check.
- Sweep every story against the edge-case classes: empty, invalid, boundary, duplicate, concurrent, unauthorized, offline.
- Prefer concrete examples over abstractions; an engineering agent implements the examples.

## Core Features

One subsection per feature: what it does, why it matters, high-level behavior, and how it interacts with other features. Functional requirements for each feature, stated precisely.

## Business Rules

Domain rules the implementation must enforce:

- Invariants that must always hold
- Validation rules and their user-facing outcomes
- Permission and visibility rules per persona
- Lifecycle and state-transition rules: which states exist, what may move where, and when
- Calculations, limits, and defaults with exact values

## User Experience

The journey from first contact to regular use:

- Key personas and their goals
- Primary user flows, step by step
- UX considerations and accessibility requirements
- Onboarding and discoverability

## Technical Constraints

Boundaries that shape the product without prescribing implementation:

- Required integrations with existing systems
- Compliance mandates or regulatory requirements
- Performance targets from the user's perspective
- Data privacy and security requirements

Implementation choices — databases, frameworks, architecture patterns — belong to the TechSpec.

## Non-Goals

Capabilities the user decided this feature will not include, each with the reason. Exclusions record user decisions, never size management: a wanted capability stays in scope no matter how large the document grows.

## Decisions

Product decisions made during grilling: the decision, why, and what was rejected. One line each when the rationale is obvious.

## Open Questions

Unresolved items, each with who can answer it or what would unblock it. Empty when the PRD is ready.
