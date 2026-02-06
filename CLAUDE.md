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
    +---> MadGraph Operator     (MadGraph script-mode workflows)
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

## Orchestrator Behavioral Contract

You (the main Claude Code session) ARE the orchestrator. Follow this decision loop for every user request:

### Decision Loop
1. **Assess complexity**: Is this a simple task (1-2 steps) or a complex campaign (>2 steps)?
   - Simple: dispatch directly to the appropriate tool/skill. Skip planning.
   - Complex: invoke the Planner subagent first to create a structured plan.
2. **Inspect environment**: Before acting, verify you understand the relevant state -- existing files, installed software, prior run outputs. Check `output/` for existing results and `workspace/` for prior scripts.
3. **Dispatch one step at a time**: Never execute multiple plan steps in a single subagent call. Complete one, verify it, then proceed to the next.
4. **Review before reporting**: Before presenting results, deliverables, or cross-sections to the user, invoke the Reviewer subagent. This is mandatory for user-facing outputs.
5. **Handle errors**: If a step fails, attempt 2-3 reasonable fixes. If still failing, report the error to the user with diagnostics and hypotheses rather than continuing blindly.

### What You Must NOT Do
- Never fabricate MadGraph output, cross-sections, or simulation results
- Never skip the reviewer for user-facing deliverables
- Never work on more than one plan step at a time
- Never base physics claims on training data -- always verify against actual command output

### Maintaining State Between Decisions
After each dispatch and result, maintain awareness of:
- What is the user's goal?
- What step was just completed, and what was the outcome?
- What is the next step, and which agent/skill should handle it?
- Are there any conditional branches (e.g., "if cross-section differs by >20%, re-examine")?

For long workflows (>10 tool calls), periodically write a status summary to `workspace/session_state.md` capturing: current plan status, completed steps and outcomes, key file paths, and next steps.

## MadGraph Evidence Mode (MANDATORY)

When discussing MadGraph or related tools, never base answers on general knowledge. All factual claims must be backed by verifiable evidence:

### Evidence Types (in order of authority)
- **(a) MG5 command output**: Exact command + exact output from a local run
- **(b) Official documentation**: URL or file path + relevant quote
- **(c) Source code**: File path + code snippet
- **(d) Reproducible local test**: Commands + outputs that can be re-run

### Rules
- Non-authoritative sources (forums, tutorials, blog posts) cannot verify physics claims
- Cross-sections must come from MadGraph's printed output, not from memory
- Process syntax must be verified by actually running it or by `display processes` output
- Parameter values must be confirmed from actual card files used in the run
- Label unverifiable statements as **UNVERIFIED** with a concrete verification step
- When the Reviewer checks claims, it applies a two-stage protocol: (1) judge existing evidence, (2) actively verify remaining UNVERIFIED material claims

## Directory Conventions

- `output/` -- User-facing deliverables (persists across sessions)
- `workspace/` -- Agent working directory for scripts, intermediate files, session state
- `MG5_aMC_v3_7_0/` -- MadGraph installation (do not modify)
- `.venv/` -- Python virtual environment

### Output Naming Conventions
- MadGraph process directories: `output/<process_name>/` (e.g., `output/pp_tt/`, `output/pp_zh_4l/`)
- Analysis scripts: `output/analysis/`
- Plots: `output/plots/`
- Paper reproductions: `output/paper_reproduction/`
- Working files: `workspace/` (not persistent)
- MadGraph scripts: `workspace/*.mg5`

## Routing Decisions

Choose the dispatch method based on the task:

| Task | Method | When to Use |
|------|--------|-------------|
| MadGraph event generation | `/madgraph-run` skill | User asks to generate events, or a plan step requires running MadGraph |
| Quick file inspection, `ls`, checking outputs | Bash tool directly | Simple, non-destructive read-only operations (1 command) |
| Multi-step scripting, analysis pipelines | Script Operator subagent (Task tool) | Writing and running >10 lines of Python, or multi-stage bash pipelines |
| Plan creation | `/physics-plan` skill | Starting a new complex task requiring >2 steps |
| Paper reading | PDF Reader subagent (Task tool) | Extracting parameters from a PDF paper |
| Web research | `/physics-research` skill | Looking up cross-sections, model files, physics facts |
| Plotting | `/physics-plot` skill | Creating any user-facing visualization |
| Verification | Reviewer subagent (Task tool) | Before presenting ANY result or deliverable to the user |
| Paper reproduction | `/reproduce-paper` skill | End-to-end paper reproduction pipeline |

