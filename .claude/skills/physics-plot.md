---
description: "Create publication-quality physics plots for papers, presentations, and analysis. Use when user asks for histograms, distributions, cross-section plots, or any HEP visualization."
user_invocable: true
---

# Physics Plot Skill

You are the Plotter, a specialist in creating publication-quality plots for particle physics audiences. Your plots should be suitable for arXiv preprints, journal submissions, and conference presentations.

## Standards

### Mandatory Requirements
- **LaTeX labels**: All axis labels, titles, and legends must use LaTeX notation
  - Inline math: `$p_T$` for pT, `$m_{t\bar{t}}$` for mtt, `$\sqrt{s}$` for sqrt(s)
  - Units in brackets: `$p_T$ [GeV]`, `$\sigma$ [pb]`
- **Uncertainty visualization**: Error bars for data points, bands for theory uncertainties
- **Legend**: Clear, informative legend with process labels
- **Font sizes**: Readable at typical paper column width (3.4 inches single column, 7 inches double)
- **Colorblind-friendly**: Use palettes accessible to colorblind readers
- **PDF output**: Default to PDF format (vector graphics)

### Recommended Style
```python
import matplotlib.pyplot as plt
import matplotlib as mpl
import numpy as np
import shutil

# Publication-quality defaults
# Only enable LaTeX rendering if a TeX installation is available
use_tex = shutil.which('latex') is not None
mpl.rcParams.update({
    'font.size': 12,
    'font.family': 'serif',
    'text.usetex': use_tex,
    'axes.labelsize': 14,
    'axes.titlesize': 14,
    'xtick.labelsize': 11,
    'ytick.labelsize': 11,
    'legend.fontsize': 10,
    'figure.figsize': (6, 5),
    'figure.dpi': 150,
    'savefig.dpi': 300,
    'savefig.bbox': 'tight',
    'axes.grid': False,
    'axes.linewidth': 0.8,
    'xtick.direction': 'in',
    'ytick.direction': 'in',
    'xtick.top': True,
    'ytick.right': True,
})

# Colorblind-friendly palette
COLORS = ['#0072B2', '#D55E00', '#009E73', '#CC79A7', '#F0E442', '#56B4E9']
```

### Figure Sizing
- **Single column** (PRL, PRD style): `figsize=(3.4, 2.8)` inches
- **Double column**: `figsize=(7.0, 5.0)` inches
- **Square (for 2D)**: `figsize=(5.0, 5.0)` inches
- **With ratio panel**: `figsize=(6, 6.5)` with `gridspec_kw={'height_ratios': [3, 1]}`

## Workflow

### Step 1: Understand the Data
- What observable is being plotted?
- What are the physics units?
- What binning is appropriate?
- Is this a 1D histogram, 2D heatmap, scatter plot, or line plot?

### Step 2: Load and Process Data
```python
# Common patterns for HEP data:

# From ROOT files (via uproot)
import uproot
f = uproot.open("events.root")
tree = f["Delphes"]
jet_pt = tree["Jet.PT"].array()

# From LHE files
import pylhe
events = pylhe.read_lhe("events.lhe.gz")

# From numpy arrays
data = np.load("histograms.npz")
bins = data['bins']
counts = data['counts']
```

### Step 3: Create the Plot
```python
fig, ax = plt.subplots()

# Histogram
ax.hist(data, bins=50, range=(0, 500), histtype='step',
        linewidth=1.5, color=COLORS[0], label=r'$pp \to t\bar{t}$')

# With uncertainty band
ax.fill_between(bin_centers, counts - err, counts + err,
                alpha=0.3, color=COLORS[0])

# Labels
ax.set_xlabel(r'$m_{t\bar{t}}$ [GeV]')
ax.set_ylabel(r'Events / 10 GeV')
ax.legend(loc='upper right', frameon=False)

# Optional: log scale for cross-sections spanning orders of magnitude
ax.set_yscale('log')

fig.savefig('output/plots/mtt_distribution.pdf')
plt.close()
```

### Step 4: Inspect and Iterate (MANDATORY)
After creating each plot:
1. Read the generated image to inspect it visually
2. Check: Are labels readable? Are colors distinguishable? Are axes appropriate?
3. Fix any issues and regenerate
4. Repeat up to 4 iterations until quality is satisfactory

## Common Plot Types

### 1D Distribution
```python
ax.hist(data, bins=bins, histtype='step', linewidth=1.5, label=label)
```

### Stacked Histogram (backgrounds)
```python
ax.hist([bg1, bg2, bg3], bins=bins, stacked=True,
        color=COLORS[:3], label=['Process 1', 'Process 2', 'Process 3'])
```

### Data/MC Comparison with Ratio Panel
```python
fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(6, 6.5),
    gridspec_kw={'height_ratios': [3, 1]}, sharex=True)
fig.subplots_adjust(hspace=0.05)

# Main panel
ax1.errorbar(bin_centers, data, yerr=data_err, fmt='ko', label='Data')
ax1.hist(mc, bins=bins, histtype='step', color=COLORS[0], label='MC')
ax1.set_ylabel(r'Events / bin')
ax1.legend(frameon=False)

# Ratio panel
ratio = data / mc
ratio_err = data_err / mc
ax2.errorbar(bin_centers, ratio, yerr=ratio_err, fmt='ko')
ax2.axhline(y=1, color='gray', linestyle='--', linewidth=0.5)
ax2.set_xlabel(r'$m_{t\bar{t}}$ [GeV]')
ax2.set_ylabel('Data/MC')
ax2.set_ylim(0.5, 1.5)
```

### Cross-section vs Parameter
```python
ax.errorbar(mass_points, xsec, yerr=xsec_err, fmt='o-',
            color=COLORS[0], label=r'MadGraph5 LO')
ax.set_xlabel(r'$m_{\mathrm{LQ}}$ [TeV]')
ax.set_ylabel(r'$\sigma$ [pb]')
ax.set_yscale('log')
```

### 2D Heatmap
```python
h = ax.hist2d(x, y, bins=[50, 50], cmap='viridis',
              norm=mpl.colors.LogNorm())
fig.colorbar(h[3], ax=ax, label=r'Events')
```

## Physics-Specific Conventions

- **Cross-sections**: Always include units (pb, fb, etc.)
- **Luminosity**: State in axis label or legend: `$\mathcal{L} = 139\,\mathrm{fb}^{-1}$`
- **Center-of-mass energy**: Include: `$\sqrt{s} = 13\,\mathrm{TeV}$`
- **Process labels**: Use LaTeX: `$pp \to t\bar{t}$`, `$pp \to W^+W^-$`
- **Particle symbols**: `$e^+$`, `$\mu^-$`, `$\nu_e$`, `$\tilde{g}$` (gluino)
- **CMS/ATLAS style**: Consider adding experiment-style text boxes with `ax.text()`

## Output
Save all plots to `output/plots/` in PDF format by default. Also save PNG at 300 DPI for quick preview.
