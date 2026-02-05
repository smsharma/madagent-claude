# Harness Review Findings

Consolidated review from 6 parallel subagent audits comparing the Claude Code harness against [MadAgents](https://github.com/heidelberg-hepml/MadAgents) (arXiv:2601.21015). **33 unique findings** organized by severity.

---

## Critical (13 items)

### C1. CLAUDE.md: No orchestrator behavioral contract

The CLAUDE.md states the main session is the orchestrator but never defines the decision loop. MadAgents has explicit rules: inspect environment, decide recipient, compose message, wait, evaluate, decide next. Without this, Claude Code will improvise workflows inconsistently across sessions.

**Fix**: Add an "Orchestrator Behavioral Contract" section with:
- Decision loop (assess complexity -> inspect environment -> dispatch one step -> review before reporting)
- What the orchestrator must NOT do (never execute plan steps itself, never skip reviewer)
- One-step-at-a-time rule

### C2. CLAUDE.md: MadGraph Evidence Mode missing from orchestrator

MadAgents enforces evidence rules across 3 agents (orchestrator, reviewer, workers). The Claude Code harness only has evidence guidance in `reviewer.md`. The orchestrator can still hallucinate cross-sections.

**Fix**: Add to CLAUDE.md:
- Never base answers on general MadGraph knowledge
- Four evidence types: (a) MG5 command output, (b) official docs, (c) source code, (d) reproducible local test
- Non-authoritative sources (forums, blogs) cannot verify claims
- Label unverifiable statements as UNVERIFIED

### C3. CLAUDE.md: References nonexistent tools

Line 120: "Track plan progress using Claude Code's TodoWrite/TaskCreate tools" -- `TodoWrite` doesn't exist in Claude Code. This will confuse the agent.

**Fix**: Replace with file-based plan tracking using `workspace/plan.md` and `workspace/session_state.md`.

### C4. madgraph.md: Incorrect NLO syntax `[QCD] [QED]`

Line 115: `generate p p > t t~ [QCD] [QED]` is wrong. MadGraph doesn't support stacking bracket notations. Correct syntax: `generate p p > t t~ [QCD QED]`, and it requires a loop model supporting both QCD and EW loops (e.g., `loop_qcd_qed_sm`).

### C5. madgraph.md: ggF Higgs missing model import

Line 341: `generate g g > h [noborn=QCD]` requires `import model loop_sm` first. The default `sm` model has no `gg -> h` coupling. Alternative: `import model heft` then `generate g g > h`.

### C6. madgraph.md: Single-top syntax will remove dominant diagrams

Line 338: `generate p p > t j $$ w+ w-` -- the `$$` excludes W from both s- and t-channel, but t-channel single-top proceeds via t-channel W exchange. This removes the signal. Correct: `generate p p > t j QCD=0` (pure EW = t-channel single top).

### C7. madgraph-run skill: Unresolved `<MG5_PATH>` placeholder

Step 3 says `<MG5_PATH>/bin/mg5_aMC script.mg5` but never resolves this. The skill should use the concrete local path: `source .venv/bin/activate && python3 MG5_aMC_v3_7_0/bin/mg5_aMC script.mg5`.

### C8. madgraph-run skill: No venv activation

The skill never mentions activating the Python virtual environment, which is required for MadGraph to run (needs Python 3.12 from `.venv/`).

### C9. Missing User CLI Operator (interactive sessions)

MadAgents has 4 dedicated tools for interactive CLI sessions (`run_int_cli_command`, `read_int_cli_output`, `int_cli_status`, `read_int_cli_transcript`). Claude Code's Bash tool only runs non-interactive subprocesses. This means only script mode works, no interactive MadGraph exploration.

**Mitigation**: Document a background-process polling pattern: launch MadGraph with output redirect, monitor via `tail`, check completion via `ps`.

### C10. reviewer.md: Wrong tool names

References `read_pdf` and `read_image` which don't exist. The correct Claude Code tool is `Read` (which supports both PDFs and images natively).

### C11. reproduce-paper skill: Pseudo-API Task() syntax

Lines 143-145 show `Task(subagent_type=..., prompt=...)` which is not valid Claude Code syntax. The Task tool is invoked via the tool-calling interface, not as a Python function.

### C12. physics-plan skill: References TaskCreate/TaskUpdate as primary mechanism

The entire skill is built around `TaskCreate`/`TaskUpdate` with `addBlockedBy`, but the plan tracking approach should use file-based state for persistence across sessions.

### C13. madgraph.md: Missing launch sequence documentation

The most critical agent pitfall: after `launch`, every card-editing block MUST end with `0` to start generation. Forgetting this causes MadGraph to hang indefinitely. This is documented nowhere in the reference.

---

## Important (12 items)

### I1. CLAUDE.md: Routing guidelines too vague

"Use Bash tool directly for simple scripts, Script Operator subagent for complex ones" -- no threshold defined. MadAgents has explicit mandatory routing rules.

**Fix**: Add mandatory routing rules:
- ALL user-facing plots MUST go through `/physics-plot`
- ALL MadGraph generation MUST go through `/madgraph-run`
- ALL results MUST pass through Reviewer before presenting to user

### I2. CLAUDE.md: No worker message composition guidelines

MadAgents specifies that orchestrator messages to workers must include: background paragraph, instructions, constraints, directory permissions, acceptance criteria. No guidance exists in the Claude Code harness.

### I3. CLAUDE.md: No error escalation policy

No guidance on what to do when things fail. MadAgents has explicit 2-3 retry protocol with detailed error reporting including software versions.

### I4. CLAUDE.md: Missing critical MadGraph pitfalls at orchestrator level

The orchestrator needs to know: divergent processes need pT cuts, NLO requires showering, width auto-calculation after mass changes, PDF availability issues, output directory conflicts.

### I5. CLAUDE.md: No future_note / scratchpad mechanism

MadAgents maintains a 2-8 line rolling scratchpad tracking: current goal, key constraints, next planned steps. This keeps decision-making coherent across many tool calls.

### I6. CLAUDE.md: No allowed assumptions list

MadAgents explicitly lists what the orchestrator MAY assume (default beam config, PDF set, output directory). Without this, the agent may ask unnecessary questions.

### I7. madgraph.md: Missing reweighting documentation

The reweight module is a major MadGraph feature for parameter scans without regenerating events. Completely absent from the reference.

### I8. madgraph.md: Missing parameter scanning syntax

MadGraph supports `scan:[170.0, 172.5, 175.0]` syntax directly in param_card for automated parameter sweeps.

### I9. madgraph.md: Missing gridpack generation

Gridpacks (`set gridpack True`) pre-compute integration grids for fast event generation. Essential for production workflows.

### I10. madgraph.md: Missing model restriction file documentation

The SM model includes restriction files (`restrict_no_b_mass.dat`, `restrict_ckm.dat`, etc.) accessed via `import model sm-no_b_mass`. The mechanism is not explained.

### I11. madgraph.md: MSSM model name wrong

Guide says `import model mssm` but locally installed directory is `MSSM_SLHA2`.

### I12. No wait/poll capability documented

MadAgents has `wait(minutes)` tool. Claude Code has no equivalent. Need to document polling pattern for long-running MadGraph generations using background processes.

---

## Nice-to-Have (8 items)

### N1. planner.md: Ambiguous output format

Says "JSON or markdown" -- should standardize on one format.

### N2. script-operator.md: No output format specification

Doesn't specify how to structure return values to the orchestrator.

### N3. pdf-reader.md: No confidence indicators

Doesn't instruct the agent to flag low-confidence extractions.

### N4. physics-plot.md: `text.usetex: True` without LaTeX check

Will crash on systems without LaTeX installed. Should add a try/except fallback.

### N5. madgraph.md: Missing FxFx merging (NLO jet merging)

Only MLM matching (`ickkw = 1`) is documented. FxFx (`ickkw = 3`) is the NLO counterpart.

### N6. madgraph.md: Missing SMEFT/EFT workflow documentation

BSM table mentions `NP=1` briefly but EFT workflows (coupling order syntax `NP^2==1`, `NP^2==2`) deserve fuller treatment.

### N7. madgraph.md: Missing compute_widths, Rivet, MadAnalysis5 integration docs

Several available features undocumented: `compute_widths` command, Rivet integration, MadAnalysis5 integration, `systematics` post-hoc command.

### N8. Feature parity: No structured decision schema

MadAgents uses Pydantic `OrchestratorDecision` with forced single-recipient routing. Claude Code relies on LLM reasoning. This is architectural and cannot be easily replicated.

---

## Top 5 Highest-Impact Changes

1. **Rewrite CLAUDE.md** with orchestrator behavioral contract, evidence mode, mandatory routing rules, worker composition guidelines, error escalation, and allowed assumptions (~300 lines target)

2. **Add environment preamble to all skills/subagents** -- resolve `<MG5_PATH>`, add venv activation, fix tool names (`read_pdf` -> `Read`), remove pseudo-API syntax

3. **Replace TaskCreate references** with file-based plan tracking (`workspace/plan.md`, `workspace/session_state.md`)

4. **Add pre-flight checks and operational patterns** -- document background process polling, timeout handling, LaTeX availability check

5. **Fix physics errors in madgraph.md** -- NLO syntax, ggF Higgs model, single-top process, MSSM name; add reweighting, gridpacks, parameter scanning, launch sequence docs, model restrictions

---

## Feature Parity Matrix (vs MadAgents)

| Category | Feature | MadAgents | Claude Code | Gap |
|---|---|---|---|---|
| Agents | User CLI Operator | Yes | **Missing** | Critical |
| Agents | All other roles (10/11) | Yes | Present | -- |
| Tools | Interactive CLI (4 tools) | Yes | **Missing** | Critical |
| Tools | `wait()` | Yes | **Missing** | Important |
| Tools | All others (bash, PDF, search, etc.) | Yes | Equivalent | -- |
| Orchestration | Structured decision schema | Pydantic | Prose | Important |
| Orchestration | Future note scratchpad | Yes | **Missing** | Important |
| Evidence | Multi-agent enforcement | 3 agents, 4 types | Reviewer only | Important |
| Evidence | Two-stage verification | Yes | **Missing** | Important |
| Planning | Typed PlanStep + auto-resolution | Yes | Generic TaskUpdate | Important |
| Errors | Structured retry protocol | Explicit | Informal | Important |
| Context | Summarization system | `Summarizer` class | Built-in compression | Equivalent |
