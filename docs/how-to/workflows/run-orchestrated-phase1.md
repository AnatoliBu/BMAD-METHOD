---
title: 'Run Orchestrated Phase 1'
description: How to execute Phase 1 (Analysis) using the orchestrated execution layer
---

# Run Orchestrated Phase 1

This guide explains how to execute Phase 1 (Analysis) of the BMAD Method using the orchestrated execution layer. The orchestrator enables autonomous, stateful workflow execution while maintaining full compatibility with BMAD v6 semantics.

## Prerequisites

- BMAD v6 installed in your project (`npx bmad-method@alpha install`)
- An AI coding assistant (Claude Code, Cursor, Windsurf, etc.)
- Understanding of BMAD Method phases

## Quick Start

### Standard User Workflow

1. **Install BMAD v6** in your target project:

   ```bash
   npx bmad-method@alpha install
   ```

2. **Run workflow-init** in your IDE/agent chat:

   ```text
   *workflow-init
   ```

   This creates/updates the workflow-status file (typically at `{planning_artifacts}/bmm-workflow-status.yaml`).

3. **Execute Phase 1 workflows** interactively using the workflow IDs from your installation:
   - Use `/workflows` or check your `_bmad/bmm/workflows/1-analysis/` directory
   - Workflow commands depend on your installed configuration
   - Example: `*workflow-init` is always available after installation

:::note[Workflow Commands]
Commands like `*brainstorm`, `*research`, `*product-brief` depend on the installed workflow set.
Always verify available workflows via `/workflows` or the workflow index in your IDE.
:::

## Mandatory vs Optional Components

### Path Conventions

Paths are shown in two formats throughout this documentation:

| Context               | Base Path              | Example                                            |
| --------------------- | ---------------------- | -------------------------------------------------- |
| **Repository**        | `src/`                 | `src/core/orchestrator/phase1-orchestrator.yaml`   |
| **Installed Project** | `_bmad/` (or `.bmad/`) | `_bmad/core/orchestrator/phase1-orchestrator.yaml` |

The installer copies files from `src/` to `_bmad/` in your target project.

### Path Variables

These variables are resolved from your project's configuration (`_bmad/bmm/config.yaml`):

| Variable               | Default Value                        | Description                    |
| ---------------------- | ------------------------------------ | ------------------------------ |
| `{project-root}`       | Current working directory            | Root of your project           |
| `{output_folder}`      | `_bmad-output`                       | Base output directory          |
| `{planning_artifacts}` | `{output_folder}/planning-artifacts` | Where planning docs are stored |

**Supported aliases for `{planning_artifacts}`:**

- `{output_folder}/planning-artifacts` (default)
- `docs/planning`
- `_bmad-output/planning-artifacts`

### Mandatory for Phase 1 Orchestration

These components are **required** for faithful Phase 1 execution.

**In Repository (`src/`):**

| Component                | Repo Path                                                      | Purpose                                  |
| ------------------------ | -------------------------------------------------------------- | ---------------------------------------- |
| workflow-init            | `src/modules/bmm/workflows/workflow-status/init/workflow.yaml` | Track selection and status file creation |
| Phase 1 workflows        | `src/modules/bmm/workflows/1-analysis/*`                       | Research, product-brief content          |
| phase1-orchestrator.yaml | `src/core/orchestrator/phase1-orchestrator.yaml`               | Main orchestration manifest              |
| status-resolver.yaml     | `src/core/orchestrator/status-resolver.yaml`                   | Multi-format status file support         |
| artifact-registry.yaml   | `src/core/orchestrator/artifact-registry.yaml`                 | Artifact registration rules              |
| checklist-evaluator.yaml | `src/core/orchestrator/checklist-evaluator.yaml`               | Hard gate validation                     |
| router-dispatcher.yaml   | `src/core/orchestrator/router-dispatcher.yaml`                 | Sub-workflow routing                     |
| instructions.md          | `src/core/orchestrator/instructions.md`                        | Integration rules for LLM                |

**In Installed Project (`_bmad/`):**

