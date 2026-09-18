---
name: hellinger-abundance-transformation
description: Use when an OTU table (e.g. 2425 OTUs over 130 sites from VSEARCH, SWARM,
  CROP or DADA2, uncurated or LULU-curated) must be transformed into Hellinger-standardized
  abundance or binary presence/absence form before computing Bray-Curtis dissimilarity
  for β-diversity or Mantel correlation analyses.
when_to_use_negative:
- Input is already a transformed or relative-abundance table — re-transforming would
  distort the Bray-Curtis dissimilarities
- 'The goal is α-diversity estimation rather than composition/turnover: the paper
  notes current approaches retaining composition metrics are ''much less so for α-diversity'',
  and Hellinger/Bray-Curtis is a β-diversity tool'
- The aim is to identify individual rare species, since LULU-style curation and dissimilarity-level
  analyses are not suitable for that purpose
tools:
- name: LULU
  role: Produces the curated OTU tables (samples as columns, OTUs as rows) that are
    subsequently Hellinger-transformed and converted to dissimilarity matrices for
    the Mantel comparison of uncurated vs curated data
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the clustering methods generating the uncurated OTU tables that are
    transformed and compared (also usable to build the match list prior to LULU curation)
- name: DADA2
  role: Clustering method generating uncurated OTU tables for the transformation and
    Mantel-test comparison
- name: SWARM
  role: Clustering method generating uncurated OTU tables for the transformation and
    Mantel-test comparison
- name: CROP
  role: Clustering method generating uncurated OTU tables for the transformation and
    Mantel-test comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:b227ed8879c55cb2a07cc95ed7e63e3f5e4889e3fed0e643c6af6948f5330554
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - CROP
  - DADA2
  - LULU
  - SWARM
  - VSEARCH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# hellinger-abundance-transformation

## Summary

Converts OTU abundance tables (samples as columns, OTUs as rows, per the LULU format) into Hellinger-transformed relative-abundance matrices — and paired binary presence/absence matrices — so that Bray-Curtis dissimilarity among the 130 Danish field sites can be computed and compared against plant survey ground truth via Mantel tests.

## When to use

Use when an OTU table (e.g. 2425 OTUs over 130 sites from VSEARCH, SWARM, CROP or DADA2, uncurated or LULU-curated) must be transformed into Hellinger-standardized abundance or binary presence/absence form before computing Bray-Curtis dissimilarity for β-diversity or Mantel correlation analyses.

## When NOT to use

- Input is already a transformed or relative-abundance table — re-transforming would distort the Bray-Curtis dissimilarities
- The goal is α-diversity estimation rather than composition/turnover: the paper notes current approaches retaining composition metrics are 'much less so for α-diversity', and Hellinger/Bray-Curtis is a β-diversity tool
- The aim is to identify individual rare species, since LULU-style curation and dissimilarity-level analyses are not suitable for that purpose

## Inputs

- OTU abundance table in LULU format (samples as columns, OTUs as rows with unique OTU id row names), e.g. otutable_test.txt with 2425 OTUs over 130 samples
- Paired vascular plant field survey site-by-species table for the 130 Danish sites (ground truth)

## Outputs

- Hellinger-transformed abundance matrix (130 sites x OTUs)
- Binary presence/absence matrix (130 sites x OTUs)
- Bray-Curtis dissimilarity matrices among the 130 sites for each OTU table variant and for the plant survey data
- Mantel test r statistics and permutation p-values assembled into mantel_results.csv

## How to apply

Load the OTU table with samples as columns and OTUs as unique row ids (the format LULU requires). For the abundance variant, Hellinger-transform the data so each sample's counts become square-rooted relative abundances (transpose first, since the LULU table has samples as columns, then scale each sample vector to sum to 1 before taking square roots); for the binary variant, convert all positive counts to 1 and zeros to 0. Compute Bray-Curtis dissimilarity among the 130 sites on both the Hellinger-transformed and binary matrices, plus a dissimilarity matrix for the paired vascular plant field survey data (564 species, per-site richness 6–93). Run Mantel tests with Pearson correlation and 999 permutations between uncurated-vs-curated pairs and between each OTU-derived matrix and the plant survey matrix. The rationale is that Hellinger transformation down-weights dominant taxa so Bray-Curtis reflects composition rather than raw read depth, mirroring the paper's evaluation of LULU curation against botanical ground truth.

## Related tools

- **LULU** (Produces the curated OTU tables (samples as columns, OTUs as rows) that are subsequently Hellinger-transformed and converted to dissimilarity matrices for the Mantel comparison of uncurated vs curated data) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the clustering methods generating the uncurated OTU tables that are transformed and compared (also usable to build the match list prior to LULU curation))
- **DADA2** (Clustering method generating uncurated OTU tables for the transformation and Mantel-test comparison)
- **SWARM** (Clustering method generating uncurated OTU tables for the transformation and Mantel-test comparison)
- **CROP** (Clustering method generating uncurated OTU tables for the transformation and Mantel-test comparison)

## Examples

```
library(vegan); otu_hellinger <- decostand(t(otutable), method = "hellinger"); otu_bray <- vegdist(otu_hellinger, method = "bray")
```

## Evaluation signals

- Uncurated-vs-curated binary Mantel r values fall within the reported 0.761–0.993 range with all p < 0.001 (999 permutations)
- OTU-vs-plant Mantel r values fall within the reported 0.57–0.78 range
- Curation yields dissimilarity matrices with the same or slightly improved correlation with plant data for all 20 OTU tables
- Hellinger-transformed sample vectors each sum to the same constant (relative abundances), and the dissimilarity matrix has dimensions 130 x 130 matching the site count
- The binary matrix contains only 0/1 values and preserves the OTU id row names of the input LULU-format table

## Limitations

- The validation dataset covers only vascular plant ITS2 soil metabarcoding from 130 Danish sites; relevance to other organism groups is inferred but not directly tested
- Curation before transformation entails a small loss of real (rare, low-abundance) species, which propagates into the transformed matrices
- Dissimilarity-based analyses are adequate for assessing composition and turnover but much less so for α-diversity

## Evidence

- [results] Mantel tests showed all un-curated vs curated dissimilarity matrices were highly correlated (mantel r 0.761-0.993 for binary data, all p < 0.001): [evidence span withheld — anchor status: not_found]
- [methods] Convert each OTU table to a binary presence/absence matrix and a Hellinger-transformed abundance matrix: [evidence span withheld — anchor status: not_found]
- [intro] OTU tables were produced from the sequence data using a set of representative clustering methods: VSEARCH22, SWARM23, CROP24 and DADA213: "OTU tables were produced from the sequence data using a set of representative clustering methods: VSEARCH22, SWARM23, CROP24 and DADA213"
- [readme] The requirements of **LULU** are that the OTU table has samples as columns and OTUs as rows, and that it has unique OTU id's as row names.: "The requirements of **LULU** are that the OTU table has samples as columns and OTUs as rows, and that it has unique OTU id's as row names."
- [intro] current approaches are likely to retain only dominant species, which may be adequate for assessing composition and turnover, but much less so for α-diversity: "current approaches are likely to retain only dominant species, which may be adequate for assessing composition and turnover, but much less so for α-diversity"
