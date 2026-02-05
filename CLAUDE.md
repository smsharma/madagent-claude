# MadAgent-Claude: MadGraph Agent Harness

## Project Overview

This is a Claude Code agent harness for **MadGraph5_aMC@NLO**, the standard Monte Carlo event generator for high-energy physics (HEP). It provides a multi-agent workflow system that automates particle physics simulation campaigns, from process definition through event generation, showering, detector simulation, analysis, and publication-quality plotting.

This harness is modeled after [MadAgents](https://github.com/heidelberg-hepml/MadAgents) (arXiv:2601.21015) and adapted for the Claude Code ecosystem using skills, subagents, and the Task tool.

## Architecture

The system follows a **hub-and-spoke orchestration pattern**:

```
User (Claude Code CLI)
    |
    v
Orchestrator (main Claude Code session)
    |
    +---> Planner subagent      (creates multi-step execution plans)
    +---> Reviewer subagent     (verifies correctness, reviews outputs)
    +---> Script Operator       (bash/Python execution via Bash tool)
    +---> MadGraph Operator     (MadGraph interactive workflows)
    +---> PDF Reader subagent   (extracts info from physics papers)
    +---> Plotter subagent      (publication-quality visualizations)
    +---> Researcher subagent   (web research, cross-referencing)
```

**Key design principles:**
- All communication flows through the main session (orchestrator) -- subagents never talk to each other directly
- Each subagent receives only its task-specific context, not the full conversation
- Plans use structured step tracking with dependencies and status management
- High-stakes deliverables go through reviewer verification before completion
- Evidence-based verification: MadGraph claims must be backed by actual command output

## Directory Conventions

When working inside a container or dedicated environment:
- `/output` -- User-facing deliverables (persists across sessions)
- `/workspace` -- Agent working directory (can be reinitialized)
- `/opt` -- Software installations (persistent)
- `/logs` -- Tool output and transcripts

When working locally (no container):
- `output/` -- Deliverables directory (create if needed)
- `workspace/` -- Working directory for intermediate files
- Current directory for scripts and configurations

## Workflow Guidelines

### Simple Tasks (1-2 steps)
Dispatch directly to the appropriate tool/skill without creating a plan. Execute, verify the output, and present to the user.

### Complex Tasks (>2 steps)
1. Use `/physics-plan` skill or invoke the Planner subagent to create a structured plan
2. Review the plan (use Reviewer subagent for complex campaigns)
3. Execute each step sequentially, updating plan status
4. After high-stakes steps, invoke Reviewer subagent for verification
5. Present final results to user with summary

### Routing Decisions
- **MadGraph simulations** → Use `/madgraph-run` skill or MadGraph Operator subagent
- **General scripting/analysis** → Use Bash tool directly for simple scripts, Script Operator subagent for complex ones
- **Paper reproduction** → Use `/reproduce-paper` skill (orchestrates full pipeline)
- **Web research** → Use `/physics-research` skill or Researcher subagent
- **PDF extraction** → Use PDF Reader subagent
- **Plotting** → Use `/physics-plot` skill or Plotter subagent
- **Plan creation** → Use `/physics-plan` skill or Planner subagent

## MadGraph Domain Knowledge

See `software_instructions/madgraph.md` for the complete MadGraph5 reference. Key concepts:

### Process Syntax
```
generate p p > t t~           # Top pair production
add process p p > t t~ j      # Add subprocess
generate p p > w+ [QCD]       # NLO QCD corrections
generate g g > h [noborn=QCD] # Loop-induced process
```

### Standard Workflow
1. **Model selection**: `import model <MODEL>` (default: SM)
2. **Process definition**: `generate`, `add process`, multiparticle labels
3. **Output**: `output <PROC_DIR>` creates process directory
4. **Launch**: `launch <PROC_DIR>` enters run configuration
5. **Configure**: Edit run_card.dat, param_card.dat via `set` commands
6. **Execute**: Event generation, showering (Pythia8), detector sim (Delphes)
7. **Inspect**: Results in `<PROC_DIR>/Events/run_XX/`

### Coupling Constraints
- `QED=N` / `QCD=N` limits coupling orders
- `[QCD]` / `[QED]` brackets specify NLO corrections
- `noborn=QCD` for loop-induced processes

### Key Configuration Cards
- `run_card.dat` -- Beam energy, PDF sets, cuts, number of events
- `param_card.dat` -- Particle masses, widths, couplings
- `pythia8_card.dat` -- Shower/hadronization settings
- `delphes_card.dat` -- Detector simulation parameters

## Operational Constraints

### Safety
- Never fabricate MadGraph output or simulation results
- Verify all physics claims against actual command output
- Default to read-only operations; confirm before destructive actions
- Never guess parameter values -- extract from documentation or ask the user

### Quality Standards
- Work in small, verifiable steps
- Inspect outputs after each execution step
- For plots: enforce LaTeX labels, uncertainty bands, proper units, colorblind-friendly palettes
- For papers: verify cross-sections, branching ratios against known values
- Plans should be reproducible and educational

### Context Management
- For long workflows, use the Task tool's subagents to isolate context
- Keep the main session focused on orchestration decisions
- Summarize intermediate results when returning from subagents
- Track plan progress using Claude Code's TodoWrite/TaskCreate tools

## Available Skills

| Skill | Purpose | Maps to MadAgents Role |
|-------|---------|----------------------|
| `/madgraph-run` | MadGraph event generation workflow | MadGraph Operator |
| `/physics-plan` | Structured multi-step planning | Planner + Plan Updater |
| `/reproduce-paper` | Full paper reproduction pipeline | Orchestrator pipeline |
| `/physics-plot` | Publication-quality plotting | Plotter |
| `/physics-research` | HEP web research | Researcher |

## Subagent Prompts

Subagent prompt files in `.claude/subagents/` define specialized behavior for Task tool invocations:
- `reviewer.md` -- Verification and quality assurance
- `planner.md` -- Plan creation and management
- `script-operator.md` -- Complex bash/Python execution

Use these by reading the prompt file and including it in the Task tool's `prompt` parameter.

## Local Installation

- **MadGraph5**: `MG5_aMC_v3_7_0/` (v3.7.0, installed locally)
- **MG5 executable**: `MG5_aMC_v3_7_0/bin/mg5_aMC`
- **Python venv**: `.venv/` (Python 3.12 via uv)
- **Activate before running**: `source .venv/bin/activate`
- **Run MadGraph**: `source .venv/bin/activate && python3 MG5_aMC_v3_7_0/bin/mg5_aMC`
- **Run MadGraph script**: `source .venv/bin/activate && python3 MG5_aMC_v3_7_0/bin/mg5_aMC script.mg5`

## Tech Stack

- **Event Generator**: MadGraph5_aMC@NLO v3.7.0 (`MG5_aMC_v3_7_0/`)
- **Showering**: Pythia 8 (install via MG5: `install pythia8`)
- **Detector Simulation**: Delphes (install via MG5: `install Delphes`)
- **Analysis Framework**: Python (uproot, numpy, matplotlib)
- **PDF Sets**: Built-in nn23lo1; LHAPDF optional (`install lhapdf6`)
- **Plotting**: matplotlib
- **Python**: 3.12 (via uv, in `.venv/`)
