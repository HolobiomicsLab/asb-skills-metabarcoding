---
name: percentile-summary-statistics
description: Use when modeling a response variable (e.g. microbial richness) against
  noisy continuous covariates (pH, temperature) where per-sample values are scattered
  and a robust upper-envelope per covariate bin is needed before curve fitting.
when_to_use_negative:
- Samples lack pH or temperature metadata, so the covariate bins cannot be populated
- The analysis requires per-sample (not binned envelope) richness values, e.g. alpha-diversity
  group comparisons between studies
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3305
tools:
- name: Pandas
  role: Loads the BIOM table and mapping file and computes per-sample observed richness
    and binned percentile summaries
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Python
  role: Implements binning, percentile computation, and nonlinear least-squares curve
    fitting (scipy.optimize.curve_fit) for the Laplace and Gaussian models
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Deblur
  role: Produced the exact-sequence OTU table whose per-sample observed richness is
    summarized via percentiles
  repo: https://github.com/cuttlefishh/deblur
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:00ae696e55359fb636edb8cea2a64031bec4e2041581697007fff4dfc37b668a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# percentile-summary-statistics

## Summary

Summarize noisy environmental gradients by computing per-bin high-percentile statistics (e.g. 99th-percentile observed richness within pH and temperature bins) before model fitting, as used to reproduce the EMP Laplace richness-gradient analysis (Fig. 2b of Thompson et al. 2017).

## When to use

Use when modeling a response variable (e.g. microbial richness) against noisy continuous covariates (pH, temperature) where per-sample values are scattered and a robust upper-envelope per covariate bin is needed before curve fitting.

## When NOT to use

- Samples lack pH or temperature metadata, so the covariate bins cannot be populated
- The analysis requires per-sample (not binned envelope) richness values, e.g. alpha-diversity group comparisons between studies

## Inputs

- EMP 16S Release 1 QC-filtered Deblur BIOM table (per-sample OTU counts)
- EMP merged and QC-filtered mapping file with pH and temperature metadata fields
- Bin count parameters (100 bins for pH, 120 bins for temperature) and percentile level (99th)

## Outputs

- Per-bin 99th-percentile richness values for pH and temperature gradients
- Fitted Laplace (two-sided exponential) and Gaussian curve parameters (a, x0, b) for each gradient
- Goodness-of-fit comparison (e.g. R²/residual sum of squares) between Laplace and Gaussian
- Plot of binned points with both fitted curves (richness_gradient_laplace_fits.png)

## How to apply

Load the QC-filtered EMP 16S Release 1 Deblur BIOM table and its mapping file into Python with Pandas, apply the mc2 minimum-sequence-count filter (minimum sequence count in an OTU equal to 2), and restrict to samples that have pH and temperature metadata. Compute per-sample observed richness, then bin samples by pH into 100 equal bins and by temperature into 120 equal bins, computing the 99th-percentile richness within each bin to obtain a robust upper envelope of the gradient. Fit richness = a*exp(-|x-x0|/b) to the binned pH and temperature data with scipy.optimize.curve_fit, and fit a Gaussian curve on the same binned points for comparison, evaluating relative goodness of fit (e.g. R²/residual sum of squares) to confirm the Laplace model outperforms the Gaussian. The binned percentile points and both fitted curves are then plotted (richness_gradient_laplace_fits.png) and the Laplace peak locations should fall near pH 7 and ~10 °C, consistent with the paper's result that non-host-associated richness peaks near neutral pH and about 10 °C.

## Related tools

- **Pandas** (Loads the BIOM table and mapping file and computes per-sample observed richness and binned percentile summaries) — https://github.com/biocore/emp/tree/master/methods
- **Python** (Implements binning, percentile computation, and nonlinear least-squares curve fitting (scipy.optimize.curve_fit) for the Laplace and Gaussian models) — https://github.com/biocore/emp/tree/master/methods
- **Deblur** (Produced the exact-sequence OTU table whose per-sample observed richness is summarized via percentiles) — https://github.com/cuttlefishh/deblur

## Examples

```
from scipy.optimize import curve_fit; laplace = lambda x, a, x0, b: a*np.exp(-np.abs(x-x0)/b); popt, _ = curve_fit(laplace, bin_centers, pct99_richness, p0=[max(pct99_richness), 7.0, 1.0])
```

## Evaluation signals

- Laplace peak location x0 falls near pH 7 for the richness-versus-pH fit and near ~10 °C for the richness-versus-temperature fit
- Laplace fit yields a better relative goodness of fit (lower residual sum of squares / higher R²) than the Gaussian on the same binned points
- Only samples with both pH and temperature metadata are included after the mc2 filter (minimum sequence count in an OTU equals 2)
- Binned structure matches the specified bin counts: 100 equal pH bins and 120 equal temperature bins, each summarized by the 99th-percentile richness
- Output figure richness_gradient_laplace_fits.png contains both the binned 99th-percentile points and both fitted curves

## Limitations

- The 99th-percentile envelope is sensitive to bin occupancy; sparse bins in the tails of the pH or temperature ranges may give unstable percentile estimates
- Fits are calibrated to non-host-associated EMP samples (mostly freshwater and soil); results may not transfer to host-associated environments
- mc2 filtering and metadata availability determine the sample set, so the 27,751-sample metadata coverage noted in the supplement bounds how many samples contribute bins

## Evidence

- [methods] mc2 (minimum sequence count in an OTU to be included equals to 2): "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [supplementary] Also shown is data type (Python/Pandas): "Also shown is data type (Python/Pandas), variable type, and number of the 27,751 samples having metadata for each field."
- [methods] EMP 16S Release 1 BIOM tables: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [other] finding: richness peaks near neutral pH and ~10 °C: [evidence span withheld — anchor status: not_found]