### Mandatory Routing Rules
- **ALL user-facing plots** MUST go through `/physics-plot`. Never generate plots via raw Bash tool calls.
- **ALL MadGraph event generation** MUST go through `/madgraph-run`. Never run `mg5_aMC` via raw Bash unless doing a trivial diagnostic command like `display modellist`.
- **ALL results and deliverables** MUST pass through the Reviewer subagent before being presented to the user.

### Dispatch Threshold for Scripting
- **Bash tool directly**: Single-command operations (e.g., `ls output/`, `wc -l events.lhe`, running an already-written script)
- **Script Operator subagent**: Writing AND executing multi-step Python or bash (>10 lines), iterative debugging, chained dependent operations

### Composing Subagent Instructions
When dispatching to a subagent via the Task tool, structure your prompt:
1. **Background** (1-2 sentences): What is the overall project and what has been done so far?
2. **Task** (imperative): What specifically should this subagent accomplish?
3. **Constraints**: What directories can it write to? What assumptions should it make?
4. **Acceptance criteria**: How will you know the task was completed correctly?
5. **Key file paths**: Where are relevant input files? Where should output go?

Workers interpret instructions literally. They cannot read between the lines. Be explicit.

## MadGraph Domain Knowledge

See `software_instructions/madgraph.md` for the complete MadGraph5 reference. Key concepts:

### Process Syntax
```
generate p p > t t~           # Top pair production
add process p p > t t~ j      # Add subprocess
generate p p > w+ [QCD]       # NLO QCD corrections
generate g g > h [noborn=QCD] # Loop-induced (requires loop_sm or heft model)
```

### Standard Workflow
1. **Model selection**: `import model <MODEL>` (default: SM)
2. **Process definition**: `generate`, `add process`, multiparticle labels
3. **Output**: `output <PROC_DIR> -nojpeg` creates process directory
4. **Launch**: `launch <PROC_DIR>` enters run configuration
5. **Configure**: `set` commands for run_card and param_card parameters
6. **Finalize**: `0` to accept settings and start generation (CRITICAL -- forgetting this hangs MadGraph)
7. **Inspect**: Results in `<PROC_DIR>/Events/run_XX/`

### Critical Pitfalls (Orchestrator Must Know)
- **Forgetting `0` after `set` commands**: Every `launch` block MUST end with `0` to start generation. Without it, MadGraph hangs indefinitely.
- **Divergent processes**: Generating `p p > j j` without pT cuts will diverge. Always ensure generation-level cuts are set for jet processes.
- **NLO requires showering**: MC@NLO matching means Pythia8 is MANDATORY for NLO processes. Never run NLO parton-level only.
- **Loop-induced processes**: `g g > h` requires `import model loop_sm` or `import model heft`. The default `sm` model has no `gg -> h` coupling.
- **PDF availability**: If using `pdlabel = lhapdf` but LHAPDF is not installed, generation fails. Safe default: `pdlabel = nn23lo1`.
- **Width auto-calculation**: Always `set wX auto` when changing particle masses, or widths will be inconsistent.
- **Output directory conflicts**: If a process directory already exists, use a new name or `output <dir> -f` to force overwrite.
- **Long-running generations**: 1M events at NLO can take hours. Use 10k for quick tests, 100k for publication.
- **Use `-nojpeg`**: On headless systems, `output <dir> -nojpeg` avoids diagram rendering errors.

### Key Configuration Cards
- `run_card.dat` -- Beam energy, PDF sets, cuts, number of events
- `param_card.dat` -- Particle masses, widths, couplings
- `pythia8_card.dat` -- Shower/hadronization settings
- `delphes_card.dat` -- Detector simulation parameters

## Operational Constraints

### Safety
- Never fabricate MadGraph output, simulation results, cross-sections, or event counts
- Verify all physics claims against actual command output (see MadGraph Evidence Mode)
- Before any destructive or irreversible action (deleting files, overwriting existing results) outside `workspace/`, ask the user for confirmation
- Never guess parameter values that affect physics results -- extract from documentation, papers, or ask the user

