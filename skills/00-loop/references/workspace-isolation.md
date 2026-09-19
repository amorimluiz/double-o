# Workspace and Dependency Isolation

Procedure behind the `workspace` step. A worktree isolates files and branches; it does not isolate processes. Ports, containers, volumes, and databases stay shared unless this reference provisions them per worktree. Run it before the first execution iteration.

## When isolation applies

Provision isolation when **both** hold:

1. The run executes in a worktree (not the primary checkout).
2. The tasks touch a **stateful shared dependency** — a database, cache, broker, object store, or search cluster that outlives a process — especially when a task mutates it (migration, seed, fixture load, destructive test).

A run that only reads files or calls external APIs needs no isolation. A run without a worktree uses the project's normal dev dependencies and must not be parallelized with another run; warn and proceed only on the user's confirmation.

## Naming

Derive every identifier from the slug so the same branch resolves to the same names on any machine. Lowercase and sanitize to a Docker-safe token.

| Artifact | Pattern | Example (`slug: photo-albums`, repo `double-o`) |
| --- | --- | --- |
| Worktree path | `.worktrees/<slug>` | `.worktrees/photo-albums` |
| Branch | `<type>/<slug>` per the git rules | `feat/photo-albums` |
| Compose project | `<repo>-<slug>` | `double-o-photo-albums` |
| Port block | `20000 + 10 * (hash(slug) mod 2000)` | `20000`–`39990`, ten ports |
| Database | `<base>_<slug>` | `app_photo_albums` |

Assign each stateful service one port inside the block by ordinal, derived from its default host port so the mapping is stable:

```text
port(service) = port_block_base + ordinal(service)
```

## Provisioning

1. **Create the worktree** from the repo root, after ensuring the target path and branch are free:

   ```bash
   git worktree add .worktrees/<slug> -b <type>/<slug>
   ```

   Serialize creation: never run two `git worktree add` concurrently against the same repo. Never use `git stash` across worktrees — `refs/stash` is shared and will apply the wrong work. Use commits on the run branch.

2. **Carry the run directory.** `.sdd/` is ignored, so it is absent from the new checkout. Copy it in and continue from there:

   ```bash
   cp -r .sdd/<slug> .worktrees/<slug>/.sdd/<slug>
   ```

   From here the run's working directory is the worktree; all paths in the task files resolve relative to it.

3. **Detect the stateful services.** Find the Compose file (`compose.yaml`, `docker-compose.yml`, or the project's variant) and the primary database name and ports from the project's env files. Classify each service as stateful (postgres, mysql/mariadb, redis, rabbitmq, kafka, minio, elasticsearch, …).

4. **Write the worktree env override.** Append a managed block to the gitignored `.env.local` (or the project's equivalent local override) — never to a tracked file:

   ```dotenv
   # >>> worktree isolation (managed) <<<
   COMPOSE_PROJECT_NAME=<repo>-<slug>
   <SERVICE>_HOST_PORT=<port_block_base + ordinal>
   DATABASE_URL=...://<user>:<pass>@<host>:<db_port>/<base>_<slug>
   # <<< worktree isolation (managed) <<<
   ```

   Keep the driver, host, and credentials from the project's existing value; swap only the port and the database name.

5. **Clone the database from a template.** Point the worktree at a new database seeded from the primary dev database, not an empty one, so migrations and seeds run against realistic shape:

   ```bash
   createdb --template <base> <base>_<slug>           # Postgres
   docker compose -p <repo>-<slug> exec -T db createdb -T <base> <base>_<slug>   # containerized Postgres
   ```

   For non-Postgres engines, adapt: dump/restore for MySQL/MariaDB, a separate database/index for others. When cloning is impractical, create the database empty and run the project's migrate + seed commands.

6. **Bring up the isolated stack.**

   ```bash
   docker compose --env-file .env.local -p <repo>-<slug> up -d
   ```

   Every compose, migration, seed, and test command in the run must use this project name and env file. Verify the primary checkout's stack is untouched.

## Teardown

Gate teardown at `done` or on request; never auto-delete. Resolve the project and path explicitly — a bare `docker compose down -v` can fall back to the directory-derived project name and destroy the primary checkout's data.

```bash
docker compose -p <repo>-<slug> down -v
dropdb <base>_<slug>
git worktree remove .worktrees/<slug>
```

Keep the branch; removing the worktree does not remove committed work. Report the branch and the removed dependencies.

## Failure modes to design around

- **Host-run scripts bypass the override.** A migration or seed run outside the container defaults to the hard-coded primary port and writes into the primary database while reporting success. Export the override into the shell (`set -a; . ./.env.local; set +a`) or pass `--env-file`/`DATABASE_URL` explicitly to every command.
- **Teardown hits the primary stack.** Always pass `-p <project>`; never rely on the compose file's default project name after the worktree path changes.
- **Shared `refs/stash`.** Never stash across worktrees; commit on the run branch instead.
- **Disk pressure.** Each worktree duplicates tracked files. Remove worktrees when their run closes.
- **Orphans.** After a crash, run `git worktree prune` and remove any leftover `.worktrees/<slug>` and compose project before re-provisioning.

## Recording

Write the resolved values into `state.yml` under `workspace` (see `state-schema.md`) and list the isolated dependencies in `workspace.isolated`. The execution loop reads this block to route commands and to tear down.
