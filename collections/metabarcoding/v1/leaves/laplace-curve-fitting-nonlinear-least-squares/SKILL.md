---
name: laplace-curve-fitting-nonlinear-least-squares
description: Use when per-sample alpha-diversity (e.g. observed richness) must be
  summarized against an environmental gradient (pH, temperature) to locate the gradient
  value at which the 99th-percentile richness peaks, using a peaked, two-sided exponential
  rather than a Gaussian.
when_to_use_negative:
- The metadata lacks the environmental variable of interest (e.g. no pH or temperature
  values), since binning and peak location cannot be computed
- You need mean/typical richness along the gradient rather than the upper bound —
  the 99th-percentile binning targets the maximum-observed richness envelope
- The feature table has not passed the mc2 minimum-sequence-count filter (count ≥
  2), which the article's workflow applies before computing richness
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3318
tools:
- name: Python
  role: Analysis language for loading the BIOM table and metadata, binning samples,
    and running the nonlinear least-squares fits
- name: Pandas
  role: Loads the mapping file (pH/temperature metadata) and joins it to per-sample
    richness computed from the Deblur BIOM table
- name: Deblur
  role: Produced the exact-sequence (V4, 90-bp tag) 16S feature table whose per-sample
    observed richness is the response variable
  repo: https://github.com/cuttlefishh/deblur
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:4f4657298f46de7d983a00c91afca164c60c33d32ad1203d87d800f74b58ba2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Pandas
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Laplace curve fitting via nonlinear least squares

## Summary

Fits a two-sided exponential (Laplace) model, richness = a*exp(-|x-x0|/b), to binned richness-vs-gradient data (pH, temperature) from the EMP 16S Release 1 dataset using scipy curve_fit, and benchmarks it against a Gaussian fit to justify the Laplace shape. This recovers the pH/temperature locations of peak microbial richness.

## When to use

Use when per-sample alpha-diversity (e.g. observed richness) must be summarized against an environmental gradient (pH, temperature) to locate the gradient value at which the 99th-percentile richness peaks, using a peaked, two-sided exponential rather than a Gaussian.

## When NOT to use

- The metadata lacks the environmental variable of interest (e.g. no pH or temperature values), since binning and peak location cannot be computed
- You need mean/typical richness along the gradient rather than the upper bound — the 99th-percentile binning targets the maximum-observed richness envelope
- The feature table has not passed the mc2 minimum-sequence-count filter (count ≥ 2), which the article's workflow applies before computing richness

## Inputs

- QC-filtered EMP 16S Release 1 Deblur BIOM table (exact-sequence feature table)
- EMP mapping file (merged, quality-filtered metadata) with pH and temperature fields
- Choice of environmental gradient to bin against (pH or temperature)

## Outputs

- Binned 99th-percentile richness values per pH bin (100 bins) and per temperature bin (120 bins)
- Fitted Laplace parameters (amplitude a, peak location x0, decay scale b) per gradient
- Comparison of Laplace vs Gaussian fit quality (R²/residual sum of squares)
- richness_gradient_laplace_fits.png showing binned points with both fitted curves

## How to apply

Load the QC-filtered EMP 16S Release 1 Deblur BIOM table and its mapping file with Pandas; restrict to samples carrying pH and temperature metadata and apply the mc2 filter (minimum sequence count in an OTU equals 2). Bin samples into 100 equal pH bins and 120 equal temperature bins and compute the 99th-percentile observed richness per bin, which emphasizes the richness ceiling rather than the mean. Fit richness = a*exp(-|x-x0|/b) to the binned points with scipy.optimize.curve_fit for each gradient, and fit a Gaussian to the same points for comparison; compare residuals/relative goodness of fit to confirm the Laplace model outperforms the Gaussian. Verify the fitted peaks fall near pH 7 and about 10 °C, consistent with the Fig. 2b result, and save the binned points with both curves as richness_gradient_laplace_fits.png.

## Related tools

- **Python** (Analysis language for loading the BIOM table and metadata, binning samples, and running the nonlinear least-squares fits)
- **Pandas** (Loads the mapping file (pH/temperature metadata) and joins it to per-sample richness computed from the Deblur BIOM table)
- **Deblur** (Produced the exact-sequence (V4, 90-bp tag) 16S feature table whose per-sample observed richness is the response variable) — https://github.com/cuttlefishh/deblur

## Examples

```
from scipy.optimize import curve_fit; laplace = lambda x, a, x0, b: a*np.exp(-np.abs(x-x0)/b); popt, pcov = curve_fit(laplace, ph_bin_centers, p99_richness, p0=[max(p99_richness), 7.0, 1.0])
```

## Evaluation signals

- Laplace peak location lands near pH 7 and ~10 °C for the non-host-associated EMP samples, matching the Fig. 2b result
- The Laplace fit has lower residuals / better relative goodness of fit (R²) than the Gaussian fit on the same binned 99th-percentile points
- Binning used the expected resolution: 100 equal pH bins and 120 equal temperature bins
- Samples were filtered per mc2 (minimum sequence count in an OTU equals 2) and only samples with pH and temperature metadata retained
- Output figure richness_gradient_laplace_fits.png contains the binned points overlaid with both fitted curves

## Limitations

- The mc2 filter (minimum OTU sequence count of 2) drops rare features, so computed richness undercounts singletons
- The analysis was restricted to non-host-associated EMP samples (mostly freshwater and soil); peak locations are not guaranteed to transfer to host-associated habitats
- Binned 99th-percentile values are sensitive to per-bin sample coverage; bins with few samples give unstable upper-bound estimates

## Evidence

- [methods] mc2 filter definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [methods] EMP Release 1 inputs: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [readme] repo purpose: "This GitHub repository describes the EMP catalogue -- how it is generated and how to use it."
- [supplementary] exact-sequence response variable: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