| Component                | Installed Path                                           | Purpose                                  |
| ------------------------ | -------------------------------------------------------- | ---------------------------------------- |
| workflow-init            | `_bmad/bmm/workflows/workflow-status/init/workflow.yaml` | Track selection and status file creation |
| Phase 1 workflows        | `_bmad/bmm/workflows/1-analysis/*`                       | Research, product-brief content          |
| phase1-orchestrator.yaml | `_bmad/core/orchestrator/phase1-orchestrator.yaml`       | Main orchestration manifest              |
| status-resolver.yaml     | `_bmad/core/orchestrator/status-resolver.yaml`           | Multi-format status file support         |
| artifact-registry.yaml   | `_bmad/core/orchestrator/artifact-registry.yaml`         | Artifact registration rules              |
| checklist-evaluator.yaml | `_bmad/core/orchestrator/checklist-evaluator.yaml`       | Hard gate validation                     |
| router-dispatcher.yaml   | `_bmad/core/orchestrator/router-dispatcher.yaml`         | Sub-workflow routing                     |
| instructions.md          | `_bmad/core/orchestrator/instructions.md`                | Integration rules for LLM                |

### Optional Components (Autopilot Mode)

These are only needed if you enable autonomous brainstorm execution:

| Component                       | Repo Path                                               | Installed Path                                            | Purpose                           |
| ------------------------------- | ------------------------------------------------------- | --------------------------------------------------------- | --------------------------------- |
| brainstorm-autopilot.yaml       | `src/core/orchestrator/brainstorm-autopilot.yaml`       | `_bmad/core/orchestrator/brainstorm-autopilot.yaml`       | Multi-agent brainstorm automation |
| autonomous-loop-controller.yaml | `src/core/orchestrator/autonomous-loop-controller.yaml` | `_bmad/core/orchestrator/autonomous-loop-controller.yaml` | Loop bounds and stop conditions   |

### Not Required for Execution

Documentation files are informational only:

- `docs/reference/workflows/*`
- `docs/reference/workflows/phase1-orchestrator.md`

## Understanding the Architecture

### What the Repository Provides

The BMAD repository provides **contracts and rules** for orchestration:

- **Workflow definitions** - What steps exist and their sequencing
- **Validation rules** - Checklists and gates for artifact quality
- **Routing logic** - How to dispatch to sub-workflows
- **Status abstraction** - How to track progress across formats

### What You Must Provide

The repository does **not** provide a ready-made execution engine. You need:

- **LLM orchestrator** - An AI agent that reads and executes the orchestrator configs
- **File system access** - Ability to create/read artifacts
- **User interaction** - For interactive workflows (or autopilot config)

## Orchestrated Execution Algorithm

When implementing or using an LLM orchestrator, follow this sequence:

### Step 1: Initialize

```yaml
# Verify installation exists
check: "{project-root}/_bmad/..." directory exists
action: If not, run "npx bmad-method@alpha install"
```

### Step 2: Run workflow-init

```yaml
# Execute workflow-init (not heuristics)
workflow: workflow-init
path: '_bmad/bmm/workflows/workflow-status/init/workflow.yaml'
output: '{planning_artifacts}/bmm-workflow-status.yaml'
```

### Step 3: Read Status

```yaml
# Use status-resolver.yaml rules
action: Find status file in priority order
formats: [.yaml, .md]
locations:
  - '{planning_artifacts}/bmm-workflow-status.yaml'
  - '{planning_artifacts}/bmm-workflow-status.md'
  - '{project-root}/bmm-workflow-status.yaml'
  - etc.
```

### Step 4: Load Orchestrator Config

```yaml
# Load phase1-orchestrator.yaml
action: Parse workflow steps, routers, checklists
source: '_bmad/core/orchestrator/phase1-orchestrator.yaml'
```

### Step 5: Execute Permitted Steps

```yaml
# Run workflows allowed by status/track
for each step in phase1_steps:
  if step.allowed_by_track:
    execute workflow at step.workflow_path
    if step.supports_autopilot AND autopilot_requested:
      load brainstorm-autopilot.yaml
      load autonomous-loop-controller.yaml
```

### Step 6: Validate and Register Artifacts

After **each** artifact is produced:

```yaml
# Validate using artifact-registry.yaml
check: File exists on disk
check: Required sections present
check: Minimum content thresholds

# Evaluate checklist using checklist-evaluator.yaml
evaluate: workflow-specific checklist
if required_items_fail: block completion
if optional_items_fail: mark_for_review

# Register in workflow-status
update: artifacts map
update: phase_attribution
update: confidence level
```

### Step 7: Complete Phase 1

```yaml
# Final status must be readable by Phase 2 "without heuristics"
verify: workflow-status file exists
verify: all required artifacts registered
verify: no blocking open questions
```

## Execution Modes

### Interactive Mode (Default)

Standard user-driven execution:

