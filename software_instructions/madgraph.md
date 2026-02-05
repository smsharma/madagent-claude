# MadGraph5_aMC@NLO Reference Guide

This document provides comprehensive instructions for working with MadGraph5_aMC@NLO, the standard Monte Carlo event generator for high-energy physics simulations.

## 1. Getting Started

### Launching MadGraph

**Interactive mode** (preferred for agent workflows):
```bash
<MadGraph_Path>/bin/mg5_aMC
```
This opens the MadGraph interactive prompt where you enter commands sequentially.

**Script mode** (for batch processing):
```bash
<MadGraph_Path>/bin/mg5_aMC script.mg5
```
Where `script.mg5` contains MadGraph commands, one per line.

### Key Paths
- MadGraph installation: typically `/opt/MG5_aMC` or `$HOME/MG5_aMC`
- Python environment: `/opt/envs/MAD` (container) or system Python
- Output directory: `/output` (container) or `./output`

## 2. Model Management

### Importing Models
```
import model sm              # Standard Model (default)
import model sm-no_b_mass    # SM with massless b-quarks
import model heft            # Higgs Effective Field Theory
import model mssm            # Minimal Supersymmetric SM
import model <MODEL_NAME>    # Any installed/downloadable model
```

### Inspecting Models
```
display modellist            # List available models
display particles            # Show all particles in current model
display multiparticles       # Show defined multiparticle labels
display parameters           # Show model parameters
display interactions         # Show available interaction vertices
display diagrams             # Show Feynman diagrams for current process
```

### Default Multiparticle Labels (SM)
- `p` / `j` = light quarks + gluon: {g, u, d, s, c, u~, d~, s~, c~}
- `l+` = {e+, mu+, ta+}
- `l-` = {e-, mu-, ta-}
- `vl` = {ve, vm, vt}
- `vl~` = {ve~, vm~, vt~}

### Defining Custom Multiparticle Labels
```
define q = u d s c b          # All quarks
define q~ = u~ d~ s~ c~ b~   # All anti-quarks
define ll = e+ e- mu+ mu-    # Charged leptons (no tau)
```

## 3. Process Definition

### Basic Syntax
```
generate <INITIAL_STATE> > <FINAL_STATE>
```

### Examples

**Leading order (LO):**
```
generate p p > t t~            # Top pair production
generate p p > w+ w-           # W pair production
generate p p > e+ ve mu- vm~   # WW to leptons
generate e+ e- > mu+ mu-      # Drell-Yan at lepton collider
```

**Adding subprocesses:**
```
generate p p > t t~
add process p p > t t~ j       # +1 jet
add process p p > t t~ j j     # +2 jets
```

**With decay chains:**
```
generate p p > t t~, (t > w+ b, w+ > e+ ve), (t~ > w- b~, w- > mu- vm~)
```

### Coupling Constraints

Control the perturbative content of the process:

```
generate p p > e+ ve QCD=0              # Pure electroweak
generate p p > t t~ QED=0               # Pure QCD
generate p p > w+ j QED=2 QCD=1         # Mixed with explicit orders
generate p p > e+ e- / z QED=2          # Exclude Z-mediated diagrams
```

**Key rules:**
- `QED=N` limits the maximum power of alpha_EW
- `QCD=N` limits the maximum power of alpha_S
- Without constraints, MadGraph generates all diagrams at the lowest combined order
- Use `/` to exclude intermediate particles: `p p > e+ e- / z` removes Z contributions

### NLO Corrections

Specify NLO contributions using bracket notation:

```
generate p p > t t~ [QCD]           # NLO QCD corrections to ttbar
generate p p > w+ [QCD]             # NLO QCD to W production
generate p p > e+ ve [QED]          # NLO EW corrections
generate p p > t t~ [QCD] [QED]     # Mixed NLO
```

**Loop-induced processes:**
```
generate g g > h [noborn=QCD]       # gg->H (loop-induced, no Born)
generate g g > h h [virt=QCD]       # Virtual corrections only
```

**Important NLO notes:**
- NLO generation requires `aMCatNLO` (included in MG5_aMC)
- Must have appropriate loop libraries (MadLoop)
- Showering is mandatory for NLO (MC@NLO matching)

## 4. Process Output

### Creating the Process Directory
```
output <PROC_DIR>               # e.g., output pp_tt
output <PROC_DIR> -nojpeg       # Skip diagram rendering
```

This creates the directory structure:
```
<PROC_DIR>/
├── Cards/
│   ├── run_card.dat          # Run parameters
│   ├── param_card.dat        # Model parameters
│   ├── proc_card_mg5.dat     # Process definition record
│   ├── me5_configuration.txt # Internal configuration
│   ├── pythia8_card.dat      # Pythia8 settings (if enabled)
│   └── delphes_card.dat      # Delphes settings (if enabled)
├── Events/                    # Will contain run results
├── SubProcesses/             # Generated code
├── Source/                   # Support code
├── bin/                      # Executables
│   ├── generate_events       # Event generation script
│   └── madevent              # Main executable
└── HTML/                     # Diagnostic webpages
```

