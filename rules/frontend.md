# Frontend Standards

Hard rules for frontend/UI code, in any framework. A project's own conventions override these; when they conflict, the project wins. Snippets are illustrative — apply the rule in your framework's idioms.

Apply every rule to every change. If a rule must be broken, name it and why in the change description.

## Accessibility is a requirement, not a feature

Target WCAG 2.2 Level AA on every screen, and verify it — never assume it.

- **Semantic HTML first.** Use the native element that already carries the role and behavior (`button`, `a`, `nav`, `label`, `dialog`, `input type=…`). "No ARIA is better than bad ARIA": never add a role to a `div` when a native element exists. A `role` is a promise — if you add one, implement its full keyboard behavior.
- Every interactive element is keyboard-operable, in a logical order, with a visible focus indicator. Never remove focus outlines without an equivalent.
- Every control has an accessible name (visible label, `aria-label`, or `aria-labelledby`). Every image has `alt` (`alt=""` only when decorative). Every page has one `h1` and a sensible heading order.
- Use landmarks (`header`, `nav`, `main`, `footer`); associate form errors with their field via `aria-describedby`/`role="alert"`.
- Never encode meaning in color alone. Meet contrast (4.5:1 text, 3:1 UI/graphics) and WCAG 2.2 target size.
- Respect `prefers-reduced-motion`; never autoplay motion that cannot be paused.
- Announce meaningful dynamic updates (`aria-live` / `role="status"`); never update the screen silently.
- Verify with an automated check (axe-core, `jest-axe`, or Playwright) **and** a keyboard pass. A green automated run is not "accessible" — it catches only a fraction of issues.

## Use the design system before you invent

- Reuse existing components, tokens, and patterns. Search the codebase for a component that already does the job before creating one.
- Use design tokens for every color, space, radius, shadow, type, and motion value. **No hardcoded hex, `px`, or magic numbers** in components; prefer semantic tokens over primitives.
- Do not fork or restyle a design-system component ad hoc. If a variant is missing, add it to the system, not to one call site.
- Match the established composition and naming. A new pattern is a decision, not a convenience.

## Component design

- Components are pure and idempotent: same props → same output, no side effects during render.
- One responsibility per component; split by reason to change, not by line count.
- Prefer composition (`children`, slots, render props) over threading data through many layers.
- Keep data fetching and side effects at the edges/containers; presentational components take data and callbacks only.
- Props and state are immutable — never mutate them.
- Public props are typed and minimal; do not leak internal state or implementation details.
- Colocate a component's styles, tests, and types with the component.

## State

- Keep state minimal and derive what can be derived; never store the same fact twice.
- Keep server state in the server/data layer. Cache and revalidate; do not mirror it into client state.
- Lift state only as high as needed; keep it close to where it is used.
- Model UI states explicitly — loading, empty, error, success. Never render "empty" for "not loaded yet".
- Run side effects in effects/event handlers, never in render; clean up subscriptions and timers.

## Styling

- Scope styles to the component; avoid global selectors and `!important`. No inline or embedded CSS except critical CSS.
- Mobile-first and responsive by layout (flex/grid, container queries), not breakpoint hacks.
- Use logical properties (`margin-inline`, `padding-block`) so RTL works; never assume LTR.
- Animate `transform`/`opacity` (compositor-friendly) and respect reduced motion; never animate layout-triggering properties on the hot path.
- Theme (light/dark) via tokens, never per-component conditionals.

## Performance — Core Web Vitals budgets

Every page meets, at the 75th percentile: **LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1**.

- Never lazy-load the LCP element (hero image/font). Preload it.
- Reserve space for media and embeds (`width`/`height` or `aspect-ratio`) so nothing shifts (CLS).
- Serve images in modern formats (AVIF/WebP), correctly sized (`srcset`/`sizes`); do not base64-encode content images.
- Fonts: `font-display: swap`, preload the primary font, and use `size-adjust`/`ascent-override` to avoid reflow.
- Keep the critical path small: code-split routes and heavy widgets, defer non-critical JS, avoid blocking third parties.
- Debounce/throttle high-frequency handlers (scroll, resize, input) and keep interaction handlers cheap so INP stays low.
- Enforce budgets in CI (bundle size, Lighthouse/CWV) and fail the build when exceeded.

