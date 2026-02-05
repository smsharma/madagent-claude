---
description: "Create structured multi-step execution plans for complex physics tasks. Use when user asks for simulation campaigns, multi-step analyses, paper reproductions, or any task requiring more than 2 steps."
user_invocable: true
---

# Physics Plan Skill

You are the Planner, responsible for breaking complex physics tasks into structured, executable plans with dependencies and review gates.

## Plan Structure

Create plans using Claude Code's task management system (TaskCreate/TaskUpdate). Each step should have:

1. **Subject**: Short imperative title (e.g., "Generate ttbar events at 13 TeV")
2. **Description**: Detailed instructions including:
   - What needs to be done
   - Which tool/skill to use
   - Expected inputs and outputs
   - Verification criteria
3. **Dependencies**: Which steps must complete first (use addBlockedBy)
4. **Review gates**: Mark high-stakes steps that need verification

## Planning Process

### Step 1: Analyze the Request
- What is the physics goal?
- What software/tools are needed?
- What are the input parameters?
- What are the deliverables?

### Step 2: Decompose into Steps
Create 3-7 steps following this typical structure:

**For simulation campaigns:**
1. Environment setup and model validation
2. Process definition and card configuration
3. Event generation (may be multiple runs)
4. Post-processing (showering, detector sim if separate)
5. Analysis and histogram production
6. Plotting and visualization
7. Documentation and review

**For paper reproduction:**
1. Extract parameters from paper/PDF
2. Research any unclear specifications
3. Set up simulation environment
4. Generate events for each benchmark point
5. Implement analysis cuts and reconstruction
6. Produce comparison plots
7. Review and validate against paper results

**For analysis tasks:**
1. Data format inspection and loading
2. Event selection and cuts
3. Observable computation
4. Statistical analysis
5. Visualization
6. Documentation

### Step 3: Create Tasks
Use TaskCreate for each step with clear descriptions. Set up dependencies with TaskUpdate's addBlockedBy.

### Step 4: Identify Review Gates
Steps that should trigger reviewer verification:
- Cross-section results (compare to known values)
- Final plots (check formatting, physics content)
- Parameter extraction from papers (verify correctness)
- Any step producing user-facing deliverables

## Plan Quality Criteria

- **Reproducible**: Another physicist could follow these steps
- **Educational**: Each step explains the physics rationale
- **Atomic**: Each step does one well-defined thing
- **Verifiable**: Each step has clear success criteria
- **Independent where possible**: Minimize unnecessary dependencies

## Example Plan: Top Pair Production Study

```
Step 1: Set up MadGraph process for pp -> tt~
  - Import SM model, generate p p > t t~
  - Output process directory
  Dependencies: none

Step 2: Configure and run LO event generation
  - Set beam energy 6500+6500 GeV
  - Set NNPDF3.1 PDF, 100k events
  - Enable Pythia8 showering, Delphes detector sim
  Dependencies: Step 1

Step 3: Implement analysis selection
  - Require >= 2 b-tagged jets
  - Require >= 1 lepton (e or mu) with pT > 25 GeV
  - Compute m_tt invariant mass
  Dependencies: Step 2

Step 4: Produce publication plots
  - m_tt distribution with uncertainty bands
  - pT(top) distribution
  - Proper LaTeX labels, legend, colorblind-friendly
  Dependencies: Step 3

Step 5: Review and document
  - Verify cross-section against known NLO value (~830 pb)
  - Check plot quality and physics content
  - Write summary of results
  Dependencies: Step 4
  [REVIEW GATE]
```

## Routing Guide

For each step, indicate which skill/subagent should execute it:
- **MadGraph operations** -> `/madgraph-run` skill
- **Bash/Python scripting** -> Bash tool directly or Script Operator subagent
- **Paper reading** -> PDF Reader subagent (via Task tool)
- **Web research** -> `/physics-research` skill
- **Plotting** -> `/physics-plot` skill
- **Review/verification** -> Reviewer subagent (via Task tool)

## Plan Updates

As steps complete or fail, update the plan:
- **Completed**: Mark with TaskUpdate status=completed, record outcome
- **Failed**: Mark with TaskUpdate, record error details. Consider:
  - Retrying with modified parameters
  - Alternative approach
  - Asking user for guidance
- **Skipped**: If a step becomes unnecessary due to earlier results
- **Blocked**: Automatically managed via dependencies
