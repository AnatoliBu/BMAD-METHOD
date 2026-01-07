# Phase 1 Orchestrator Instructions
# Integration document for BMAD v6 Phase 1 Orchestrated Execution

<critical>The orchestrator is governed by: {project-root}/_bmad/core/orchestrator/phase1-orchestrator.yaml</critical>
<critical>All behavior is derived from on-disk workflow assets</critical>
<critical>If something is missing on disk, it is an error—not an opportunity to improvise</critical>

## Overview

This document provides instructions for executing Phase 1 (Analysis) of the BMAD Method
using the orchestrated execution layer. The orchestrator enables autonomous, stateful
workflow execution while maintaining full compatibility with BMAD v6 semantics.

## Core Principles

### 1. BMAD Is the Source of Truth

- All behavior derived from workflow assets (workflow.yaml, routers, templates, checklists)
- Never invent workflows, steps, or artifacts
- Missing assets are errors, not opportunities to improvise

### 2. Orchestration Is Structural

- Execute as stateful workflow, not conversation
- Store progress in artifacts, not chat history
- Model-to-model interaction is execution logic, not conversation

### 3. Workflow Execution Is Deterministic

- workflow.yaml declares what may happen
- Orchestrator only executes what is declared
- May stop, pause, or fail—never silently skip

## Execution Modes

### Interactive Mode (Default)

Full user interaction at each step:

1. Present step to user
2. Wait for input/confirmation
3. Execute step actions
4. Update status
5. Proceed to next step

### Autopilot Mode (Brainstorm Only)

Autonomous execution with bounded loops:

1. Enabled via `autopilot=true` parameter
2. Limited to brainstorm step
3. Analyst and Explorer agents alternate
4. Completion based on artifact criteria
5. Terminates at max_iterations or convergence

## Phase 1 Workflow

### Step 1: Workflow Init (Required)

```yaml
workflow: workflow-init
agent: analyst
path: {project-root}/_bmad/bmm/workflows/workflow-status/init/workflow.yaml
output: {planning_artifacts}/bmm-workflow-status.yaml
```

Actions:
- Determine project type (greenfield/brownfield)
- Select track (method/enterprise)
- Choose discovery workflows
- Generate workflow status file

### Step 2: Brainstorm (Optional)

```yaml
workflow: brainstorm
agent: analyst
path: {project-root}/_bmad/core/workflows/brainstorming/workflow.md
output: {planning_artifacts}/analysis/brainstorm-session-{date}.md
supports_autopilot: true
```

Completion criteria (Principle #14):
- [ ] Project brief exists
- [ ] Assumptions documented
- [ ] Research plan defined
- [ ] Open questions bounded (≤10)

### Step 3: Research (Optional)

```yaml
workflow: research
agent: analyst
path: {project-root}/_bmad/bmm/workflows/1-analysis/research/workflow.md
router: research-type-router
```

Routes:
- market → `./market-steps/step-01-init.md`
- domain → `./domain-steps/step-01-init.md`
- technical → `./technical-steps/step-01-init.md` (Enterprise only)

### Step 4: Product Brief (Optional)

```yaml
workflow: product-brief
agent: analyst
path: {project-root}/_bmad/bmm/workflows/1-analysis/create-product-brief/workflow.md
output: {planning_artifacts}/product-brief.md
```

## Status Abstraction

The workflow status is an abstraction supporting multiple formats and locations:

### Supported Formats
- `.yaml` - Standard YAML format
- `.md` - Markdown with YAML frontmatter

### Search Locations (Priority Order)
1. `{planning_artifacts}/bmm-workflow-status.yaml`
2. `{planning_artifacts}/bmm-workflow-status.md`
3. `{project-root}/bmm-workflow-status.yaml`
4. `{project-root}/bmm-workflow-status.md`
5. `docs/bmm-workflow-status.yaml`
6. `docs/bmm-workflow-status.md`

### Required Fields
```yaml
artifacts: {} # Map of artifact_id -> file_path
phase_attribution: {} # Map of artifact_id -> phase
confidence: {} # Map of artifact_id -> high|medium|low
key_findings: [] # Optional list of findings
```

## Artifact Registry

Every artifact produced MUST be registered:

```yaml
# Registration format
artifact_id:
  path: "relative/path/to/artifact.md"
  phase: 1
  confidence: "high|medium|low"
  created_at: "ISO datetime"
  workflow_id: "originating workflow"
```

### Registration Rules
- Artifact must exist on disk before registration
- Registration updates workflow-status file
- Unregistered artifacts are invisible to Phase 2

## Checklist Evaluation

Checklists are hard gates:

```yaml
# Example checklist evaluation
checklist: brainstorm-completion
items:
  - id: project_brief_exists
    criteria: "Project brief section exists with content"
    required: true
    
  - id: assumptions_documented
    criteria: "At least one assumption stated"
    required: true
```

### Failure Handling
- **block**: Prevents workflow completion
- **mark_for_review**: Allows completion with low confidence
- **warn**: Informational only

## Autonomous Loop Controls

All autonomous loops must have bounds:

```yaml
autopilot_loop:
  max_depth: 3
  max_iterations: 10
  stop_conditions:
    - completion_criteria_met
    - convergence_detected
    - user_stop
    - max_iterations_reached
    - timeout
```

### On Incomplete
Record the following:
- blocked_reason
- low_confidence_items
- open_questions

## Phase 1 Completion Contract

Phase 1 is complete when:
- [ ] Workflow steps executed or explicitly skipped
- [ ] Artifacts generated and propagated to status
- [ ] Status updated and readable by Phase 2

### Phase 2 Can Load When
- Workflow-status file exists
- All required artifacts referenced
- No heuristics needed to locate artifacts

## Orchestrator Commands

### /workflow-status
Display resolved workflow status from first found location.

### /workflow-status --validate
Validate status file against orchestrator requirements.

### /workflow-status --update
Update status with new artifact or finding.

## Path Compatibility

Support for common path variations:

| Primary | Alias |
|---------|-------|
| `_bmad/` | `.bmad/` |
| `{planning_artifacts}` | `{output_folder}/planning-artifacts` |
| `{planning_artifacts}` | `docs/planning` |

## Error Handling

| Code | Error | Action |
|------|-------|--------|
| E001 | Artifact not found | Mark missing, update status |
| E002 | Validation failed | Record, lower confidence |
| E003 | Duplicate registration | Reject unless --force |
| E004 | Unregistered reference | Error - does not exist |
| R001 | Router not found | Fail with clear error |
| R002 | No matching route | Prompt user selection |
| R003 | Gate blocked | Inform track requirement |

## Replaceable Design

The orchestrator can be removed without breaking:
- Workflow assets (workflow.yaml files)
- Produced artifacts
- Downstream phase behavior

If removing the orchestrator breaks BMAD semantics, the design is wrong.

---

*Phase 1 Orchestrator v1.0.0 - BMAD v6 Compatible*