## 5. Launch and Run Configuration

### Basic Launch
```
launch <PROC_DIR>                    # Standard launch
launch <PROC_DIR> -n my_run          # Custom run name
```

### Launch Menu Options
After `launch`, MadGraph presents switch options:

```
1. shower/hadronization (Pythia8)     [ON/OFF]
2. detector simulation (Delphes)      [ON/OFF]
3. decay (MadSpin)                    [ON/OFF]
```

Toggle with:
```
shower = ON        # or Pythia8 = ON
detector = ON      # or Delphes = ON
madspin = ON       # or decay = ON
```

### Card Editing Methods

**Method 1: Interactive `set` commands (preferred for agents)**
```
set nevents 10000
set ebeam1 6500
set ebeam2 6500
set pdlabel lhapdf
set lhaid 303400           # NNPDF31_nnlo_as_0118
set ptj 20                 # Minimum jet pT cut
set etaj 5.0               # Maximum jet eta cut
set mt 172.5               # Top mass in param_card
set wt auto                # Auto-calculate top width
```

**Method 2: Direct file editing**
Edit the card files directly before launching, or specify paths:
```
<path_to_custom_run_card>
<path_to_custom_param_card>
```

**Method 3: Using `0` to accept defaults**
```
0   # Accept all current settings and start generation
```

### Key run_card.dat Parameters

| Parameter | Description | Typical Value |
|-----------|-------------|---------------|
| `nevents` | Number of events to generate | 10000-1000000 |
| `ebeam1`, `ebeam2` | Beam energies (GeV) | 6500 (13 TeV LHC) |
| `pdlabel` | PDF set type | `lhapdf` or `nn23lo1` |
| `lhaid` | LHAPDF grid ID | 303400 (NNPDF3.1) |
| `iseed` | Random seed | 0 (auto) |
| `ptj` | Min jet pT (GeV) | 20.0 |
| `etaj` | Max jet |eta| | 5.0 |
| `ptl` | Min lepton pT (GeV) | 10.0 |
| `etal` | Max lepton |eta| | 2.5 |
| `drjj` | Min deltaR(jet,jet) | 0.4 |
| `drll` | Min deltaR(lep,lep) | 0.4 |
| `mmll` | Min m(l+l-) (GeV) | 0.0 |
| `use_syst` | Systematic variations | True |
| `scalefact` | Scale factor | 1.0 |
| `dynamical_scale_choice` | Scale choice | -1 (default) |

### Key param_card.dat Parameters

The param_card contains all model parameters. For the SM:
- Block `MASS`: particle masses (mt, mw, mz, mh, etc.)
- Block `DECAY`: particle widths (set to `auto` for automatic calculation)
- Block `YUKAWA`: Yukawa couplings
- Block `SMINPUTS`: SM inputs (alpha_EW, G_F, alpha_S)

```
set mt 172.5        # Top quark mass
set mh 125.0        # Higgs mass
set wt auto         # Auto-compute top width
set ww auto         # Auto-compute W width
set wz auto         # Auto-compute Z width
set wh auto         # Auto-compute Higgs width
```

## 6. Pythia8 Showering

When shower is enabled, Pythia8 performs:
- Parton showering (ISR + FSR)
- Hadronization (Lund string model)
- Underlying event
- Particle decays

### Key Pythia8 Settings (pythia8_card.dat)
```
! Tune selection
Tune:pp = 14                  # Monash 2013 tune

! PDF settings
PDF:pSet = LHAPDF6:NNPDF31_lo_as_0130

! Jet matching (for MLM-matched samples)
JetMatching:merge = on
JetMatching:scheme = 1
JetMatching:setMad = on
JetMatching:qCut = 30.0      # Matching scale in GeV

! Process-specific
TimeShower:QEDshowerByL = on  # QED radiation from leptons
```

### Output with Showering
Events are stored in:
- `<PROC_DIR>/Events/run_01/tag_1_pythia8_events.hepmc.gz` (HepMC format)
- Or LHE format pre-showering: `<PROC_DIR>/Events/run_01/unweighted_events.lhe.gz`

## 7. Delphes Detector Simulation

When detector simulation is enabled, Delphes provides fast simulation of:
- Tracking (with configurable efficiency and resolution)
- Calorimetry (ECAL + HCAL)
- Particle flow
- Jet clustering (anti-kT, R=0.4 by default)
- b-tagging
- Tau-tagging
- Missing transverse energy

### Output Format
ROOT file: `<PROC_DIR>/Events/run_01/tag_1_delphes_events.root`

### Accessing Delphes Output
```python
import ROOT
f = ROOT.TFile.Open("tag_1_delphes_events.root")
tree = f.Get("Delphes")

# Or using uproot (no ROOT installation needed):
import uproot
f = uproot.open("tag_1_delphes_events.root")
tree = f["Delphes"]
jets = tree["Jet"].arrays()
```

