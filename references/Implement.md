# Implement.md

## Role of this file

This file is the execution runbook.

Treat:

- `Prompt.md` as the full product specification,
- `Plan.md` as the milestone-by-milestone source of truth,
- `Documentation.md` as the running lab notebook / project changelog.

## Startup protocol

At the start of every work session:

1. Read `Prompt.md`.
2. Read `Plan.md`.
3. Read `Documentation.md`.
4. Inspect the current repo tree.
5. Determine the next incomplete milestone.
6. Restate the immediate subgoal in `Documentation.md` before major edits if helpful.

## Execution loop

For each milestone:

1. Understand the acceptance criteria.
2. Implement the smallest complete slice that can satisfy them.
3. Keep code changes scoped to the current milestone.
4. Add or update tests with the implementation.
5. Run the milestone validation commands.
6. If validation fails, fix immediately.
7. Update `Documentation.md` with:
   - milestone completed,
   - files added/changed,
   - decisions made,
   - validation results,
   - next milestone.

## Scope discipline

Do not widen scope casually.

Specifically:

- do not add a web dashboard in v1,
- do not add automatic email/phone/payment/identity features,
- do not add unconstrained browser automation,
- do not add mandatory Redis/Kubernetes/distributed infra,
- do not build provider-specific hacks that break the common abstraction,
- do not collapse all agent logic into one giant class.

If a feature sounds nice but is not in `Prompt.md` or the current milestone, defer it and record it in `Documentation.md` under backlog.

## Provider implementation rules

### General
- All providers must implement the common provider interface.
- Providers must be selectable by role.
- Errors must be surfaced clearly.
- Mockable/testable interfaces are required.

### OpenAI
- Support local/user-managed Codex integration.
- Support OpenAI API integration.

### Gemini
- Support local/user-managed Gemini CLI integration.
- Support Gemini API integration.

### Claude Code
- Optional only.
- Experimental only.
- Disabled by default.
- Must be implemented as a local passthrough adapter to a user-managed local installation if present.
- Must not proxy Claude.ai consumer credentials.
- Must have warnings in docs and CLI output.

## File and sandbox rules

- Never read or write outside the configured sandbox roots.
- Respect per-role read/write restrictions when implemented.
- Use isolated run workspaces.
- Never mutate `references/` except for documentation notes or explicit operator-approved refresh scripts.
- Never treat external repos as importable runtime modules.

## License / copying rules

- Reference repos with custom or unclear licensing are **reference-only** unless explicitly approved otherwise.
- Reimplement ideas instead of copying code from restricted sources.
- If permissive code is copied, add attribution and update `THIRD_PARTY.md`.
- When unsure, do not copy; write a clean-room version.

## Coding style rules

- Python 3.11+
- type annotations throughout
- small, composable modules
- minimal dependencies
- docstrings for public interfaces
- descriptive names over clever names
- explicit config models
- prefer pure functions for transformations and small service objects for integrations

## State rules

- SQLite + files is the source of truth in v1.
- Make all state transitions explicit.
- Persist runs, tasks, events, decisions, metrics, and job metadata.
- A process crash should not erase run progress.

## Testing rules

Every milestone should leave the repo in a working state.

Add tests for:

- config and path policy
- state transitions
- provider registry behavior
- adapter dry-run/mock behavior
- benchmark import and manifest validation
- orchestration and task lifecycle
- scheduler policy enforcement
- channel message parsing and formatting

Prefer deterministic tests.
Avoid brittle tests that require live external services by default.

## Documentation rules

After each meaningful change:

- update `Documentation.md`,
- update or add docs in `docs/` when user-facing behavior changes,
- keep README examples accurate.

When a command changes, update docs in the same milestone.

## CLI design rules

The CLI is a first-class interface.

Requirements:

- clear help text
- sensible subcommands
- machine-readable output option if practical
- human-readable table/summary output by default
- non-zero exit codes on failure

## Agent design rules

- Implement role separation cleanly.
- Agents should operate on explicit task objects and artifacts.
- Do not rely on one endlessly growing transcript.
- Keep the Concierge lightweight and state-aware, not omniscient.
- The Director owns the global objective and phase transitions.

## Runner rules

### Local runner
- Use subprocess execution with captured stdout/stderr.
- Persist commands, exit codes, durations, and outputs.

### Slurm runner
- Submit with `sbatch`.
- Render scripts deterministically.
- Enforce quota policy before submission.
- Persist job IDs and transitions.
- Make resumption robust.

## Channel rules

- Slack and Discord are optional integrations.
- Channels are for operator control and visibility, not primary memory.
- All significant information must still land in files/SQLite.
- Never store secrets in git.

## Quality bar

Prefer a smaller, robust v1 over a sprawling fragile prototype.

When choosing between:

- completeness vs stability → choose stability,
- cleverness vs clarity → choose clarity,
- speed vs auditability → choose auditability for control/state code,
- hidden magic vs explicit files/config → choose explicit.

## When blocked

If blocked by missing secrets, missing external repos, or operator-only setup:

1. do not fake completion,
2. record the blocker in `Documentation.md`,
3. create the next best offline/testable slice,
4. clearly list what the operator must provide.

## What not to do

- Do not silently skip validations.
- Do not claim support for a provider that is not actually wired.
- Do not implement dangerous identity/payment features.
- Do not break the sandbox contract.
- Do not hide architectural changes from `Documentation.md`.

## Completion rule

A milestone is complete only when:

1. code is implemented,
2. tests/validations pass,
3. docs are updated,
4. `Documentation.md` records the result.
