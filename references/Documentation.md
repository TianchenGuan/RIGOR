# Documentation.md

## Purpose

This file is the living lab notebook for implementation.
It records decisions, status, completed milestones, blockers, and next steps.

## Project snapshot

- Working name: `research-os`
- Status: bootstrap / pre-implementation
- Objective: implement a publishable multi-agent research operating system with durable run memory, benchmark-pack ingestion, provider flexibility, cluster-aware execution, and operator messaging.

## Initial architecture decisions

### Accepted decisions

1. **Language/runtime**
   - Python 3.11+
   - `uv` for dependency and environment management

2. **State model**
   - SQLite + files by default
   - optional Redis only after MVP

3. **Execution model**
   - long-horizon file stack per run:
     - `Prompt.md`
     - `Plan.md`
     - `Implement.md`
     - `Documentation.md`

4. **Provider model**
   - dual-mode support:
     - personal / subscription mode
     - lab / API mode

5. **Provider priorities for MVP**
   - OpenAI Codex local adapter
   - OpenAI API adapter
   - Gemini CLI adapter
   - Gemini API adapter
   - optional local Claude Code passthrough adapter

6. **Claude policy for MVP**
   - do not proxy Claude.ai login
   - if supported, only as user-managed local passthrough
   - disabled by default

7. **Channel priorities for MVP**
   - CLI first
   - Slack next
   - Discord next

8. **Agent roles**
   - Director
   - PM/Triage
   - Librarian
   - Planner
   - Builder
   - Runner
   - Analyst
   - Writer
   - Reviewer
   - Concierge

9. **Runner priorities**
   - local subprocess runner first
   - Slurm `sbatch` runner second

10. **External references**
    - keep under `references/`
    - treat as read-only inputs
    - keep legal/runtime boundary clean

11. **Documentation bar**
    - repo should be understandable to a new GitHub visitor without private context

## Sandbox / cluster decisions

### Accepted

- default sandbox is repo-only
- operator can expand sandbox roots explicitly
- target personal cluster use should support:
  - workspace under user-controlled home/personal directory
  - scratch/temporary directory under xtmp or equivalent
- no access to other user folders
- scheduler integration should use `sbatch`
- resource policy should be configurable

### Initial quota targets to encode as config defaults/examples

- max 2 nodes of the 4×RTX PRO 6000 class
- max 25 GPUs total per project
- max 300 CPUs per project
- max 5T RAM per project

These are examples/defaults for the target operator environment, not global invariants.

## Benchmark decisions

### Accepted

- build a generic benchmark-pack format
- support current course projects and archived course-project folders as first-class input examples
- include at least one toy benchmark in-repo for smoke testing
- keep datasets optional/not vendored by default

### Notes

The benchmark importer should normalize raw project folders rather than assuming a single fixed benchmark shape.

## Repo philosophy decisions

### Accepted

- copy patterns first, code second
- prefer clean-room implementation over entangling multiple agent repos
- keep dependencies lightweight
- make the CLI and docs good enough for public release

## Open questions

1. Final public name of the repo/package.
2. Exact benchmark chosen for the first end-to-end demonstration.
3. Whether `Librarian` ships in MVP as a concrete role or as Planner support utilities.
4. Whether reviewer rubrics should be YAML-driven from the beginning.
5. Whether local CLI adapters should use structured output parsing or file-based handoff wrappers.

These questions should not block early milestones unless implementation depends on them.

## Milestone status

- [ ] Milestone 0 — Repo scaffold and engineering baseline
- [ ] Milestone 1 — Reference catalog and legal boundary
- [ ] Milestone 2 — Config, paths, sandbox, and SQLite state
- [ ] Milestone 3 — Run control plane and four-file stack
- [ ] Milestone 4 — Provider abstraction and provider registry
- [ ] Milestone 5 — OpenAI Codex local adapter + OpenAI API adapter
- [ ] Milestone 6 — Gemini local adapter + Gemini API adapter
- [ ] Milestone 7 — Optional Claude Code local passthrough adapter
- [ ] Milestone 8 — Benchmark-pack schema and importer
- [ ] Milestone 9 — Core agent roles and task orchestration
- [ ] Milestone 10 — Builder/Runner/Analyst verified loop
- [ ] Milestone 11 — Writer/Reviewer and artifact packaging
- [ ] Milestone 12 — Slack and Discord channels
- [ ] Milestone 13 — Slurm runner and quota policy
- [ ] Milestone 14 — Example benchmark and end-to-end demo run
- [ ] Milestone 15 — Publishability pass

## Blockers

None recorded yet.

## Backlog / deferred ideas

- Redis-backed coordination
- Anthropic API adapter
- richer multi-run dashboards
- advanced history compaction and summarization
- operator approval workflows for high-risk actions
- benchmark leaderboards / experiment comparison views
- optional web UI

## Operator dependencies

The operator is expected to provide:

- the external reference repos under `references/`
- benchmark/course-project input folders
- local provider tool installation and sign-in where applicable
- environment variables / API keys for lab mode
- Slack/Discord app tokens if channel integrations are desired
- cluster access and Slurm environment if scheduler mode is desired

## Working rules for future updates

When a milestone is completed, append:

- date/time if available
- summary of work completed
- files touched
- validation commands run
- results
- remaining issues
- next milestone

When blocked, append:

- blocker description
- exact missing dependency or credential
- what can continue in parallel

## Change log

### Bootstrap entry
- Created the initial implementation notebook.
- Recorded project scope, key decisions, milestone plan, and cluster policy targets.
- No code changes yet.
