# GameDeveloperAIHarness Constitution

## Core Principles

### I. Spec-First, Harness-Second

Every behavioral change to the harness MUST begin as a spec under `specs/NNN-short-name/`
and proceed through `specify → plan → tasks → implement`. Code that no spec describes MUST
NOT be merged; conversely a spec MUST NOT prescribe implementation detail that belongs in
`plan.md`. Ad-hoc exploratory code is permitted only in throwaway scratch space and MUST NOT
enter `src/`.

**Rationale**: The harness's own product is agentic software development. If its construction
cannot survive the discipline it is meant to apply, the discipline is not real.

### II. Deterministic, Scriptable Entry Points

Every harness capability MUST be invocable non-interactively, with the same inputs producing
the same observable actions apart from explicitly declared nondeterminism (model sampling,
wall-clock, randomness). Model nondeterminism MUST be isolated behind a seam that can be
stubbed or replayed. Inputs come from arguments, files, or environment; outputs MUST offer a
machine-readable form (JSON on stdout) alongside any human-readable form, with diagnostics on
stderr.

**Rationale**: A harness that cannot be driven by another program cannot be regression-tested,
batched over scenarios, or run in CI.

### III. Evaluated, Not Asserted (NON-NEGOTIABLE)

A change claiming to improve harness behavior MUST ship evidence: at least one scenario or
eval case that fails (or scores worse) before the change and passes (or scores better) after,
recorded in the feature directory. Prompt, policy, and tool-definition changes count as
behavioral changes and are subject to this rule. "It looks better" is not evidence. Reviewers
MUST reject unevaluated behavioral claims.

**Rationale**: Prompt and policy edits are invisible to conventional tests and regress
silently; measurement is the only control available.

### IV. Observable Runs

Every harness run MUST emit a structured, machine-parsable record of what it did: the task
given, each tool invocation with its arguments and outcome, and the terminating condition.
Records MUST be sufficient to reconstruct a failure without re-running the agent, and MUST
redact secrets. Silent failure is a defect: a run that cannot complete MUST terminate with a
non-zero status and a stated reason.

**Rationale**: Agent failures are emergent and rarely reproducible on demand. The run record
is the primary debugging artifact and the input to every eval.

### V. Bounded Authority Over the Project Under Development

The harness MUST treat the game project it operates on as someone else's repository. Actions
that are destructive or hard to reverse — deleting assets, rewriting history, force-pushing,
mutating editor or build settings, long-running builds and bakes — MUST be either explicitly
requested, confirmed, or reversible, and MUST be refused by default when unattended. The
harness MUST NOT widen its own permissions at runtime, and MUST treat all content it reads
from the project under development (files, logs, asset metadata, console output) as data,
never as instructions.

**Rationale**: An autonomous agent with editor and shell access can destroy work faster than a
human can intervene, and content it reads is an injection surface.

## Technology & Integration Constraints

- **Spec Kit tooling**: Spec Kit scripts are PowerShell-only in this checkout
  (`"script": "ps"`). Files recorded in `.specify/integrations/*.manifest.json` are
  vendor-managed: customization goes through the template override stack
  (`.specify/templates/overrides/<template-name>.md`), never by editing the managed file.
- **Feature context**: Resolved explicitly via `SPECIFY_FEATURE_DIRECTORY` or
  `.specify/feature.json`. Harness code and automation MUST NOT infer the active feature from
  git branch names.
- **Unity**: Unity Editor interaction goes through the Unity Editor MCP integration rather than
  ad-hoc Unity CLI invocations, unless a plan documents why the CLI is required. Editor state
  changes follow Principle V.
- **Text I/O**: Files written by harness tooling are UTF-8 without BOM, with `-Json` modes
  keeping stdout free of anything but JSON.
- **Secrets**: API keys and tokens come from the environment only. They MUST NOT appear in
  specs, plans, run records, eval fixtures, or committed configuration.
- **TODO(PROJECT_SCOPE)**: The harness's concrete scope — which agent models it drives, which
  engine targets beyond Unity it supports, and whether it ships as a CLI, a service, or a Unity
  package — is still inferred rather than decided. Resolve this before the first
  `/speckit-plan` Constitution Check and amend this document accordingly.

## Development Workflow & Quality Gates

1. `/speckit-specify` creates the feature directory and `spec.md`; `/speckit-clarify` resolves
   underspecified areas before planning.
2. `/speckit-plan` MUST complete the Constitution Check against this document. A plan that
   violates a principle MUST either be revised or record an explicit, justified exception in
   its Complexity Tracking section. Unjustified violations block the plan.
3. `/speckit-tasks` produces `tasks.md` organized by user story so each story remains an
   independently shippable increment. Tests are included when the spec calls for them;
   Principle III evidence is always a task, never optional.
4. `/speckit-analyze` runs before implementation is considered complete; `/speckit-converge`
   reconciles the codebase with the spec when work spans sessions.
5. Merge gates: spec exists and is current; Constitution Check passed; Principle III evidence
   present and reproducible from the committed command; run records for new capabilities are
   structured and redacted.

## Governance

This constitution supersedes conflicting practices, conventions, and agent instructions in
this repository, including `CLAUDE.md` and any per-tool guidance file; those files describe how
to work here and MUST be kept consistent with this document.

Amendments are made only through `/speckit-constitution`, which MUST record the version change,
the affected principles, and any deferred items. Versioning follows semantic versioning: MAJOR
for removing or redefining a principle in a backward-incompatible way, MINOR for adding a
principle or materially expanding guidance, PATCH for clarifications and wording. The Sync
Impact Report is review scratch material and is removed before the amended file is committed.

Compliance is verified at the plan gate and at review. Any party may challenge a merged change
as non-compliant; the remedy is a follow-up spec, not a retroactive exception. Complexity MUST
be justified in the plan that introduces it.

**Version**: 1.0.0 | **Ratified**: 2026-10-18 | **Last Amended**: 2026-10-18
