---
name: environmental-gradient-binning
description: Use when asked to locate the pH or temperature value at which microbial
  richness peaks in non-host-associated EMP samples, or to compare Laplace vs. Gaussian
  fits of richness along an environmental gradient.
when_to_use_negative:
- The question concerns taxonomic composition or abundance (e.g., counting genera
  in samples with pH > 8) rather than locating a richness maximum along a continuous
  environmental gradient.
- Samples are host-associated or lack pH/temperature metadata, leaving too few samples
  to bin into 100/120 equal-width bins.
- The input is clustered-reference OTU data (e.g., closed-reference against Greengenes
  13.8) instead of the exact-sequence Deblur table the gradient analysis was built
  on.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0610
tools:
- name: Python
  role: Scripting environment (with Pandas) used to load the BIOM table and mapping
    file, filter, bin, and fit the gradient curves
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Pandas
  role: Loads and manipulates the mapping file metadata (pH, temperature) and joins
    per-sample richness to environmental covariates
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Deblur
  role: Produces the exact-sequence (unclustered) OTU table whose observed richness
    is binned along the pH and temperature gradients
  repo: https://github.com/cuttlefishh/deblur
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:9c63a332fd87cd9422fbfa5e700cd34d3dfd04ce79867bfc1efcf72bec74c4cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# environmental-gradient-binning

## Summary

Bins environmental metadata gradients (e.g., pH into 100 bins, temperature into 120 bins) from EMP 16S Release 1 Deblur data and fits two-sided exponential (Laplace) and Gaussian curves to per-bin 99th-percentile observed richness to locate the gradient peaks.

## When to use

Use when asked to locate the pH or temperature value at which microbial richness peaks in non-host-associated EMP samples, or to compare Laplace vs. Gaussian fits of richness along an environmental gradient.

## When NOT to use

- The question concerns taxonomic composition or abundance (e.g., counting genera in samples with pH > 8) rather than locating a richness maximum along a continuous environmental gradient.
- Samples are host-associated or lack pH/temperature metadata, leaving too few samples to bin into 100/120 equal-width bins.
- The input is clustered-reference OTU data (e.g., closed-reference against Greengenes 13.8) instead of the exact-sequence Deblur table the gradient analysis was built on.

## Inputs

- QC-filtered EMP 16S Release 1 Deblur BIOM feature table (~25,000 samples, 97 studies)
- Merged and quality-filtered EMP mapping file with pH and temperature metadata
- Habitat-delineated subset of non-host-associated samples (mostly freshwater and soil)

## Outputs

- Per-bin 99th-percentile richness points for pH (100 bins) and temperature (120 bins)
- Laplace fit parameters (a, x0, b) for richness vs. pH and richness vs. temperature, with peak locations near pH 7 and ~10 °C
- Gaussian fit comparison and relative goodness-of-fit statistics (R² / residual sum of squares)
- Figure richness_gradient_laplace_fits.png showing binned points with both fitted curves

## How to apply

Load the QC-filtered EMP 16S Release 1 Deblur BIOM table and its mapping file into Python with Pandas, then apply the mc2 filter (minimum sequence count in an OTU equal to 2) and keep only non-host-associated samples that have pH and temperature metadata. Compute per-sample observed richness, bin samples by pH into 100 equal bins and by temperature into 120 equal bins, and compute the 99th-percentile richness within each bin to make the gradient robust to outlier samples. Fit richness = a*exp(-|x-x0|/b) to each binned gradient with scipy.optimize.curve_fit, and fit a Gaussian on the same binned points for comparison; use relative goodness of fit (R² or residual sum of squares) to confirm the Laplace model outperforms the Gaussian. The fitted peak locations x0 should fall near pH 7 and ~10 °C, reproducing the Fig. 2b result; save binned points with both fitted curves as richness_gradient_laplace_fits.png.

## Related tools

- **Python** (Scripting environment (with Pandas) used to load the BIOM table and mapping file, filter, bin, and fit the gradient curves) — https://github.com/biocore/emp/tree/master/methods
- **Pandas** (Loads and manipulates the mapping file metadata (pH, temperature) and joins per-sample richness to environmental covariates) — https://github.com/biocore/emp/tree/master/methods
- **Deblur** (Produces the exact-sequence (unclustered) OTU table whose observed richness is binned along the pH and temperature gradients) — https://github.com/cuttlefishh/deblur

## Examples

```
p, cov = scipy.optimize.curve_fit(lambda x,a,x0,b: a*np.exp(-np.abs(x-x0)/b), bin_centers, p99_richness, p0=[rmax, 7.0, 1.0])
```

## Evaluation signals

- The Laplace peak x0 lands near pH 7 for the richness-vs-pH fit and near ~10 °C for the richness-vs-temperature fit, matching the Fig. 2b result.
- The Laplace model achieves better relative goodness of fit (higher R² / lower residual sum of squares) than the Gaussian on the same binned data.
- Bin counts match the specification: 100 equal bins for pH and 120 equal bins for temperature.
- Only samples passing the mc2 filter (minimum sequence count in an OTU equal to 2) and having pH and temperature metadata remain after filtering.
- richness_gradient_laplace_fits.png is produced and overlays the binned 99th-percentile richness points with both fitted curves.

## Limitations

- The finding (peak near neutral pH ~7 and ~10 °C) is specific to non-host-associated EMP samples, mostly freshwater and soil, and may not transfer to host-associated habitats.
- The mc2 filter and exact-sequence Deblur processing differ from clustered-OTU alternatives (closed-reference against Greengenes 13.8 or Silva 123), so richness values are processing-dependent.
- Equal-width binning plus 99th-percentile aggregation can be unstable in gradient regions with few samples.

## Evidence

- [methods] mc2 (minimum sequence count in an OTU to be included equals to 2): "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [methods] EMP 16S Release 1 contents: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [supplementary] Deblur exact sequences: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [supplementary] Python/Pandas tooling: "Also shown is data type (Python/Pandas), variable type, and number of the 27,751 samples having metadata for each field."
- [methods] Deblur preferred OTU method: "deblur refers to the exact-sequence de novo OTU picking method Deblur"
