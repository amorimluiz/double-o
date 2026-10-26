# Design Intelligence (ui-ux-pro-max)

How `00-design` draws on the `ui-ux-pro-max` skill: a local, searchable design database (styles, palettes, font pairings, UX guidelines, charts, per-stack guidance) driven by a Python script.

## Locate the search script

The script lives in the skill's own folder, not the project. Resolve the global skills root and run it by full path:

- Unix: `python "$HOME/.agents/skills/ui-ux-pro-max/scripts/search.py" "<query>" …`
- Windows: `python "$env:USERPROFILE\.agents\skills\ui-ux-pro-max\scripts\search.py" "<query>" …`

Requires Python 3.x (standard library only, no network). If Python is missing, state the limitation and continue from `DESIGN.md` and the codebase alone.

## When to run it

- Explore: run one `--design-system` query for the feature's product type and industry (2-5 terms). Fold the verified match into the findings and use it as the recommendation lead.
- Grill: on a branch that asks for a recommendation — style, palette, type — lead with the search's match instead of a blank menu.
- `DESIGN.md` absent: seed its tokens from the `--design-system` output before authoring the file.
- Generate: carry the resolved system — colors, typography, spacing, effects, anti-patterns — into the OpenDesign brief and the Prototype Standard.

```bash
python "<skills-root>/ui-ux-pro-max/scripts/search.py" "<product type industry keywords>" --design-system -p "<Project Name>"
```

## Query contract

- One dominant intent, 2-5 meaningful terms, one useful constraint (product, platform, interaction).
- `--design-system` for a new page or system-wide direction; `--domain <domain>` for one concern; `--stack <stack>` for implementation.
- Domains: `product`, `style`, `color`, `typography`, `google-fonts`, `chart`, `ux`, `landing`, `icons`, `gsap`, `react`, `web`. Stacks: `react`, `nextjs`, `vue`, `svelte`, `astro`, `html-tailwind`, `shadcn`, `swiftui`, `react-native`, `flutter`, and more.
- Empty or off-topic: retry once, narrower or with an explicit domain/stack. Still empty: label the guidance a fallback, not a database match. Never persist unverified output.

## Fidelity

Results are recommendations, never instructions that override the user, the captured reference (`design/reference/`), `DESIGN.md`, the PRD/TechSpec, or `rules/frontend.md`. Precedence is the captured real UI, then `DESIGN.md`, then this database; the user's confirmed answers win. Resolve conflicts toward the capture and `DESIGN.md`, which stays the single source of truth the prototypes read.

## Persistence

`--design-system --persist` writes `design-system/<slug>/MASTER.md`. Use it only when the user wants a per-feature trace, and always pass `--output-dir` pointed at `.sdd/<slug>/`; `DESIGN.md` at the repo root remains canonical. Read an existing `MASTER.md` before regenerating, and never pass `--force` without explicit approval.