### Key Delphes Branches
- `Jet` -- Reconstructed jets (pT, eta, phi, mass, BTag, TauTag)
- `Electron` -- Reconstructed electrons
- `Muon` -- Reconstructed muons
- `Photon` -- Reconstructed photons
- `MissingET` -- Missing transverse energy
- `Particle` -- Generator-level particles (truth)
- `Track` -- Reconstructed tracks

## 8. MadSpin (Spin-Correlated Decays)

MadSpin preserves spin correlations in particle decays. Use when:
- Decay chains are too complex for direct specification in `generate`
- You need spin correlations but direct generation is computationally expensive

### Configuration
In the MadSpin card (`madspin_card.dat`):
```
set max_weight_ps_point 400
set Nevents_for_max_weight 75
set BW_cut 15.0
set spinmode onshell         # or 'full' for off-shell effects

decay t > w+ b, w+ > l+ vl
decay t~ > w- b~, w- > l- vl~
```

## 9. Common Physics Processes

### Standard Model
| Process | MadGraph Command |
|---------|-----------------|
| Drell-Yan | `generate p p > e+ e-` |
| W+jets | `generate p p > w+ j, w+ > l+ vl` |
| Z+jets | `generate p p > z j, z > l+ l-` |
| Top pair | `generate p p > t t~` |
| Single top (t-channel) | `generate p p > t j $$ w+ w-` |
| WW production | `generate p p > w+ w-` |
| ZZ production | `generate p p > z z` |
| Higgs (ggF) | `generate g g > h [noborn=QCD]` |
| Higgs (VBF) | `generate p p > h j j QCD=0` |
| ttH | `generate p p > t t~ h` |

### BSM Examples
| Process | MadGraph Command |
|---------|-----------------|
| SUSY squark pair | `generate p p > go go` (after `import model mssm`) |
| EFT (dim-6) | `generate p p > t t~ NP=1` (after `import model SMEFTsim_...`) |
| Extra Higgs | `generate p p > h2` (after importing appropriate model) |
| Leptoquark pair | `generate p p > lq lq~` (after importing LQ model) |

## 10. Advanced Features

### Systematic Variations
Enable in run_card: `use_syst = True`
This generates event weights for scale and PDF variations.

### Multi-run Campaigns
```python
# In script mode, loop over parameter points:
import model sm
generate p p > t t~
output pp_tt

# Launch with different masses
launch pp_tt -n run_mt170
set mt 170.0
0

launch pp_tt -n run_mt172
set mt 172.5
0

launch pp_tt -n run_mt175
set mt 175.0
0
```

### Decay Chain Syntax Details
```
# Direct decay specification
generate p p > t t~, (t > w+ b), (t~ > w- b~)

# Nested decays
generate p p > t t~, (t > w+ b, w+ > e+ ve), (t~ > w- b~, w- > mu- vm~)

# Inclusive (all decays)
generate p p > t t~   # Then use MadSpin for decays
```

### Process Exclusion
```
generate p p > e+ e- / a        # Exclude photon-mediated
generate p p > t j $$ w+ w-     # Exclude W from s-channel
```
- `/` excludes particle from s-channel propagators
- `$$` excludes particle from both s- and t-channel

### Merging and Matching
For combining different jet multiplicities (MLM matching):
```
generate p p > z
add process p p > z j
add process p p > z j j
output pp_z_012j

launch pp_z_012j
set ickkw 1                    # Enable MLM matching
set xqcut 20                   # Matching scale
```

## 11. Troubleshooting

### Common Issues
| Issue | Solution |
|-------|---------|
| "No process found" | Check particle names, coupling orders, model |
| "Process too complex" | Reduce final-state particles, use decay chains |
| Negative cross-section events | Normal for NLO; increase statistics |
| Slow generation | Reduce phase space cuts, increase max diagram count |
| Memory issues | Reduce number of subprocesses, split into jobs |
| PDF not found | Install via `lhapdf install NNPDF31_lo_as_0130` |

### Useful Commands
```
display processes               # Show defined processes
display diagrams                # Show Feynman diagrams
history                        # Show command history
help <command>                 # Get help on specific command
```

## 12. Output Inspection

### Cross-section Results
After generation, MadGraph prints:
```
Cross-section :   XXX.X +- X.X pb
```

Results are also stored in:
- `<PROC_DIR>/Events/run_01/run_01_tag_1_banner.txt` (full run banner)
- `<PROC_DIR>/crossx.html` (summary webpage)

### Event Files
- **LHE** (Les Houches Events): `unweighted_events.lhe.gz` -- standard HEP event format
- **HepMC**: `tag_1_pythia8_events.hepmc.gz` -- after Pythia showering
- **ROOT**: `tag_1_delphes_events.root` -- after Delphes detector sim

### Reading LHE Files
```python
import gzip
import xml.etree.ElementTree as ET

with gzip.open('unweighted_events.lhe.gz', 'rt') as f:
    content = f.read()

# Or using pylhe:
import pylhe
events = pylhe.read_lhe('unweighted_events.lhe.gz')
for event in events:
    for particle in event.particles:
        print(particle.id, particle.px, particle.py, particle.pz, particle.e)
```