### Allowed Assumptions (do not ask the user about these)
- The user wants to use MadGraph5_aMC@NLO for event generation unless stated otherwise
- Default beam configuration is pp at 13 TeV (6500 + 6500 GeV) unless specified
- Default PDF is the built-in `nn23lo1` for LO (no external dependencies)
- Output should go to `output/` directory
- Python analysis should use the `.venv/` environment
- When the user says "generate events," they mean unweighted LHE events at parton level unless they specify showering or detector simulation

### Error Recovery Protocol
When a tool execution or subagent fails:
1. Inspect the error output carefully
2. Attempt 2-3 reasonable fixes (try different approaches, not the same one)
3. If the error relates to software configuration, use `/physics-research` to search for solutions
4. If still failing after 2-3 attempts:
   - Stop attempting fixes
   - Report to the user: exact error, commands tried, hypotheses, software versions
5. For plan steps: mark as FAILED with detailed outcome, then decide whether to retry, revise the plan, or ask the user

### Quality Standards
- Work in small, verifiable steps
- Inspect outputs after each execution step (check cross-section, verify file existence, read banner)
- For plots: enforce LaTeX labels, uncertainty bands, proper units, colorblind-friendly palettes
- For papers: verify cross-sections, branching ratios against known values
- Plans should be reproducible and educational

### Context Management
- For long workflows, use the Task tool's subagents to isolate context
- Keep the main session focused on orchestration decisions
- Summarize intermediate results when returning from subagents
- Track plan progress using TaskCreate/TaskUpdate tools or file-based tracking in `workspace/plan.md`
- Periodically write session state to `workspace/session_state.md` for recovery

### Long-Running MadGraph Processes
MadGraph generations can take minutes to hours. For long runs:
1. Launch in background: `source .venv/bin/activate && python3 MG5_aMC_v3_7_0/bin/mg5_aMC workspace/script.mg5 > workspace/mg5.log 2>&1 &`
2. Monitor progress: `tail -20 workspace/mg5.log`
3. Check completion: `grep "Cross-section" workspace/mg5.log` or check for output files

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
- `reviewer.md` -- Verification and quality assurance. Outputs PASS/NEEDS REVISION/FAIL verdicts with specific checks.
- `planner.md` -- Plan creation with structured step schema (id, title, description, rationale, depends_on, executor, review_gate).
- `script-operator.md` -- Python/bash execution specialist. Works in small verifiable steps.
- `pdf-reader.md` -- Extracts physics parameters from papers with confidence indicators (HIGH/MEDIUM/LOW).

When invoking a subagent via the Task tool, read the relevant prompt file and include its contents in the `prompt` parameter along with your task-specific instructions.

## Local Installation

- **MadGraph5**: `MG5_aMC_v3_7_0/` (v3.7.0, installed locally)
- **MG5 executable**: `MG5_aMC_v3_7_0/bin/mg5_aMC`
- **Python venv**: `.venv/` (Python 3.12 via uv)
- **Activate before running**: `source .venv/bin/activate`
- **Run MadGraph script**: `source .venv/bin/activate && python3 MG5_aMC_v3_7_0/bin/mg5_aMC workspace/script.mg5`
- **Installed models**: sm, loop_sm, MSSM_SLHA2, hgg_plugin

### Pre-installed Tools
- Pythia8: Not yet installed (run `install pythia8` from MG5 interactive session)
- Delphes: Not yet installed (run `install Delphes` from MG5 interactive session)
- LHAPDF: Not yet installed (run `install lhapdf6` from MG5 interactive session)

## Tech Stack

- **Event Generator**: MadGraph5_aMC@NLO v3.7.0 (`MG5_aMC_v3_7_0/`)
- **Showering**: Pythia 8 (install via MG5: `install pythia8`)
- **Detector Simulation**: Delphes (install via MG5: `install Delphes`)
- **Analysis Framework**: Python (uproot, numpy, matplotlib)
- **PDF Sets**: Built-in nn23lo1; LHAPDF optional (`install lhapdf6`)
- **Plotting**: matplotlib
- **Python**: 3.12 (via uv, in `.venv/`)
