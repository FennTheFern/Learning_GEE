# Learning Google Earth Engine (Python) — Toward Building Your Own Glyphosate-Style Pipeline

This plan is scoped for **~4–7 hours/week**, aimed at someone with **no prior GEE experience** but solid ML/Python background. It's built around one throughline: by the end, you'll have written a scaled-down but real version of the change-detection front-end of the glyphosate spray-detection project — cleaned time series, harmonic seasonal baselines, per-polygon residual anomaly flags.

**Estimated total time:** ~11–13 hours of core content across 6 modules → roughly 2–3 weeks at your stated pace, or 1 focused week if you go heavier.

## Setup (do this first, ~20 min)

1. Sign up for Earth Engine access: https://code.earthengine.google.com/register (uses a Google Cloud project — free tier is enough for everything here)
2. `pip install earthengine-api geemap pandas numpy matplotlib`
3. Open `01_gee_fundamentals.ipynb` and run the auth cell — it'll walk you through the browser login flow once

## Modules

| # | Notebook | Core skill | Direct link to glyphosate project |
|---|----------|-----------|-----------------------------------|
| 1 | `01_gee_fundamentals.ipynb` | Client vs. server-side objects, `Image`/`ImageCollection`, `.getInfo()` | The mental model everything else depends on |
| 2 | `02_filtering_compositing.ipynb` | `.map()`, cloud masking, seasonal composites | Cleaning a year of imagery before analysis |
| 3 | `03_vegetation_indices_timeseries.ipynb` | NDVI/EVI, point time series → pandas | The atomic "index vs. date" extraction step |
| 4 | `04_zonal_stats_feature_collections.ipynb` | `FeatureCollection`, `reduceRegions` at scale | Extracting per-COMTRS-chunk stats without a Python loop |
| 5 | `05_harmonic_regression_baseline.ipynb` | `linearRegression` reducer, seasonal baseline + residuals, async `Export` | The expectation model your change-point detector needs |
| 6 | `06_capstone_anomaly_detection.ipynb` | End-to-end pipeline, recall-first flagging | A minimal, runnable version of the candidate-generation step |

Each notebook follows the same structure: **lesson** (concepts + a worked example), **exercises** (small, targeted), **project** (a reusable function you build yourself, meant to be lifted into your real codebase later).

## How to work through it

- Do the modules in order — 4 and 5 both lean hard on patterns from 1–3.
- Don't skip the exercises to jump to the project section; the exercises are deliberately small enough to expose gaps before the project makes them expensive to debug.
- Where a notebook says "swap in your real COMTRS asset," that's a deliberate seam — the toy data keeps notebooks runnable standalone, but the functions are written to be copy-pasteable into your actual project once you're ready.
- Module 6 has more TODOs and less hand-holding on purpose — treat it like real project work, not a tutorial.

## After the capstone

Two topics are relevant to your project but are more "general ML engineering" than "GEE-specific," so they're intentionally out of scope here — worth tackling as follow-ups once this curriculum is done:
- Multiple-instance learning framing (bags = COMTRS chunks, instances = pixels)
- Batching a true state-wide, multi-year export within GEE's request/quota limits (Module 5 sets up the concepts; the real COMTRS-scale version will need tiling by geography and/or date range)

## A note on GEE quirks you'll hit early

- `print()` on an `ee` object never shows pixel values — only `.getInfo()` does, and it always costs a network round trip. Use it sparingly, mainly for small summaries/metadata.
- Functions passed to `.map()` can't contain Python `if`/`for` over server-side values — use image/feature methods (`.where()`, `.expression()`, boolean band math) instead.
- `reduceRegion` (singular) is for one geometry; `reduceRegions` (plural) is for many geometries against one image — mixing these up is the most common performance trap for people scaling up their first pipeline.
