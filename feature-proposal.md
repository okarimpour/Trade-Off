# System Architecture Tool — Feature Proposal
### Integrating the Value-Driven Tradespace Exploration Workflow

---

## What the Workbook Does

The uploaded workbook (Adam Ross's Value-Driven Tradespace Quicktool) walks through a complete **Design-Value Loop** across three phases:

**Phase 1 — Frame the Value Model** (Week 2): Define the problem, decompose a value proposition into a hierarchy of goals/objectives/attributes, build Single Attribute Utility (SAU) curves for each attribute, assign MAU weights, and validate the model with test cases.

**Phase 2 — Generate & Evaluate the Design Space** (Week 3): Brainstorm design variables, score their impact on attributes via a Design-Value Mapping (DVM) matrix using a 0-1-3-9 scale, enumerate discrete levels for each design variable, auto-generate a full-factorial set of candidate designs (up to 5,000), run each design through parametric evaluation equations to compute attribute scores, then compute SAU and MAU for every design.

**Phase 3 — Explore & Select** (Week 4): Visualize all designs on a Cost vs. MAU tradespace scatterplot, identify the Pareto front, color-code points by design variable to reveal patterns, sweep weights for sensitivity analysis, and select the most promising alternatives with justification.

---

## How It Maps to Your App

Your app already has **Value Hierarchy** and **Trade-Off Analysis**. Below is a proposal for how the full workbook workflow could be absorbed as new tools on the landing page — each one self-contained but interconnected through shared data.


### 1. Enhance Value Hierarchy → "Attribute Definer"

**What changes:** Your existing Value Hierarchy builds the tree and assigns weights. The workbook also requires each leaf attribute to carry metadata: *units*, *acceptable range (min/max)*, *preferred direction (smaller/larger is better)*, and a *text definition*.

**How to simplify it:** When editing a leaf node in the tree, show additional fields below the weight input — units, min desirable, max acceptable, direction toggle. This data then flows downstream to every other tool automatically. No new landing page tile needed; it's just a richer leaf-node editor within the existing Value Hierarchy.

**Integration:** The Trade-Off tool already reads VH leaves as criteria. With the extra metadata, it could auto-populate the "Goal" row (maximize/minimize) and give meaning to the value ranges.


### 2. New Tool: "SAU Curve Editor"

**What it does:** For each attribute from the Value Hierarchy, the user defines a Single Attribute Utility curve — a mapping from raw attribute levels (e.g., 0–100 kg) to a 0–1 utility score. The workbook does this with a table of (level, SAU score) pairs and auto-generates a chart.

**Simplified version for the app:**
- Select a saved Value Hierarchy; the app lists its leaf attributes.
- For each attribute, provide an interactive graph canvas where the user clicks to place control points on a 2D grid (x = attribute level, y = 0–1 utility).
- Auto-draw the interpolated curve (linear segments or optional spline).
- Support a "quick preset" toggle: linear increasing, linear decreasing, exponential, S-curve — so users don't have to manually place every point.
- Validate monotonicity and range coverage.

**Integration:** SAU curves are saved per hierarchy and automatically used by the Tradespace tool (below) to convert raw attribute scores into utility scores.


### 3. New Tool: "Design-Value Map (DVM)"

**What it does:** A matrix where rows are *design variables* (the engineering choices you control) and columns are *attributes* (from the Value Hierarchy). Each cell gets a 0-1-3-9 impact score indicating how much that design variable drives that attribute.

**Simplified version for the app:**
- Select a saved Value Hierarchy to auto-populate the column headers with leaf attributes.
- Add design variable rows with name and units.
- Click each cell to toggle through 0 → 1 → 3 → 9 (color-coded: gray → blue → yellow → red).
- Auto-compute row totals (total impact of each DV) and column totals (total coverage of each attribute).
- A "Prune Weak Drivers" button that highlights DVs below a threshold, letting the user remove low-impact variables to keep the design space manageable.

**Integration:** The final set of design variables (with their enumeration levels) feeds directly into the Tradespace Generator.


### 4. New Tool: "Tradespace Generator"

**What it does:** This is the engine that takes design variables + an evaluation model and produces thousands of candidate designs with computed attribute scores and MAU values.

**Simplified version for the app (this is the biggest simplification opportunity):**

The workbook requires users to write Excel formulas to map DVs → attributes, which is the hardest and most error-prone part. The app could offer two modes:

**Mode A — Manual Scoring (lightweight):** For each design in the generated set, the user manually enters or estimates attribute scores. This works for small design spaces (< 50 alternatives) and is essentially an expanded version of the existing Trade-Off matrix.

**Mode B — Formula Builder (advanced):** A guided interface where the user defines simple relationships per attribute:
- "Attribute X = DV1 × constant + DV2" using a formula bar with autocomplete for DVs and constants.
- Support for intermediate variables.
- The app then auto-evaluates every design combination.

**Design space generation itself:**
- For each design variable, define discrete levels (e.g., Material: [Aluminum, Titanium, Composite]).
- Choose enumeration: full factorial, Latin Hypercube, or random sampling.
- Show design space size live ("3 × 4 × 5 = 60 designs").
- Cap at ~5,000 designs for browser performance.
- Auto-compute SAU (from curves defined in Tool 2) and MAU (weighted sum using VH weights).

**Integration:** Outputs the full design table that feeds into the Tradespace Explorer.


### 5. New Tool: "Tradespace Explorer"

**What it does:** The interactive visualization layer — the payoff of the entire workflow.

**Simplified version for the app:**
- **Scatterplot:** Cost (x-axis) vs. MAU (y-axis) for all generated designs. Each point is a candidate design. Built with Chart.js (already in your app).
- **Pareto Front:** One-click button to compute and highlight the Pareto-optimal designs (non-dominated set). Draw the Pareto curve as a connected overlay.
- **Color-by-DV:** Dropdown to color all points by a selected design variable, revealing clustering patterns (e.g., "all titanium designs cluster in the upper-right").
- **Hover/Click Inspection:** Hover a point to see its design variable values, attribute scores, and MAU. Click to pin it as a "point of interest."
- **Points of Interest Table:** A sidebar list of pinned designs for comparison and eventual selection.
- **Filter/Brush:** Drag a box on the scatterplot to zoom into a region; slider filters for attribute ranges.

**Integration:** Points of interest can be exported directly to the Trade-Off Analysis tool for deeper multi-method comparison (Pugh, MAU, etc.).


### 6. New Tool: "Sensitivity Analysis"

**What it does:** Tests how robust the rankings are when value model assumptions change.

**Simplified version for the app:**
- **Weight Sweep:** For each attribute, sweep its weight from 0 → 1 (redistributing the remainder proportionally among other attributes) and re-compute MAU for the pinned points of interest.
- **Line Chart:** One line per pinned design; x-axis = weight of the swept attribute, y-axis = MAU. Crossing lines mean rank reversals.
- **Robustness Score:** Auto-flag designs that maintain high MAU across all weight sweeps.
- **Threshold Sensitivity:** Show at what weight values the top-ranked design changes — answering "how wrong do my weights have to be before a different choice wins?"

**Integration:** Results feed back into the final design selection justification.


---

## Proposed Landing Page Layout

```
┌─────────────────────────────────────────────────────────┐
│              System Architecture Tool                    │
├─────────────┬─────────────┬─────────────┬───────────────┤
│   Value     │  SAU Curve  │  Design-    │  Tradespace   │
│  Hierarchy  │   Editor    │  Value Map  │  Generator    │
│  (existing) │   (new)     │   (new)     │   (new)       │
├─────────────┼─────────────┼─────────────┼───────────────┤
│ Trade-Off   │ Tradespace  │ Sensitivity │               │
│  Analysis   │  Explorer   │  Analysis   │               │
│ (existing)  │   (new)     │   (new)     │               │
└─────────────┴─────────────┴─────────────┴───────────────┘
```

The tools are ordered left-to-right, top-to-bottom following the Design-Value Loop workflow. Each tile could show a small status indicator (e.g., "3 hierarchies saved", "120 designs generated") to guide the user through the sequence.


---

## Data Flow Summary

```
Value Hierarchy ──→ SAU Curve Editor ──→ Tradespace Generator ──→ Tradespace Explorer
      │                    │                      ↑                       │
      │                    │               Design-Value Map               │
      │                    │              (design variables +             │
      ▼                    ▼               enumeration levels)            ▼
Trade-Off Analysis ←─────────────────────────────────────── Sensitivity Analysis
(existing, for final                                        (weight sweeps on
 deep-dive comparison)                                       pinned designs)
```

Everything is linked through shared Firestore collections per user. A hierarchy saved in Value Hierarchy is immediately available in SAU Curve Editor, DVM, and Trade-Off. Designs generated in the Tradespace Generator flow into the Explorer and Sensitivity tools. Points of interest can be sent to Trade-Off for final multi-method evaluation.


---

## Implementation Priority (Suggested)

| Priority | Tool                | Effort | Impact | Rationale |
|----------|---------------------|--------|--------|-----------|
| 1        | Attribute Definer   | Low    | High   | Enhances existing VH; prerequisite for everything |
| 2        | SAU Curve Editor    | Medium | High   | Core to the value model; visually engaging |
| 3        | Design-Value Map    | Medium | Medium | Simple matrix UI; familiar from Trade-Off |
| 4        | Tradespace Explorer | High   | High   | The "wow" visualization; can start with CSV import |
| 5        | Tradespace Generator| High   | High   | Complex but enables full automation |
| 6        | Sensitivity Analysis| Medium | Medium | Adds rigor; builds on Explorer data |

---

## Key Simplifications vs. the Workbook

The workbook is an educational tool that exposes every intermediate step. The app can abstract away much of the manual busywork:

1. **No manual formula entry.** The workbook makes users write Excel formulas to link design variables to attributes. The app can offer preset relationship templates (linear, polynomial, lookup table) or let advanced users enter formulas, while most users simply score alternatives directly.

2. **Automatic SAU/MAU computation.** Once curves and weights are defined, every downstream calculation happens automatically — no copy-paste between sheets.

3. **Interactive Pareto identification.** The workbook requires manually typing design IDs. The app computes and highlights the Pareto front with one click.

4. **Live sensitivity feedback.** Instead of static weight-sweep charts, the app can offer interactive sliders where dragging a weight immediately updates the tradespace visualization.

5. **Unified data model.** The workbook scatters data across 16 sheets with fragile cross-references. The app stores everything in a connected data model where changes propagate automatically.
