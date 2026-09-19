# Question Protocol

How to run the grilling conversation that turns an idea into decided requirements. The mechanics are mandatory in every `00-prd` and `00-techspec` session.

## Mechanics

- Use the runtime's dedicated interactive question tool — the mechanism that presents a question and pauses until the user answers. When the runtime has none, present the question as the complete message and stop generating.
- One question per message: exactly one question mark, then stop. A topic that needs more exploration gets its follow-up after the user answers.
- Lead with a recommendation: one line stating which option you would pick and why, so the user reacts to a position instead of facing a blank menu.
- Multiple-choice whenever options can be predetermined: labeled A, B, C with your recommendation first, plus a fallback "Other — describe". Open-ended only when the answer space is genuinely unbounded.
- Never answer for the user, and never batch questions to save turns.

## Grilling Method

- Map the topic into a decision tree: which decisions exist, and which depend on which.
- Ask the question that unblocks the most downstream decisions first; resolve dependencies one at a time.
- Chase vague answers: "it depends" gets "on what?", "probably" gets pinned down. A load-bearing branch left fuzzy resurfaces as rework after the document ships.
- Explore before asking: when the codebase, an existing artifact, or the research tracks already answer a question, take the answer from there and move to the next branch. User answers are for genuine unknowns — intent, priorities, and trade-offs.
- Stop when every branch has a confirmed decision or is explicitly parked in Open Questions with the user's consent. The question count is an output of the decision tree, not a budget.

## Phases

1. Discovery — the core problem or opportunity, who is affected, what prompted the initiative.
2. Understanding — what users need, why it matters, who they are, what must be true when it ships, known constraints.
3. Refinement — scope boundaries, expected behavior per feature, remaining open questions.

## Progression Gates

- Complete at least one full Understanding round before any direction is decided.
- Decide the direction only when every branch it depends on is resolved: purpose, constraints, and expected behavior.

## Focus Boundaries

- Product sessions (`00-prd`) ask WHAT, WHY, and WHO; implementation topics — databases, APIs, frameworks, code structure, testing strategy, deployment — belong to the technical session.
- Technical sessions (`00-techspec`) ask HOW and WHERE: architecture and component boundaries, data models and storage, API design and integration, testing strategy, performance.
- Never spend a question on what the codebase can answer: explore first. User answers are for trade-offs, priorities, and risk appetite.
