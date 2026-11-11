# Verla

Save, share and decide together what film or series to watch.

Verla is a social platform for organising films and series: a personal library, shared watchlists, groups and **Jams**, which are group sessions for deciding what to watch next. It is built in public as a Go modular monolith.

> **Status: H0, walking skeleton, in progress.** There are no product features yet. The current goal is for one authenticated request to travel web → API → database, start with `task up`, and pass CI. Expect missing pieces.

## Stack

| Layer | Choice |
|---|---|
| API | Go, `net/http` + [chi](https://go-chi.io/) |
| Database | PostgreSQL 18, accessed with [pgx](https://github.com/jackc/pgx), [sqlc](https://docs.sqlc.dev/) and [goose](https://github.com/pressly/goose); no ORM |
| Contract | OpenAPI (`api/openapi.yaml`), contract-first: [oapi-codegen](https://github.com/oapi-codegen/oapi-codegen) for Go and openapi-typescript for the web client |
| Web | Svelte 5 + SvelteKit + TypeScript |
| Local infra | Docker Compose, [Task](https://taskfile.dev/) |
| Deploy | Docker Compose + Caddy on a single machine |

## Architecture in one paragraph

Verla is a **modular monolith**: one deployable binary, split into modules (`identity`, `catalog`, `library`, `watchlists`, `jams`, …) with strict boundaries. Each module exposes a small public contract and keeps everything else under Go's `internal/`, so the compiler, not a convention, prevents one module from importing another's internals. Each module owns its own PostgreSQL schema. No module queries another module's tables. Architectural decisions are recorded as ADRs in [`docs/adr/`](docs/adr/).

## Requirements

- [Go](https://go.dev/dl/) (version in `apps/api/go.mod`)
- [Docker](https://docs.docker.com/get-docker/) with Compose v2
- [Task](https://taskfile.dev/installation/)
- Node.js + pnpm (for `apps/web`)

## Getting started

```sh
cp .env.example .env   # local credentials, never committed
task up                # start local infrastructure and wait until it is healthy
task --list            # see every available task
task down              # stop everything
```

## Tasks

| Task | What it does |
|---|---|
| `task up` | Starts the local stack and waits until every service is healthy |
| `task down` | Stops the local stack (data is kept) |
| `task reset` | Stops the stack and deletes its volumes (asks first) |
| `task ps` / `task logs` | Service state and health / follow logs (`task logs -- postgres`) |
| `task db:psql` | Opens a `psql` shell inside the database container |
| `task generate` | Regenerates code from the OpenAPI contract and SQL (not implemented yet) |
| `task test` | Runs the test suites (not implemented yet) |
| `task lint` | Runs linters (not implemented yet) |

Tasks that are not implemented yet fail on purpose, so CI can never report a false green.

## Repository layout

```text
apps/
  api/            Go API (cmd/verla + internal/platform + internal/modules)
  web/            SvelteKit app
api/              OpenAPI contract
packages/
  api-client/     Generated TypeScript client
migrations/       SQL migrations, one folder per module
deployments/
  compose.yaml    Root of the local stack: project name + includes
  postgres/       One self-contained Compose fragment per service
docs/
  adr/            Architecture decision records
```

Folders are created when they get their first file, so some of the folders above may not exist yet.

Each service gets its own folder under `deployments/` with a self-contained `compose.yaml`, and the root file pulls them together with [`include`](https://docs.docker.com/reference/compose-file/include/). The result is one Compose project (`verla`) with one network, so `depends_on` works across services. A fragment can still be validated on its own with `docker compose -f deployments/<service>/compose.yaml config`. Dockerfiles live next to the app they build (`apps/api`, `apps/web`).

## Roadmap

| Milestone | Goal |
|---|---|
| **H0** | Walking skeleton: web → API → DB, CI and minimal deploy |
| H1 | Usable foundations: accounts, catalogue via TMDB, personal library |
| H2 | Shared watchlists with permissions, transactional outbox |
| H3 | Jams: group decisions, messaging, background jobs |
| H4 | Persistent groups and chat |
| H5 | Android app |

## Attribution

Film and series data will come from [TMDB](https://www.themoviedb.org/). This product uses the TMDB API but is not endorsed or certified by TMDB.
