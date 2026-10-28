# Local development

Start with [AGENTS.md](../AGENTS.md). This is a documentation-only repository:
Git inspection and documentation review work today; application setup, launchers,
tests and generated architecture graphs do not exist yet. Git is sufficient for
the current work. Docker and WSL are not prerequisites.

Canonical design:

- [Development workflow and AGENTS.md requirements](https://linear.app/flamehorn-games/document/03-ai-tabletop-development-workflow-and-agentsmd-requirements-26690832cbb7)
- [Architecture and observability](https://linear.app/flamehorn-games/document/02-ai-tabletop-architecture-and-observability-be0f781dd581)
- [Product and rules design](https://linear.app/flamehorn-games/document/01-ai-tabletop-product-and-rules-design-b89aab63bcbf)

## Workspaces and branches

Keep an **agent workspace** for implementation, integration and development
processes, and a separate **user playtesting clone** on `develop`. Choose locations
explicitly; no absolute directory on the user's machine is assumed. The user
updates their testing clone when ready. Agents must not edit/reset it, change its
branch, migrate/delete its data, restart its processes or use it for integration
without explicit permission. Preserve unrelated user and agent work everywhere.

Concurrent agents use separate worktrees/directories and feature branches. Inspect
`git worktree list` before allocating a checkout; never change another worker's
branch. Serialize integration into `develop` and give runtime processes distinct
ports and data locations. A Git worktree alone does not isolate processes or data.

| Branch | Purpose |
| --- | --- |
| `feature/FLAME-<number>-short-description` | One issue's implementation and local verification |
| `develop` | Shared integration and user playtesting |
| `main` | Explicitly accepted, identified tested version |

For each issue:

1. Read the issue/design/instructions and inspect the repository. Present a plan
   listing files, scope/exclusions, dependencies/contracts/configuration/migrations,
   verification and feature-to-`develop` integration in the coding conversation.
   **Wait for the user's explicit acceptance before implementation.**
2. Fetch and start the feature from the latest appropriate `develop` in an
   agent-owned workspace. Implement and verify within the accepted plan.
3. Commit and push the verified feature branch to `origin`.
4. In an agent-owned checkout/worktree, fetch again and bring `develop` up to date
   safely. Integrate the feature, inspect the combined diff and rerun affected
   checks, then push `develop` normally. If it advances remotely, update and
   revalidate; never force-push shared branches. Surface substantive design
   conflicts instead of overwriting other work. A local merge is sufficient;
   a PR is optional and adds no mandatory approval gate.
5. Hand off the exact feature and integrated `develop` commits with verification
   and testing instructions. Do not alter `main`: promotion requires explicit
   user acceptance of an identified tested commit/build, not a newer tip.

FLAME-149 bootstrap only: if a remote recheck confirms `develop` is missing, its
approved plan may create it from current appropriate `main` and implement on
`feature/FLAME-149-development-workflow`. Push the feature before integrating and
pushing `develop`. Future issues start from `develop`.

Approval stays in the coding conversation, not a Linear attachment, plan-hash service
or GitHub gate. No GitHub Actions, hosted CI substitute, Nx Cloud, compulsory hooks
or remote approval machinery is required. FLAME-149 makes no remote settings
changes. Changing the agreement itself needs explicitly approved scope.

| Scenario | Required behaviour |
| --- | --- |
| Fresh issue or assignment | Investigate, present the file-aware plan, then stop for explicit acceptance. |
| Approved issue, same scope | Implement, verify, push feature, integrate/push `develop` and hand off without another routine approval. |
| Material scope expansion | Stop, explain the smallest revised plan and wait for acceptance before expanding. |

Routine debugging and equivalent choices within scope are autonomous. Recover
the approved plan when resuming; issue status alone cannot supply approval.

## Available commands and checks

Run these from the agent repository root in PowerShell or another Git-capable
shell. They inspect the current checkout without starting an application:

```sh
git status --short --branch
git branch -avv
git worktree list
git diff
git diff --check
git diff --cached --check
```

Inspect new files too; ordinary `git diff` does not display untracked files.
After committing, check the actual changed range, not just a clean working tree.
For the bootstrap, whose base remains on `origin/main`:

```sh
git diff --check origin/main...HEAD
git diff --text --check origin/main...HEAD
git diff --text origin/main...HEAD -- AGENTS.md README.md docs/development.md .gitignore
```

The original bootstrap README used UTF-16. `--text` includes its conversion to
UTF-8 in diff review and whitespace checks; also read the resulting file normally.

For later work, use the recorded feature base and the integration base/commit to
review the appropriate ranges. Check relative Markdown targets, canonical links,
ignore patterns and the three workflow scenarios above. Run applicable existing
checks; there are currently no type, lint, architecture, unit, integration or UI
test commands. Do not install tooling just to claim an empty suite or graph ran.

Once `origin/develop` exists and a later issue's plan is approved, this Git example
can allocate a separate worker in an unused repository-relative directory. Replace
the example issue and directory before running; do not reuse an occupied checkout:

```sh
git fetch origin
git worktree add -b feature/FLAME-123-short-description .worktrees/flame-123 origin/develop
```

Proceed only if each command succeeds. Keep `.worktrees/` local and do not share a
running worker's checkout. No extra worker is required for a single-agent issue.

## Planned scaffold requirements (not available commands)

The selected stack is strict TypeScript, Svelte 5/Vite, Node.js/Fastify, pnpm/Nx
and TypeBox/JSON Schema contracts. SQLite/Drizzle will provide persistence;
Vitest/Playwright will provide tests. XState v5 is intended selectively for session
lifecycle inspection. Pin compatible versions and verify integration support in
the approved scaffold issue; none of these dependencies is installed by M0.

Keep public contracts independent; rules own pure mechanics, session coordinates
decisions, AI proposes commands from permitted observations, persistence stores
records, server composes and commits, and UI presents public results. See
[AGENTS.md](../AGENTS.md) for boundaries and SRD provenance/test requirements.

| Environment | UI | Local server |
| --- | --- | --- |
| Agent development | 4310 | 4311 |
| User playtesting | 4320 | 4321 |

These are reserved defaults; port availability has not been verified. The future
frontend and server must bind locally by default, check assigned ports and fail
clearly on conflicts. Never silently choose another port or kill an unrelated
process. Each UI must target its matching backend. Additional simultaneous
workers, graph viewers and inspectors need explicit non-conflicting allocations;
do not reuse the user's playtest pair or assume all workers can use 4310/4311.

Separate runtime configuration, saves, SQLite files, logs, credentials and relevant
session state for each environment/worker. Repository-relative `.local/dev/`,
`.local/playtest/` and `.local/workers/<worker>/` are suggested local locations,
not existing configuration or implemented defaults; explicit external paths are
also possible. Worktrees must not resolve these to shared mutable files by accident.
Port separation alone does not isolate browser/session state. Use separate browser
profiles for development and playtesting, and verify cookie/session/provider
configuration when the runtime is implemented.

The scaffold must provide separate development/playtest launch commands and a
visible environment, branch and commit indicator. Reserve `test` for automated
checks, not user playtesting. Keep those checks local: type/lint/boundary checks,
rules/unit/integration tests, relevant UI tests and a graph of the actual workspace.
Add file-level inspection only when useful; hosted services are unnecessary.

## Local files and testing handoff

[.gitignore](../.gitignore) excludes local environment files (while allowing
secret-free `.env.example` and `.env.*.example`), credential directories/key files,
local runtime data/databases/logs, worker checkouts and common generated output.
Keep runtime files in designated ignored locations and inspect staged content;
ignore rules are not a credential scanner and do not protect already tracked files.
Do not commit real credentials/configuration, sensitive observations, saves or
databases. Source, lockfiles and intentional fixtures belong in version control.
Do not add a repository code licence without the user's explicit decision.

Every handoff gives changed files/scope, deviations, exact verification commands
and results, unavailable/unrun checks, remote feature branch and commit, integrated
`develop` commit and review/testing steps. Use Linear's existing `In Review` when
ready for review/testing and `Done` only when acceptance criteria are satisfied.
A push is not evidence of playtesting or permission to promote `main`.

The user can review the handed-off commits remotely or, when ready and with a
clean playtesting clone already on `develop`, run these themselves:

```sh
git pull --ff-only origin develop
git rev-parse HEAD
```

Compare the result with the handed-off commit; a newer tip is not automatically
covered by that verification or acceptance. Agents do not update the user's clone
on their behalf without permission. For M0, review `AGENTS.md`, this guide, README
and `.gitignore`; there is no application to launch or playtest yet.
