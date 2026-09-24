# Fidelity Capture and Gate

How `00-design` grounds prototypes in the app's **real** interface so a new screen extends what exists instead of inventing a parallel visual language. AI generators default to generic shadcn/Tailwind styling unless they are handed the actual screens, tokens, and components; this procedure supplies them and then proves the result conforms.

Precedence, highest first: the captured real UI, then `DESIGN.md`, then `ui-ux-pro-max`. A database recommendation never overrides the captured system.

## 1. Reach the running app

- Resolve the dev-server command from the project README, `AGENTS.md`, or `package.json`; start it and confirm it serves.
- Drive the browser with the `agent-browser` skill. Open the screens the feature touches, authenticated when the project requires it.
- The app is the source of truth; a screenshot alone throws away exact values, so read the rendered styles too.

## 2. Capture each in-scope screen

For every screen the feature adds to or changes:

- **Screenshot** at the reference viewport and one narrow viewport.
- **Computed styles** for the token roles actually in use: surfaces, text, primary and destructive actions, borders, radius, spacing rhythm, the type scale, and shadows. Read them from the live DOM, not from memory.
- **Component inventory**: the real controls present (buttons, inputs, selects, cards, tables, dialogs, nav, badges) and how each is composed. This is the reuse palette.

## 3. Write the capture under `design/reference/`

```
design/reference/
  screens/NN-<screen>.png      # reference screenshots
  screens/NN-<screen>.md       # what the screen shows, its primary action, its states
  tokens.md                    # extracted tokens, each with its source (computed style | DESIGN.md)
  components.md                # component inventory with the real usage per element
  fidelity.md                  # the contract: allowed, forbidden, extend-only, approvals
```

`fidelity.md` is the contract the prototypes are held to:

- **Allowed** tokens and components — the closed set from `tokens.md` and `components.md`.
- **Forbidden** — invented colors, spacing, radii, type, or shadows; a new component where an existing one fits; a parallel styling system.
- **Extend-only** — a prototype extends the existing visual language. It never introduces a new one.
- **New components** — a genuinely new component is a decision that requires explicit user approval at the gate, with the reason no existing component fits.

## 4. Static fallback

When the app cannot be reached, extract the same set from the codebase: CSS variables, the Tailwind theme, theme files, existing components, and `DESIGN.md`. Label the capture **degraded** in `fidelity.md` and state the reason, and say so at the gate. Never present a degraded capture as the live interface.

## 5. Fidelity gate

Before the approval gate, compare every generated prototype against the capture and iterate until it conforms:

- **No invented tokens**: every color, spacing, radius, type, and shadow value maps to `tokens.md`.
- **Components reused**: every control maps to `components.md`; a new component has the recorded approval.
- **Layout and hierarchy**: the prototype's structure and primary action match the reference note for the screen.

Report the comparison at the gate — what matched, what changed, and any remaining divergence with its reason. A prototype that reads as a different product is a failure of this gate, not a styling nit.
