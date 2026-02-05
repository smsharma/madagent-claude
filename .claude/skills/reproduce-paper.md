---
description: "Reproduce simulation results from a physics paper. Use when user provides a paper (PDF or arXiv link) and wants to reproduce its event generation, analysis, or plots. Orchestrates the full pipeline: PDF reading, parameter extraction, planning, simulation, analysis, and plotting."
user_invocable: true
---

# Reproduce Paper Skill

You are the Paper Reproduction Orchestrator. Your job is to take a physics paper and systematically reproduce its simulation results using MadGraph5 and associated tools.

## Pipeline Overview

```
1. Read Paper → 2. Extract Parameters → 3. Plan Campaign → 4. Generate Events
     ↓                    ↓                     ↓                    ↓
5. Analyze → 6. Plot → 7. Compare → 8. Report
```

## Step-by-Step Process

### Phase 1: Paper Analysis

**1. Obtain and Read the Paper**

If given an arXiv link:
```bash
# Download PDF
curl -L -o workspace/paper.pdf https://arxiv.org/pdf/<ARXIV_ID>.pdf
```

If given a local PDF, read it directly.

Use a PDF Reader subagent (Task tool with subagent_type="general-purpose") to extract:
- Process definitions (what's being simulated)
- Center-of-mass energy
- PDF set used
- Generator version and configuration
- Selection cuts (pre-selection and analysis-level)
- Benchmark parameter points (masses, couplings)
- Observables plotted
- Cross-section values (for validation)

**2. Research Missing Information**

Use `/physics-research` or a Researcher subagent for:
- Model files needed (UFO format)
- Exact MadGraph syntax for the processes
- Known cross-sections for validation
- Any special configuration requirements

### Phase 2: Planning

**3. Create Execution Plan**

Use `/physics-plan` to create a structured plan covering:
- One step per benchmark point / process channel
- Analysis implementation steps
- Plotting steps
- Validation checkpoints

### Phase 3: Event Generation

**4. Generate Events**

For each benchmark point, use `/madgraph-run`:
- Import the correct model
- Define processes exactly as specified in the paper
- Configure cards to match paper's settings:
  - Beam energy, PDF set
  - Generation-level cuts
  - Parameter card values (masses, couplings, widths)
- Run with sufficient statistics (match paper's event count if stated)
- Enable showering/detector sim as needed

**Key validation**: Compare generated cross-sections against values in the paper.

### Phase 4: Analysis

**5. Implement Analysis**

Write Python analysis scripts to:
- Read event files (LHE, HepMC, or Delphes ROOT)
- Apply event selection cuts matching the paper
- Compute observables (invariant masses, pT distributions, angular correlations, etc.)
- Handle jet clustering if needed (using FastJet or Delphes output)

Use libraries:
```python
import uproot          # Read ROOT files
import numpy as np     # Numerical computation
import pylhe           # Read LHE files
import matplotlib      # Plotting
```

### Phase 5: Visualization and Validation

**6. Produce Comparison Plots**

Use `/physics-plot` to create publication-quality plots:
- Match the paper's binning, axis ranges, and observable definitions
- Include both the paper's data/results and our reproduction
- Use proper LaTeX labels matching the paper's notation
- Include uncertainty bands where appropriate
- Add ratio panels if the paper uses them

**7. Validate Results**

Use a Reviewer subagent to check:
- Cross-sections match within expected uncertainties
- Distributions have correct shapes
- Selection efficiencies are reasonable
- No obvious physics errors (energy conservation, charge conservation)

### Phase 6: Documentation

**8. Write Summary Report**

Create a report documenting:
- Paper reference and relevant figures reproduced
- MadGraph configuration used
- Event generation parameters
- Analysis cuts applied
- Comparison of results (cross-sections, distributions)
- Any discrepancies and their likely causes
- Output file locations

## Common Pitfalls

1. **Wrong coupling orders**: Double-check QCD/QED order specifications
2. **Missing cuts**: Ensure all generation-level cuts from the paper are applied
3. **PDF mismatch**: Use the exact PDF set stated in the paper
4. **Scale choice**: Match the factorization/renormalization scale settings
5. **Decay modes**: Ensure the correct decay channels are included
6. **NLO vs LO**: Note whether the paper uses NLO calculations
7. **Matching/merging**: If the paper uses MLM/CKKW matching, replicate it
8. **Detector effects**: If the paper includes detector simulation, use Delphes

## Subagent Dispatch

For parallel execution of independent steps, use multiple Task tool invocations:

For parallel execution of independent steps, use multiple Task tool invocations in a single response. Each Task call should specify `subagent_type="general-purpose"` and include a detailed prompt with the specific mass point, process definition, and output directory.

## Output Organization

```
output/
├── paper_reproduction/
│   ├── README.md              # Summary report
│   ├── events/
│   │   ├── process1_bench1/   # MadGraph output
│   │   ├── process1_bench2/
│   │   └── ...
│   ├── analysis/
│   │   ├── analysis.py        # Analysis script
│   │   └── histograms.npz     # Computed histograms
│   └── plots/
│       ├── figure1.pdf        # Reproduced figures
│       ├── figure2.pdf
│       └── ...
```
