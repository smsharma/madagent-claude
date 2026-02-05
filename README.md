# MadAgent-Claude

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) agent harness for **MadGraph5_aMC@NLO**, the standard Monte Carlo event generator for high-energy particle physics.

MadAgent-Claude provides a multi-agent workflow that automates particle physics simulation campaigns end-to-end: process definition, event generation, parton showering, detector simulation, analysis, and publication-quality plotting -- all driven by natural language.

Inspired by [MadAgents](https://github.com/heidelberg-hepml/MadAgents) ([arXiv:2601.21015](https://arxiv.org/abs/2601.21015)) and adapted for the Claude Code ecosystem.

## Architecture

The system uses a **hub-and-spoke orchestration pattern** built on Claude Code's skills and Task tool subagents:

```
User ─── Claude Code CLI
              │
              ▼
        Orchestrator
              │
    ┌─────────┼─────────┐
    │         │         │
    ▼         ▼         ▼
 Planner   Script    MadGraph
           Operator   Operator
    │         │         │
    ▼         ▼         ▼
 Reviewer  PDF Reader  Plotter
              │
              ▼
          Researcher
```

- **Orchestrator** (main Claude Code session) routes tasks and manages state
- **Subagents** (via Task tool) handle specialized work in isolated contexts
- **Skills** (slash commands) provide user-invocable workflows

All communication flows through the orchestrator -- subagents never talk to each other directly.

## Skills

| Skill | Description |
|-------|-------------|
| `/madgraph-run` | Run MadGraph event generation from process definition through showering and detector simulation |
| `/physics-plan` | Create structured multi-step execution plans for complex simulation campaigns |
| `/reproduce-paper` | Reproduce simulation results from a physics paper (PDF or arXiv link) |
| `/physics-plot` | Generate publication-quality plots from simulation output |
| `/physics-research` | Research HEP topics, cross-sections, and experimental results |

## Quick Start

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Python 3.12+ (via [uv](https://github.com/astral-sh/uv) recommended)
- MadGraph5_aMC@NLO v3.7.0

### Setup

```bash
# Clone the repo
git clone https://github.com/smsharma/madagent-claude.git
cd madagent-claude

# Create Python environment
uv venv --python 3.12
source .venv/bin/activate

# Download and extract MadGraph5
wget https://launchpad.net/mg5amcnlo/3.0/3.7.x/+download/MG5_aMC_v3.7.0.tar.gz
tar xzf MG5_aMC_v3.7.0.tar.gz && rm MG5_aMC_v3.7.0.tar.gz

# (Optional) Install Pythia8 and Delphes from within MadGraph
# python3 MG5_aMC_v3_7_0/bin/mg5_aMC
# > install pythia8
# > install Delphes

# Start Claude Code in the project directory
claude
```

### Example Usage

**Generate top quark pair production events:**
```
/madgraph-run Generate 10k pp > tt~ events at 13 TeV with Pythia8 showering
```

**Reproduce results from a paper:**
```
/reproduce-paper https://arxiv.org/abs/XXXX.XXXXX
```

**Create a simulation plan:**
```
/physics-plan Study Z boson production with up to 2 jets at NLO QCD,
scan beam energies from 7 to 14 TeV
```

**Make publication plots:**
```
/physics-plot Plot the invariant mass distribution of the 4-lepton system
from output/pp_zh_4l/
```

## Project Structure

```
madagent-claude/
├── CLAUDE.md                    # Agent instructions and domain knowledge
├── .claude/
│   ├── skills/                  # User-invocable skill definitions
│   │   ├── madgraph-run.md
│   │   ├── physics-plan.md
│   │   ├── reproduce-paper.md
│   │   ├── physics-plot.md
│   │   └── physics-research.md
│   └── subagents/               # Task tool subagent prompts
│       ├── reviewer.md
│       ├── planner.md
│       ├── script-operator.md
│       └── pdf-reader.md
├── software_instructions/
│   └── madgraph.md              # Complete MadGraph5 reference guide
├── output/                      # Deliverables (gitignored)
└── workspace/                   # Working files (gitignored)
```

## How It Works

1. **Simple tasks** (1-2 steps) are dispatched directly to the appropriate tool or skill
2. **Complex tasks** go through a structured pipeline:
   - The **Planner** subagent creates a multi-step execution plan
   - Each step is executed sequentially with progress tracking
   - The **Reviewer** subagent verifies high-stakes outputs
   - Results are summarized and presented to the user

The agent enforces evidence-based verification: all MadGraph claims must be backed by actual command output. It never fabricates simulation results.

## References

- [MadAgents: Multi-Agent LLM System for Monte Carlo Simulations](https://arxiv.org/abs/2601.21015) -- Heimel et al., 2025
- [MadGraph5_aMC@NLO](https://launchpad.net/mg5amcnlo) -- Alwall et al.
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)

## License

MIT