1. Present workflow step to user
2. Wait for user input/confirmation
3. Execute step actions
4. Update workflow-status
5. Proceed to next step

### Autopilot Mode (Brainstorm Only)

Autonomous execution with bounded loops:

1. Enable with `autopilot=true` parameter
2. Analyst and Explorer agents alternate
3. Intermediate artifacts captured in `{planning_artifacts}/analysis/intermediate/`
4. Terminate on:
   - Completion criteria met
   - Convergence detected (no new ideas in 2 iterations)
   - Max iterations reached (default: 10)
   - User stop request
   - Timeout

### Hybrid Mode

Mix of autopilot brainstorm and interactive research:

```yaml
steps:
  - workflow-init: interactive
  - brainstorm: autopilot (if enabled)
  - research: interactive (always)
  - product-brief: interactive (always)
```

## Status File Locations

The orchestrator supports multiple status file locations for flexibility:

| Priority | Location                                        | Format   |
| -------- | ----------------------------------------------- | -------- |
| 1        | `{planning_artifacts}/bmm-workflow-status.yaml` | YAML     |
| 2        | `{planning_artifacts}/bmm-workflow-status.md`   | Markdown |
| 3        | `{project-root}/bmm-workflow-status.yaml`       | YAML     |
| 4        | `{project-root}/bmm-workflow-status.md`         | Markdown |
| 5        | `docs/bmm-workflow-status.yaml`                 | YAML     |
| 6        | `docs/bmm-workflow-status.md`                   | Markdown |

## Track Gating

Certain workflows are gated by track selection. **Gating is enforced by the router-dispatcher**, not by user commands. If you attempt to invoke a gated workflow directly, the dispatcher will block it with error `R003`.

| Workflow           | Method Track | Enterprise Track |
| ------------------ | ------------ | ---------------- |
| Brainstorm         | ✅           | ✅               |
| Market Research    | ✅           | ✅               |
| Domain Research    | ✅           | ✅               |
| Technical Research | ❌           | ✅               |
| Product Brief      | ✅           | ✅               |
| Security Review    | ❌           | ✅               |
| DevOps Planning    | ❌           | ✅               |

## Artifact Propagation Rules

Every artifact produced must follow these rules:

1. **Exist on disk** before registration
2. **Pass validation** (required sections, minimum content)
3. **Be registered** in workflow-status file
4. **Have phase attribution** (which phase created it)
5. **Have confidence level** (high/medium/low)

**Critical:** Unregistered artifacts are **invisible** to Phase 2.

## Minimal Orchestrator Contract

For any LLM orchestrator implementation, the following contract must be satisfied:

### Input Requirements

| Parameter         | Type    | Required | Description                              |
| ----------------- | ------- | -------- | ---------------------------------------- |
| `project_root`    | string  | Yes      | Absolute path to project root            |
| `autopilot`       | boolean | No       | Enable brainstorm autopilot mode         |
| `status_format`   | string  | No       | Preferred status format (`yaml` or `md`) |
| `status_location` | string  | No       | Override status file location            |

### Output Guarantees

| Output                    | Description                                      |
| ------------------------- | ------------------------------------------------ |
| `bmm-workflow-status.*`   | Updated status file at configured location       |
| Artifact files            | Created at paths defined in workflow definitions |
| `artifacts` map in status | List of all registered artifacts with metadata   |

### Hard Guarantees

The orchestrator **must** ensure:

1. **Artifact existence** - File exists on disk before registration in status
2. **Checklist execution** - If workflow declares a checklist, it is evaluated
3. **Status updates** - Status file updated after each workflow step completion
4. **Track gating** - Router-dispatcher enforces track-based access control
5. **Bounded loops** - Autopilot respects `max_iterations` and `stop_conditions`

## Troubleshooting

### Status file not found

```text
Error: No workflow status file found
```

**Solution:** Run `*workflow-init` to create the status file.

### Artifact validation failed

```text
Error: E002 - Artifact failed validation
```

**Solution:** Check the checklist requirements for the workflow. Ensure required sections exist and meet minimum content thresholds.

### Gate blocked by track

```text
Error: R003 - Route blocked by track gate
```

**Solution:** The requested workflow requires a different track. Either change your track selection or use an allowed workflow.

## Related Documentation

- [Phase 1 Orchestrator Reference](../../reference/workflows/phase1-orchestrator.md)
- [Core Workflows](../../reference/workflows/core-workflows.md)
- [Global Configuration](../../reference/configuration/global-config.md)
