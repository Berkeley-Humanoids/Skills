---
name: paper-figures
description: Produce publication-ready scientific figures that are clear, compact, accessible, and visually consistent. Use this skill whenever the user is making a figure, plot, chart, panel, or schematic for a paper, manuscript, preprint, thesis, poster, or conference submission — including matplotlib/seaborn/matplotlib-style plotting code, multi-panel figure layouts, colormap or palette choices, font/line/marker sizing, or figure export settings. Trigger it even when the user just says "make a plot for my paper," "clean up this figure," "what colormap should I use," or pastes plotting code to improve, and especially when targeting Nature, Science, IEEE, NeurIPS, ICML, or arXiv. Do not wait for the words "publication-ready."
---

# Paper Figures

Make figures readable, quiet, and consistent. Every axis, label, and color must earn its place. Build at **final print size** from the start — never make oversized plots and shrink them, or fonts, line weights, and spacing drift out of spec.

Apply one visual language across all figures in a paper: same font, sizes, line weights, marker scale, palette, panel labels, and export settings. Nature rules below are the strict baseline; adapt dimensions and export format to the actual venue.

## Start here: matplotlib defaults

Set these once, then tune per figure. They encode most of the rules below.

```python
import matplotlib as mpl

MM = 1 / 25.4
SINGLE_COL = 89 * MM    # ~89-90 mm, single column
DOUBLE_COL = 183 * MM   # ~180-183 mm, double column
MAX_HEIGHT = 170 * MM   # leave room for caption

mpl.rcParams.update({
    "font.family": "Arial",        # or Helvetica
    "font.size": 6,
    "axes.labelsize": 6, "axes.titlesize": 7,
    "xtick.labelsize": 5, "ytick.labelsize": 5,
    "legend.fontsize": 5,
    "axes.linewidth": 0.6,
    "xtick.major.width": 0.5, "ytick.major.width": 0.5,
    "xtick.major.size": 2.5, "ytick.major.size": 2.5,
    "lines.linewidth": 1.0, "lines.markersize": 3.5,
    "pdf.fonttype": 42, "ps.fonttype": 42,   # keep text editable
    "savefig.dpi": 300,
})

fig, ax = plt.subplots(figsize=(SINGLE_COL, SINGLE_COL * 0.65))
ax.spines[["top", "right"]].set_visible(False)  # drop unless frame means something
```

## Layout

- Single column ≈ 89 mm; double column ≈ 183 mm; full-page max height ≈ 170 mm; extended-data page ≈ 183 × 247 mm. Use the venue's exact widths.
- Arrange panels in reading order (left→right, top→bottom). Label them lowercase **a, b, c**.
- Size panels by information density — a schematic shouldn't claim the same area as a complex quantitative plot.
- Align panel edges, use consistent gutters, share axes where comparable. Minimize whitespace without letting labels collide.

## Typography

- Sans-serif, editable (Arial/Helvetica). Don't outline text unless the venue demands it.
- Axis/tick/legend/annotation text 5–7 pt; panel labels 8 pt bold; in-figure table text ~7 pt; sequences/code in Courier or similar.
- Keep text black or dark gray. Use sentence case, no trailing full stops on labels.
- Color labels via swatches/keylines/direct annotation, not colored text.
- Put units in the axis label: `Torque (N m)`, `Velocity (rad s⁻¹)`, `Success rate (%)`.

## Lines, markers, grids

- Hairlines/subtle gridlines 0.25–0.4 pt; axes/ticks 0.5–0.75 pt; data lines 0.75–1.25 pt; schematic outlines 0.75–1.5 pt.
- Markers 2.5–5 pt — large enough to survive reduction. Drop top/right spines on standard 2D plots.
- Use error bars, confidence bands, and transparency to clarify uncertainty, not to hide data.

## Color

- RGB unless the venue requires CMYK. Use colorblind-safe palettes; never rely on color alone — add labels, symbols, or line styles. The figure should still read in grayscale.
- **Default qualitative palette** (Okabe–Ito):

```text
Black #000000   Orange #E69F00   Sky blue #56B4E9   Bluish green #009E73
Yellow #F0E442  Blue #0072B2     Vermillion #D55E00  Purple #CC79A7
```

- Continuous data: perceptually uniform maps (viridis, cividis, batlow, Crameri). Avoid jet/rainbow and red/green-only contrasts.
- Mute secondary data; reserve strong contrast for the main comparison.

## Per-plot guidance

- **Line:** label lines directly, avoid crowded legends, share axes across comparable panels.
- **Bar:** prefer dot/box/violin/interval plots when distributions or individual samples matter; start y at zero unless a truncated axis is explicitly justified; show sample size.
- **Scatter:** use transparency, jitter, density contours, or hex bins for dense data; keep regression lines secondary unless they are the result.
- **Images/microscopy:** scale bars not magnification factors; consistent crops, contrast, and annotation; keep labels editable, don't flatten text into raster.
- **Schematics:** limited visual vocabulary, grid-aligned objects, consistent arrows; no gradients, shadows, 3D, or decorative textures.

## Export

- Prefer vector: PDF, SVG, EPS. Raster only when necessary: TIFF/PNG/JPEG at ≥300 dpi at final size (never upscale). Keep `pdf.fonttype=42` so text stays editable.

```python
plt.savefig("figure.pdf", bbox_inches="tight", pad_inches=0.03)
plt.savefig("figure.png", dpi=300, bbox_inches="tight", pad_inches=0.03)
```

## Review checklist

- Built at final size; all text readable; font family/size consistent; vector text editable.
- Panel labels lowercase bold and consistently placed; axes carry units.
- Colorblind-safe; legible in grayscale; no red/green-only or unjustified rainbow.
- Line weights neither hairline-invisible nor heavy; panels aligned and logically ordered.
- Whitespace tight but not cramped; legends compact or replaced by direct labels.
- Raster ≥300 dpi; scale bars present where needed; export format and file size match venue limits.
- The main claim reads from the figure without leaning on the caption.

## Resources

- Nature artwork guide: https://www.nature.com/documents/natrev-artworkguide.pdf
- Nature figure specs: https://research-figure-guide.nature.com/figures/preparing-figures-our-specifications/
- Nature panel building/export: https://research-figure-guide.nature.com/figures/building-and-exporting-figure-panels/
- Nature Extended Data guide: https://research-figure-guide.nature.com/figures/extended-data-formatting-guidelines/
- Nature-style matplotlib reference: https://github.com/hoanglongcao/nature-plot-style
