# Why Microsoft PowerPoint Drops SVG Arrowheads & Shifts Text Baselines (and How to Fix It)

Standard SVGs generated for web browsers frequently break in two predictable ways when inserted into **Microsoft PowerPoint** (`Insert -> Pictures -> This Device... -> Convert to Shape`) or design importers:

---

## Problem 1: `<marker>` Arrowheads Vanish

Browsers render arrowheads defined in `<defs><marker id="arrow">...</marker></defs>` via `marker-end="url(#arrow)"`:

```xml
<defs>
  <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
    <path d="M 0 1 L 10 5 L 0 9 z" fill="#1D4ED8" />
  </marker>
</defs>
<line x1="40" y1="80" x2="220" y2="80" stroke="#1D4ED8" stroke-width="2" marker-end="url(#arrow)" />
```

PowerPoint's SVG parser has incomplete support for CSS/URL `<marker>` references, often dropping arrowheads silently when importing or ungrouping into native Office shapes.

### Fix: Flatten Markers into Explicit Polygons

Compute the endpoint tangent angle, shorten the shaft slightly so the stroke cap does not protrude past the tip, and emit an explicit filled `<polygon>`:

```xml
<g data-role="connector">
  <line x1="40" y1="80" x2="210" y2="80" stroke="#1D4ED8" stroke-width="2" />
  <polygon points="220,80 208,74 208,86" fill="#1D4ED8" />
</g>
```

---

## Problem 2: Vertical Baseline Drift (`dominant-baseline`)

Browsers center box labels vertically using `dominant-baseline="central"` or `dominant-baseline="middle"`. PowerPoint frequently ignores `dominant-baseline` and falls back to `alphabetic` baseline, causing labels inside pills and nodes to sink toward the bottom border.

### Fix: Bake Vertical Centering Directly into `y`

Convert centered baselines directly into explicit `alphabetic` baseline coordinates (`y_alphabetic ≈ y_center + 0.35 * font_size`) and explicit `x` / `y` coordinates on multi-line `<tspan>` nodes.

---

## Automated One-Click Compatibility Export

You can generate both standard SVG and PowerPoint/Figma compatibility packages automatically using the **[Image to PowerPoint & Figma Vector workflow](https://fig2svg.online/image-to-powerpoint-vector)** on **[Fig2SVG](https://fig2svg.online/)**. On the Downloads screen, toggle **PowerPoint & Figma compatibility mode** before downloading your SVG or ZIP bundle.
