---
name: gaussian-vs-laplace-model-comparison
description: Use when you have per-sample alpha diversity (e.g. observed richness)
  plus an environmental covariate (pH, temperature) and need to locate the richness
  optimum and decide whether a Laplace (two-sided exponential) or Gaussian shape better
  describes the gradient.
when_to_use_negative:
- Samples lack pH or temperature metadata — the gradient cannot be binned or fit
- The analysis targets host-associated samples, where the pH ~7 / ~10 °C richness
  optimum was not established
- Input is already a per-sample richness vector without the underlying feature table,
  so the mc2 filter (minimum sequence count in an OTU equal to 2) cannot be applied
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Analysis language for loading the BIOM table/mapping file, binning, and fitting
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Pandas
  role: Loads and filters the mapping file (pH, temperature) and joins it to the Deblur
    BIOM table
- name: Deblur
  role: Produced the exact-sequence (V4, post-515f) feature table whose per-sample
    richness is modeled
  repo: https://github.com/cuttlefishh/deblur
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:b84fc0a93327ee9a3629299c8068a1df5afd95e35c102f3e26a0e81fb52b9d3b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - Pandas
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gaussian-vs-laplace-model-comparison

## Summary

Fits a two-sided exponential (Laplace) richness = a*exp(-|x-x0|/b) model to binned microbial richness gradients and benchmarking it against a Gaussian fit, to locate the environmental optimum (pH, temperature) where community richness peaks.

## When to use

Use when you have per-sample alpha diversity (e.g. observed richness) plus an environmental covariate (pH, temperature) and need to locate the richness optimum and decide whether a Laplace (two-sided exponential) or Gaussian shape better describes the gradient.

## When NOT to use

- Samples lack pH or temperature metadata — the gradient cannot be binned or fit
- The analysis targets host-associated samples, where the pH ~7 / ~10 °C richness optimum was not established
- Input is already a per-sample richness vector without the underlying feature table, so the mc2 filter (minimum sequence count in an OTU equal to 2) cannot be applied

## Inputs

- QC-filtered EMP 16S Release 1 Deblur BIOM table (exact-sequence feature table)
- EMP mapping file with pH and temperature metadata per sample
- Non-host-associated sample subset (mostly freshwater and soil)

## Outputs

- Fitted Laplace parameters (a, x0, b) for richness vs. binned pH and vs. binned temperature
- Gaussian fit parameters on the same binned data for comparison
- Binned 99th-percentile richness points with both fitted curves, plotted as richness_gradient_laplace_fits.png
- Peak locations (expected ~pH 7 and ~10 °C) with relative goodness-of-fit (R²/residual sum of squares)

## How to apply

Load the QC-filtered EMP 16S Release 1 Deblur BIOM table and mapping file with Pandas, apply the mc2 filter (minimum sequence count in an OTU equal to 2), and restrict to samples carrying pH and temperature metadata before computing per-sample observed richness. Bin samples by pH into 100 equal bins and by temperature into 120 equal bins, then compute the 99th-percentile richness within each bin so the fit tracks the upper envelope rather than the mean. Fit richness = a*exp(-|x-x0|/b) to the binned pH and temperature points with scipy.optimize.curve_fit (nonlinear least squares), and fit a Gaussian to the same binned points as the competing model. Judge the result by comparing relative goodness of fit (e.g. R² / residual sum of squares) — the Laplace must outperform the Gaussian — and by checking that the fitted peaks land near pH 7 and ~10 °C, consistent with the Fig. 2b result for non-host-associated EMP samples. Save the binned points with both fitted curves as richness_gradient_laplace_fits.png.

## Related tools

- **Python** (Analysis language for loading the BIOM table/mapping file, binning, and fitting) — https://github.com/biocore/emp/tree/master/methods
- **Pandas** (Loads and filters the mapping file (pH, temperature) and joins it to the Deblur BIOM table)
- **Deblur** (Produced the exact-sequence (V4, post-515f) feature table whose per-sample richness is modeled) — https://github.com/cuttlefishh/deblur

## Examples

```
from scipy.optimize import curve_fit; popt, pcov = curve_fit(lambda x, a, x0, b: a*np.exp(-abs(x-x0)/b), ph_bin_centers, p99_richness, p0=[1000, 7.0, 2.0])
```

## Evaluation signals

- Laplace peak location x0 falls near pH 7 for the pH gradient and near ~10 °C for the temperature gradient (Fig. 2b consistency check)
- Laplace fit achieves lower residual sum of squares (higher R²) than the Gaussian fit on the same binned points
- Binning used exactly 100 equal bins for pH and 120 equal bins for temperature, with 99th-percentile richness per bin
- Per-sample richness was computed after the mc2 filter and restricted to samples with both pH and temperature metadata
- Output figure richness_gradient_laplace_fits.png shows binned points overlaid with both fitted curves

## Limitations

- Model conclusions are specific to non-host-associated EMP samples (mostly freshwater and soil) in EMP 16S Release 1; host-associated environments were not covered by this finding
- Using 99th-percentile richness within equal-width bins means bin count (100 for pH, 120 for temperature) and bin boundaries influence the fit; the workflow does not report sensitivity to these choices
- The two-sided exponential is symmetric by construction (|x-x0|), so asymmetric richness gradients around the optimum cannot be captured

## Evidence

- [methods] Laplace function and binning of pH/temperature gradients: [evidence span withheld — anchor status: not_found]
- [methods] Laplace model form fitted by nonlinear least squares: [evidence span withheld — anchor status: not_found]
- [methods] mc2 filter definition applied before richness computation: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [results] Biological finding the fit should reproduce: [evidence span withheld — anchor status: not_found]
- [methods] EMP 16S Release 1 data products used as input: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results"
