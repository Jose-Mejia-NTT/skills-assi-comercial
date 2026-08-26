# bcv-hab-orchestrator

Human-in-the-loop orchestrator for the BCV HU/HAB pipeline on GitHub Copilot.

## What it is

This is not a skill that writes code or performs analysis. It is a **conductor agent** that guides the user through the three BCV skills in the correct order and validates gates between them.

Because GitHub Copilot does not support autonomous custom agents that invoke other skills, this orchestrator works as an **interactive playbook**: it tells the user which skill to run, what inputs to provide, and what gate to validate next.

## Pipeline steps

1. `bcv-business-resolution` → `business-resolution.yaml`
2. `bcv-technical-impact-and-story` → `technical-impact-analysis.yaml` + `technical-story-enriched.md`
3. `bcv-implementation-orchestrator` → `implementation-orchestration-plan.md` + `implementation-prompts/`

## State file

The orchestrator maintains:

```text
docs/historial/<hu-slug>-pipeline-state.yaml
```

This file records the current phase, gate status, and blockers so the session can be resumed.

## Usage

1. Provide the HU/HAB text or identifier.
2. The orchestrator initializes the state file and tells you to run Step 1.
3. After each step, paste the artifact(s) back or confirm the file path.
4. The orchestrator validates the gate and either stops with blockers or proceeds to the next step.

## Files

| File | Purpose |
|---|---|
| `AGENT.md` | Main orchestration instructions. |
| `assets/pipeline-state.template.yaml` | Template for the state file. |
