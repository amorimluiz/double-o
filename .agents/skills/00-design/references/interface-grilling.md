# Interface Grilling

How to explore and question the interface before prototypes exist. The mechanics — one question per message, recommendation first, multiple-choice with a fallback, explore before asking — are the shared protocol in `00-prd/references/question-protocol.md`; this file owns only what is specific to design.

## Decision tree

Walk the branches in order; each one unblocks the next. Explore the codebase and `DESIGN.md` first, and never spend a question on what they already answer.

1. **Screens & flow** — which screens or views the feature needs, new versus changed, their entry points, and how the user moves between them. Everything downstream depends on this.
2. **Information hierarchy** — per screen, what the user sees first and the single primary action. Resolve competing primary actions here, not in layout.
3. **States** — per screen, the loading, empty, error, success, and disabled/permission states, and what each shows. A state left out is a prototype that lies.
4. **Layout & density** — the content model (list, table, cards, form), narrow-to-wide behavior, and whether it is mobile-first. Reuse the project's existing layout patterns.
5. **Components & reuse** — which existing components and tokens cover each element, and which genuinely new component the feature needs. A new component is a decision, not a convenience.
6. **Interaction & motion** — the actions, confirmations, destructive flows, and any transition the user should feel. Respect reduced motion.
7. **Content** — real labels, empty-state copy, error messages, and locale behavior. Copy is interface, not filler.
8. **Accessibility** — focus order, keyboard operation, target size, and contrast targets where the standard has a genuine choice.

## Progression gates

- Cover screens and flow before layout: a grid decision made before the screen inventory is rework.
- Decide hierarchy and states before components: the state set is what a component must support.
- Stop when every branch has a confirmed decision or is parked with the user's consent. The question count follows the tree, not a budget.

## Focus boundaries

- Interface sessions ask WHAT the user sees and does; data models, endpoints, and code structure belong to `00-techspec` and are read, not re-decided.
- Reuse the project's vocabulary and existing patterns; a design question is about intent and trade-offs, not about what the codebase already fixes.
