# NDNSF Agentic Workflow

This project uses three agent workflow tools by default: CodeGraph, Spec Kit,
and GSD Core. They serve different purposes and should be used proactively.

## CodeGraph

Use CodeGraph before broad code exploration, architecture questions, impact
analysis, bug tracing, or source edits.

```bash
codegraph status .
codegraph explore "<symbols, files, or question>"
```

Use `rg` after CodeGraph for exact strings, scripts, docs, logs, configs, and
generated names. If the CodeGraph index is stale, run `codegraph sync .` before
relying on it.

## Spec Kit

This repository is initialized as a Spec Kit project. Use Spec Kit for new
features, protocol/API changes, architecture changes, evaluation-plan changes,
or any task that needs durable requirements and implementation steps.

Normal flow:

```text
$speckit-constitution
$speckit-specify
$speckit-clarify    # optional, when requirements are ambiguous
$speckit-plan
$speckit-tasks
$speckit-analyze    # optional, before implementation
$speckit-implement
$speckit-converge
```

Spec Kit project files live in `.specify/`, and Codex skills live in
`.agents/skills/`. Read `.specify/memory/constitution.md` before creating or
modifying Spec Kit artifacts.

Do not force Spec Kit onto tiny one-line fixes, direct command-output requests,
or straightforward slide/text edits unless the user asks for a formal spec.

## GSD Core

GSD Core is installed for Codex on this machine. Use GSD for long-running,
multi-phase, unclear, or stateful work.

Normal flow:

```text
$gsd-discuss-phase
$gsd-plan-phase
$gsd-execute-phase
$gsd-verify-work
$gsd-progress / $gsd-resume-work as needed
```

Good GSD fits in this repository include VM setup, benchmark campaigns, major
NDNSF protocol changes, distributed-inference work, and proposal-wide slide
revisions.

## Priority

For code work, CodeGraph comes first. For durable feature work, Spec Kit defines
the spec and plan. For long-running multi-phase work, GSD manages state and
verification. These tools supplement the NDNSF-specific rules in `AGENTS.md`;
they do not override security, naming, testing, or MiniNDN validation rules.
