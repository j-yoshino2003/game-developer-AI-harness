# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A workspace for improving a game-developer AI harness (`GameDeveloperAIHarness`). It currently
contains **no application source code and no commits** — only a freshly initialized
[GitHub Spec Kit](https://github.com/github/spec-kit) scaffold (v1.2.1, `claude` integration,
PowerShell scripts). Everything here is spec-driven-development (SDD) tooling: the product itself
is still to be specified and built.

Expect the first real work to be either (a) filling in the constitution and the first feature spec,
or (b) creating the harness source tree under `src/`/`tests/` as a plan dictates. There is no
build, lint, or test command yet — do not invent one; add it as part of the feature that needs it.

## Spec-driven development workflow

Work flows through numbered feature directories under `specs/` (not yet created):

```
specs/NNN-short-name/
  spec.md        # what & why (speckit-specify)
  plan.md        # technical approach (speckit-plan)
  tasks.md       # dependency-ordered task list (speckit-tasks)
  research.md, data-model.md, quickstart.md, contracts/   # optional plan outputs
  checklists/requirements.md                              # spec-quality checklist
```

The canonical cycle is `specify → plan → tasks → implement`, with review gates between each
(see `.specify/workflows/speckit/workflow.yml`). `speckit-clarify` runs between specify and plan;
`speckit-analyze` and `speckit-converge` run after tasks exist.

### Running a stage

The stage instructions live in `.claude/skills/<name>/SKILL.md`, so each stage is invocable as
`/speckit-<name>`. Follow the `SKILL.md` literally — each one is a detailed procedure, not a
summary. Available:
`speckit-constitution`, `speckit-specify`, `speckit-clarify`, `speckit-plan`, `speckit-tasks`,
`speckit-implement`, `speckit-analyze`, `speckit-checklist`, `speckit-converge`,
`speckit-taskstoissues` (deprecated).

Skills shell out to the PowerShell helpers in `.specify/scripts/powershell/`, always run from the
repo root and almost always with `-Json`:

```bash
pwsh .specify/scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
```

| Script | Used by | Purpose |
| --- | --- | --- |
| `create-new-feature.ps1 <description>` | specify | allocate the next `specs/NNN-…` directory (`-ShortName`, `-Number`, `-Timestamp`, `-DryRun`) |
| `setup-plan.ps1 -Json` | plan | create `plan.md` from the template; returns `FEATURE_SPEC`/`IMPL_PLAN`/`FEATURE_DIR`/`BRANCH` |
| `setup-tasks.ps1 -Json` | tasks | returns `TASKS_TEMPLATE_CONTENT` + `AVAILABLE_DOCS`; requires `plan.md` and `spec.md` |
| `check-prerequisites.ps1 -Json [-RequireSpec] [-RequireTasks] [-IncludeTasks] [-PathsOnly] [-Template NAME]` | clarify, analyze, implement, checklist, converge | validate stage prerequisites and resolve paths |
| `resolve-template.ps1 <name>` | — | resolve one template through the override stack |

All of them `. common.ps1`, which holds the shared logic worth knowing before touching a script:
feature resolution, template override resolution (`Resolve-Template`/`Resolve-TemplateContent`),
and `Format-SpecKitCommand`.

### Feature context is explicit, not git-derived

There is **no branch-name magic**. The "current feature" is resolved in this order:

1. `SPECIFY_FEATURE_DIRECTORY` env var (wins; also gets persisted),
2. `.specify/feature.json`'s `feature_directory` key,
3. error.

`Get-CurrentBranch` reads only `SPECIFY_FEATURE`, falling back to the feature directory's
basename — so `BRANCH` in script output is a feature identifier, not necessarily a real git branch.
`.specify/feature.json` is gitignored machine-local state; pass `-NoPersist` (or set
`SPECIFY_FEATURE_NO_PERSIST=1`) when resolving paths read-only so you don't overwrite a pinned
feature. Scripts error out rather than guessing, so set the env var when operating outside a
normal stage invocation.

## Files that are managed, not authored

`.specify/integrations/*.manifest.json` records a sha256 for every installed file under
`.specify/` and `.claude/skills/`. Hand-editing a scripts/templates/skill file breaks that
integrity record and the change is liable to be reverted by `specify` on refresh. To customize
behavior, use the documented override stack instead: add
`.specify/templates/overrides/<template-name>.md` (e.g. `tasks-template.md`), which
`Resolve-Template` prefers over the core template.

Project-owned, meant to be edited: `.specify/memory/constitution.md` (and new `specs/` content).

## Constitution

`.specify/memory/constitution.md` is ratified at **v1.0.0** with five principles: spec-first,
deterministic scriptable entry points, evaluated-not-asserted (every behavioral change ships
before/after evidence — prompt and tool-definition edits count), observable runs, and bounded
authority over the game project the harness operates on. It supersedes conflicting guidance in
this file, and every `plan.md` must pass its **Constitution Check** before Phase 0 research.
Amend it only via `speckit-constitution`, which is scoped to the constitution and must not touch
source files. One open item remains: `TODO(PROJECT_SCOPE)` under Technology & Integration
Constraints — the harness's concrete scope is still inferred, and must be settled before the first
plan's Constitution Check.

## Conventions worth preserving

- Scripts are PowerShell-only here (`"script": "ps"` in `.specify/init-options.json`); the upstream
  bash/Python twins are not installed. Keep behavior parity with those twins in mind — several
  comments in the scripts exist to document exactly that.
- Scripts write UTF-8 **without BOM** (`UTF8Encoding($false)`); keep it that way.
- In `-Json` mode, status messages go to stderr so stdout stays pure JSON.
- Spec Kit command names use dots internally but hyphens when invoked: `speckit.plan` → `/speckit-plan`.
- `tasks.md` task format is `[ID] [P?] [Story] Description` — `[P]` means parallelizable, `[Story]`
  ties the task to a user story (US1, US2, …) so each story stays an independently shippable
  increment. Tests are opt-in, only when the spec asks for them.

## Unity

A `unity-editor-mcp` server is available in this environment (scene/asset/play-mode/test control,
`run_tests`, `build`, `console`). There is no Unity project in this repository yet; if the harness
targets Unity, prefer those MCP tools over shelling out to the Unity CLI, and use the latest Unity
CLI version unless told otherwise.
