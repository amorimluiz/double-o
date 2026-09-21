# Git Workflow

Hard rules for version control, in every language and stack. A project's own conventions override the specifics here — merge strategy, DCO sign-off, commit language, branch prefixes. Where a project does not specify, these defaults apply. Snippets are illustrative.

Apply every rule to every change. If a rule must be broken, name it and why in the change description.

## Commit message

Use Conventional Commits: `type(scope): description`.

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

- **Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`. Only `feat` and `fix` are normative; the others are conventions with no version bump unless the change is breaking.
- **Scope** names the affected area (`feat(auth):`). Omit it when the change is repo-wide; keep one consistent vocabulary, never a new scope per commit.
- **Description:** imperative mood, lowercase after the prefix, no trailing period, ≤50 characters (hard cap 72). It must complete "If applied, this commit will ___".
- **Body** after a blank line: explain why, not what — the diff already shows what. Wrap at 72.
- **Breaking changes:** `!` before the colon plus a `BREAKING CHANGE:` footer describing the migration. This forces a MAJOR bump.
- Keep the subject plain text; the type prefix carries the meaning.
- **Language:** English by default. Use another language only when the repository requires it.
- Reference issues in a footer (`Closes #123`, `Refs #123`). Where the project bans close keywords in commits, put them in the pull request instead.
- `revert` uses `revert: <original subject>` with a `Refs:` footer naming the reverted SHA.

```
feat(auth): add refresh token rotation

Sessions were expiring mid-request under clock skew. Rotate the refresh
token on each use so short-lived access tokens renew safely.

BREAKING CHANGE: clients must persist the rotated refresh token.
Closes #412
```

## Atomic commits

One commit is one logical change: self-contained, buildable, testable, and revertible on its own.

- Code and its tests go in the same commit. Never split them.
- Keep mechanical changes (rename, move, format) separate from behavior changes.
- If describing the commit needs the word "and", split it. Do not split so finely that intermediate commits fail to build.
- The commit is the unit of `revert` and `bisect`. A commit that cannot be built or tested alone is a checkpoint, not a commit.
- Atomicity is not a size limit: a large single-purpose refactor is atomic; a small mixed change is not.

## Branches — GitHub Flow

- Branch from `main`, do the work, open a pull request, merge, delete the branch.
- Keep branches short-lived. When a branch outlives its task, integrate it or split it.
- Name: `type/short-description` in kebab-case, optionally with a ticket (`feat/refresh-token`, `fix/412-null-session`). Types mirror the commit types.
- Never commit directly to a shared or protected `main`. In a solo workspace with no review and no protection, direct commits are allowed; still keep `main` releasable.
- Hide incomplete features behind flags instead of keeping a long-lived branch.

## Merging

- If the repository defines a merge strategy, follow it. Otherwise default to a **merge commit** (`--no-ff`) so the integration point is explicit.
- Rebase only your own unpublished branch to keep it current; never rebase or rewrite published history.
- Never force-push a shared branch. On your own branch use `--force-with-lease`, and only when necessary.
- On shared history, prefer `revert` — a new commit — over rewriting.
- Delete the branch after merge.

## Keep out of history

Never commit:

- secrets: `.env`, keys, tokens, credentials — commit a `.env.example` instead;
- build output and generated artifacts (`dist/`, `target/`, caches, coverage);
- dependency trees (`node_modules/`, `vendor/`, virtualenvs) — commit the lockfile;
- IDE, editor, and OS files (`.idea/`, `.vscode/` project settings, `.DS_Store`);
- large binaries or data unless the project has an explicit LFS policy;
- debug code, logs, commented-out code, or temporary hacks.

Add ignore rules before the first commit, not after.

## Secrets in history

Deleting a committed secret does not remove it: history, clones, and forks keep it.

1. **Rotate or revoke the credential first.** That is the step that stops harm.
2. Assume it is already compromised.
3. Scan the full history, purge the blob, force-push the rewrite, and have every collaborator re-clone.
4. Prevent recurrence: a pre-commit scan, a CI scan on every push, and server-side push protection.
5. Keep secrets in a manager or environment, never in tracked files.

## Rewriting history

- Safe: local commits never pushed, and your own branch that nobody has pulled.
- Forbidden: anything on a shared branch, `main`, a release branch, or that someone may have pulled.
- Never rewrite history to hide a mistake. Report the failure instead.

## Provenance and AI

- The human is the author and is accountable for every change, including generated code.
- Disclose material AI assistance with the trailer the agent harness mandates. The opencode harness requires `Co-authored-by: <tool> <email>` (e.g. `Co-authored-by: Claude <noreply@anthropic.com>`), separated from the body by a blank line and placed at the very end of the commit message. When the harness mandates nothing, fall back to `Assisted-by: <tool>`.
- Never attribute sign-off to an AI.
- Never let a tool answer reviewers; the author explains and defends the change.
- Commit signing is not required. Use `Signed-off-by` only where the project requires a DCO.

## Agent guardrails

Hard rules for an automated agent.

- Never commit, push, create a branch, or open a pull request unless explicitly asked.
- Never use `--no-verify` or any flag that skips hooks, signing, or checks. Fix the underlying failure.
- Never force-push, amend, or reset published history.
- Never commit secrets or write secret-shaped values into tracked files.
- Never commit unrelated files, formatting-only churn, or debug output.
- Keep diffs small and reviewable (aim for ~200 changed lines); split or stack otherwise.
- Run the tests, linter, and build before proposing a commit; report the real output.

## Before done

- Show `git status` and the exact diff to be committed.
- State the commit message and the one logical change it names.
- Confirm no secrets, artifacts, or unrelated files are included.
