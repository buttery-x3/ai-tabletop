# Agent instructions

AI Tabletop is a local-first virtual tabletop for human and AI controllers using
one authoritative rules engine. This repository currently contains documentation
only; the runtime, packages and application checks have not been scaffolded.

## Read and plan before implementation

Canonical specifications live in Linear:

- [Development workflow and working agreement](https://linear.app/flamehorn-games/document/03-ai-tabletop-development-workflow-and-agentsmd-requirements-26690832cbb7)
- [Architecture and observability](https://linear.app/flamehorn-games/document/02-ai-tabletop-architecture-and-observability-be0f781dd581)
- [Product and rules design](https://linear.app/flamehorn-games/document/01-ai-tabletop-product-and-rules-design-b89aab63bcbf)

1. Read the issue, these instructions, applicable local instructions and relevant
   design. Inspect actual files, Git state and available checks. If a required
   specification is inaccessible or truncated, request its relevant text in the
   coding conversation instead of guessing.
2. Present a proportionate implementation plan in that conversation: outcome,
   exclusions, approach, expected files and purposes, verification, dependencies,
   contract/configuration/migration changes, feature branch and `develop`
   integration. Do not request blanket permission to change any files.
3. **Stop and wait for explicit user acceptance in the same conversation.**
   Read-only investigation is allowed; implementation edits, commits, dependency
   installation and scaffolding must wait. Issue assignment, design approval and
   Linear status are not implementation approval. Never self-approve.
4. After acceptance, implement, debug, verify, push the feature, integrate and push
   `develop`, then hand off without another routine approval. Stop for a revised
   plan only if the outcome requires material expansion of the approved scope
   (for example, an unapproved dependency, migration, public contract or subsystem).
   Preserve unrelated work and report genuine external blockers.
5. When resuming, recover the approved plan from the conversation or ask for the
   missing context. Do not infer acceptance from issue status.

Approval belongs in the conversation. Do not require a Linear plan attachment,
plan hash/version service, GitHub approval gate or separate approval application.
Do not weaken this agreement during an unrelated issue; changing it requires its
own explicitly approved scope.

## Branches and workspaces

- Use one `feature/FLAME-<number>-short-description` branch per issue, starting
  from the latest appropriate `develop`. Make no feature edits on shared branches.
  Concurrent agents need separate worktrees/directories and feature branches;
  never switch branches beneath another worker.
- Push the verified feature branch before integration. Serialize integration in
  an agent-owned checkout/worktree: fetch, account for current `develop`, merge,
  recheck the combined result and push normally. If the remote advances, update
  safely and revalidate. Never force-push shared branches or overwrite substantive
  conflicting work; surface conflicting design decisions. A PR is optional.
- The user's playtesting checkout, processes and data are not an agent workspace.
  Without explicit permission, do not edit/reset it, switch its branch, migrate
  or delete its data, or restart its processes. Never use it to merge.
- `main` is the accepted version. Promote only after explicit user acceptance of
  an identified tested commit/build, never an untested newer `develop` tip.
- Bootstrap exception for FLAME-149 only: after rechecking the remote and obtaining
  plan approval, create missing `develop` from the appropriate current `main`,
  then work on `feature/FLAME-149-development-workflow`. Leave `main` unchanged.

## Runtime isolation (requirements for the future scaffold)

| Environment | UI port | Local server port |
| --- | --- | --- |
| Agent development | 4310 | 4311 |
| User playtesting | 4320 | 4321 |

Check availability; these are reserved defaults, not a claim that ports are free.
Bind locally by default and connect each UI to its matching backend. Fail clearly
on an occupied assigned port; never silently move ports or kill unrelated
processes. Additional workers, graph viewers and inspectors need explicit
non-conflicting allocations; the playtesting pair stays reserved for the user.

Separate saves, databases, logs, runtime configuration, credentials and relevant
browser/session state per environment and concurrent worker. Port separation
alone is insufficient. Use repository-relative or explicitly configured paths,
never guessed absolute user paths. See [the development guide](docs/development.md).

## Implementation boundaries

The following are intended responsibilities, not existing packages:

- Use strict TypeScript and shared TypeBox/JSON Schema contracts for typed,
  runtime-validated external interfaces. Commands request actions; events record
  committed facts. Validate identity and permissions using trusted context.
- Contracts define public schemas/types and import no implementations. Use public
  package entrypoints; avoid deep imports, forbidden dependencies and cycles.
- Rules own mechanics and state transitions, with explicit random/time inputs.
  Keep network calls, UI state, model SDKs and database access out of the pure core.
- Session coordinates commands and pending decisions; rules retain authority over
  legality, timing and action economy. Preserve resumable decisions and deduplicate
  retries through one authorized commit path.
- AI receives permitted observations and proposes commands; it cannot import
  persistence or mutation internals. Persistence stores records, not adjudications.
- Server composes modules, assigns permissions and owns execution/commit boundaries.
  Svelte consumes public observations and submits commands. Keep authoritative
  calculations out of components, HTTP glue and model prompts; display derived
  results from the engine. Closing the browser must not discard server-owned state.
- Target **SRD 5.2.1** faithfully. Pin content versions, retain source provenance and
  required attribution, and add source-linked tests for mechanics, invalid paths
  and meaningful interactions. Maintain implementation-linked coverage as rules
  arrive; distinguish content presence from executable, verified behaviour.
  State unsupported mechanics explicitly; never invent simplified substitute
  rules or disguise missing implementation as DM discretion.
- Use scripted controllers/fixtures and fixed or recorded random inputs for
  deterministic rules tests. Live-model/paid-network tests need authorization.
  Filter observations per participant; keep privileged state and private agent
  observations out of player contexts. Redact credentials from logs/exports.

## Verify and hand off

Run applicable existing local type, lint, boundary, unit, integration and UI checks.
Inspect the final diff and links/paths; run `git diff --check` against the changed
range. Report exact commands/results, unavailable or unrun checks and limitations.
There are currently no application checks or launch commands; never describe
absent tests as passing. Do not weaken tests or acceptance criteria to obtain a pass.

Keep changes within the approved issue: no unrelated redesigns, reformatting,
dependency changes or telemetry infrastructure. No GitHub Actions, substitute
hosted CI, Nx Cloud, remote approval machinery or compulsory hooks. FLAME-149
must not change remote settings, install the stack or scaffold the application.
Do not commit credentials, real local configuration, saves, databases or sensitive
logs. Review third-party licences before imports; a repository code licence needs
the user's explicit decision.

Hand off changed files/scope, deviations, verification evidence, remote feature
branch and commit, exact integrated `develop` commit, and review/testing steps.
Keep long-form specifications in Linear instead of duplicating them here. Use
existing statuses: `In Review` when delivered to `develop` for review/testing;
`Done` only when acceptance criteria are satisfied. A push does not imply user
playtesting or acceptance for promotion to `main`.
