---
name: isolation-by-distance-interpretation
description: Use when sample sites span multiple geographic locations and you need
  to test whether community turnover (e.g. Bray-Curtis beta-diversity of prokaryotic
  OTUs) correlates with inter-site distance via a Mantel test.
when_to_use_negative:
- When beta-diversity is computed for eukaryotic (ITS) communities — the article's
  Mantel test was reported for Bray-Curtis beta-diversity of prokaryotes only
- When you need a mechanistic dispersal model rather than a simple correlation — a
  significant Mantel r does not establish causation, and here correlations between
  intestinal bacteria, feed type, and volatile compounds could not currently be stated
- When exact geographic coordinates are unavailable — the study notes exact farm locations
  cannot be disclosed at farmers' demand, limiting reproducibility of the distance
  matrix
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: IBD
  role: Runs the isolation-by-distance Mantel test between geographic distance and
    beta-diversity matrices
- name: XLSTAT
  role: XLSTAT 2017 (Addinsoft, Paris, France) used to perform the Mantel test correlation
    analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:3c37015041ccbcdd6b3dafaf24875227fab55c207cbcb7977cd70477742eedf2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IBD
  - XLSTAT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# isolation-by-distance-interpretation

## Summary

Tests whether microbial beta-diversity correlates with inter-farm geographic distance using a Mantel test computed with IBD v1.52 and XLSTAT 2017, as applied to Bray-Curtis dissimilarities of prokaryotic (16S) communities from trout farm inflow- and pond-water samples.

## When to use

Use when sample sites span multiple geographic locations and you need to test whether community turnover (e.g. Bray-Curtis beta-diversity of prokaryotic OTUs) correlates with inter-site distance via a Mantel test.

## When NOT to use

- When beta-diversity is computed for eukaryotic (ITS) communities — the article's Mantel test was reported for Bray-Curtis beta-diversity of prokaryotes only
- When you need a mechanistic dispersal model rather than a simple correlation — a significant Mantel r does not establish causation, and here correlations between intestinal bacteria, feed type, and volatile compounds could not currently be stated
- When exact geographic coordinates are unavailable — the study notes exact farm locations cannot be disclosed at farmers' demand, limiting reproducibility of the distance matrix

## Inputs

- Bray-Curtis beta-diversity matrix computed from prokaryotic (16S V3-V4) OTU abundance data
- Inter-farm geographic distance matrix between the three semi-intensive trout farms
- Per-water-source sample grouping (inflow/I-source vs pond-water)

## Outputs

- Mantel test r statistic and p-value per water source (e.g. pond_water_r = 0.969, pond_water_p < 0.0001)
- Significance flag per water source (inflow-water: not significant, p > 0.05)
- Structured JSON record encoding both outcomes, e.g. mantel_test_results

## How to apply

First compute a beta-diversity matrix — in this study, Bray-Curtis dissimilarity of prokaryotic (16S) communities only — and pair it with an inter-farm geographic distance matrix. Run the Mantel test with IBD v1.52 and XLSTAT 2017 (Addinsoft, Paris, France), testing each water source separately: inflow-water (I-source) samples versus pond-water samples. Interpret the two outcomes independently: here pond-water characteristic species count correlated significantly with inter-farm distance (r = 0.969, p < 0.0001) while no significant correlation was observed for the I-source samples (p > 0.05). A significant pond-water result supports the conclusion that pond-water biodiversity and microbial enrichment are more related to location than to water source; a non-significant inflow result indicates source-water communities are not structured by distance. Record r, exact p-threshold, water source, and significance flag per comparison, and note the limitation that exact farm locations were withheld at farmers' request, which limits geographic reproducibility.

## Related tools

- **IBD** (Runs the isolation-by-distance Mantel test between geographic distance and beta-diversity matrices)
- **XLSTAT** (XLSTAT 2017 (Addinsoft, Paris, France) used to perform the Mantel test correlation analysis)

## Evaluation signals

- Encoded pond-water statistics match the reported values exactly: r = 0.969 and p < 0.0001 (significant)
- Inflow-water (I-source) outcome recorded as not significant with p > 0.05
- Both water-source comparisons tested separately with distinct significance flags in the JSON record
- Beta-diversity input confirmed to be Bray-Curtis dissimilarity from prokaryotic (16S) data only
- Interpretation consistent with the article's finding that pond-water biodiversity is more related to location than the water source

## Limitations

- Exact farm locations cannot be disclosed based on farmer demand, limiting geographic reproducibility of the distance matrix
- The Mantel test showed no significant correlation for inflow-water samples, so isolation-by-distance patterns cannot be generalized across water sources
- Human- and industrial-related activities around the sampling area affected the microbiota, confounding pure distance-based interpretation

## Evidence

- [discussion] Mantel test run with IBD and XLSTAT on prokaryotic Bray-Curtis beta-diversity: "tested using a Mantel test and performed using IBD v1.5259 and XLSTAT 2017 (Addinsoft, Paris, France) software"
- [discussion] No significant Mantel correlation for inflow (I-source) samples: "Based on the Mantel test, no significant correlation was observed for the I-source samples"
- [discussion] Pond-water biodiversity related to location rather than water source: "the biodiversity and microbial enrichment of eukaryotes and prokaryotes in pond-water are more related to location than the water source"
- [discussion] Exact farm locations withheld, limiting reproducibility: "The exact location of these farms cannot be disclosed based on the farmer demand."
- [discussion] Beta-diversity computed via Bray-Curtis for prokaryotes only: "was calculated according to the Simpson diversity index56"