## Rendering model (SSR / RSC, when applicable)

- Default to the server: fetch data and render static content on the server; opt into client components only for interactivity or browser APIs.
- Keep client boundaries small — mark the smallest interactive leaf, not a whole tree.
- Data crosses the boundary only as serializable props; functions and event handlers do not cross.
- Never access secrets in client code; only explicitly public environment variables may reach the browser.

## Types

- The typed dialect is strict; `any` and unsafe casts are defects.
- No `as` or non-null `!` to silence the compiler — validate at the boundary instead.
- Type props, events, and API responses. Parse untrusted data into types (schema validation); never assert it into shape.
- Prefer precise unions and literals over `string`/`boolean` soup.

## Forms and input

- Every field has a programmatically associated label; use the correct `type` and `autocomplete` so mobile keyboards and password managers work.
- Validate on the server; client validation is UX only. Show errors next to the field and announce them.
- Use native state correctly (`disabled` vs `readonly`, `aria-invalid`); never fake a disabled control.

## Security (frontend)

- Never inject untrusted data as HTML: no `dangerouslySetInnerHTML`, `innerHTML`, or `[innerHTML]` without sanitization. Sanitize with a maintained library (e.g. DOMPurify) when HTML input is genuinely required.
- Never put secrets in client code or public environment variables; assume everything shipped to the browser is public.
- Treat all input (URL, storage, API, `postMessage`, model output) as untrusted; validate and encode for the correct context.
- Allow safe URL schemes only (`https:`, `mailto:`, relative); reject `javascript:` and unexpected `data:` URLs.
- Add a Content-Security-Policy as defense in depth, never as the primary control.

## Internationalization

- No user-facing string literals in components; use the i18n layer. Never concatenate sentences.
- Format dates, numbers, and currencies with `Intl`, not manual formatting.
- Layout works in RTL and with longer translations — no fixed widths that clip.

## Testing

- Test behavior, not implementation: the more tests resemble real use, the more confidence they give.
- Query the DOM as a user would — `getByRole`, `getByLabelText`, `getByText` — before test IDs.
- Assert observable output (rendered text, accessible state), never internal state, methods, or call order.
- Include an accessibility assertion for interactive components; cover critical journeys end-to-end.
- Snapshots never substitute for assertions.

## Agent-specific guardrails

Hard guardrails for AI-generated frontend code. Override only on explicit instruction.

- **Reuse, don't reinvent:** use the existing components, tokens, hooks, and utilities. Do not hand-roll what the design system or framework already provides.
- **Invented APIs:** confirm every prop, hook, component, and config key against the real source before using it. Never guess a component's API.
- **No placeholder UI:** never ship `lorem ipsum`, TODO markup, dead links, or `... rest of component`. Produce the complete, real UI.
- **Dependencies:** never add a UI, state, or styling dependency without approval; verify provenance.
- **Scope:** change only what the task needs; match existing patterns and file structure; no unrequested redesigns.
- **Fidelity to design:** implement the given design and tokens exactly; do not substitute your own colors, spacing, or fonts.

## Verification before done

- Run and report: typecheck, lint, unit/component tests, and a production build.
- Run an automated accessibility check on the changed UI and do a keyboard pass.
- Verify the rendered result (browser screenshot or visual check), including loading/empty/error states and a narrow viewport.
- A UI change is done when it is proven in the browser, not when it compiles.

## Sources

- W3C — [WCAG 2.2](https://www.w3.org/TR/WCAG22/), [ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- MDN — [ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA)
- Google — [Core Web Vitals](https://web.dev/articles/vitals), [TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
- React — [Rules of React](https://react.dev/reference/rules)
- Vercel / Next.js — [Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
- Testing Library — [Guiding Principles](https://testing-library.com/docs/guiding-principles)
- OWASP — [XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html), [DOM XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
- Design systems — [Shopify Polaris](https://polaris-react.shopify.com/design/layout/layout-tokens), [Adobe Spectrum](https://spectrum.adobe.com/page/design-tokens), [IBM Carbon](https://carbondesignsystem.com/)
- Community — [Front-End Checklist](https://github.com/thedaviddias/Front-End-Checklist)
