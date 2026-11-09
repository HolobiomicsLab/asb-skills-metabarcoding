---
name: bray-curtis-dissimilarity-computation
description: Use when you have a site-by-OTU abundance table (e.g., 130 Danish sites
  x 2425 OTUs) or paired ground-truth survey data and need to quantify beta-diversity
  and test whether OTU curation changes dissimilarity structure.
when_to_use_negative:
- The input is already a dissimilarity/distance matrix, not a raw OTU table with counts
- The goal is single rare-species identification — the article notes LULU curation
  may not be suitable for identifying single rare species in community data sets,
  so curated-table beta-diversity should not be over-interpreted at species level
- Sample metadata or site pairing is unavailable, so a valid uncurated-vs-curated
  or OTU-vs-plant Mantel comparison cannot be constructed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3307
tools:
- name: LULU
  role: Post-clustering curation of OTU tables whose curated vs uncurated Bray-Curtis
    dissimilarity matrices are compared; produces the curated tables via lulu(otutable,
    matchlist) in R
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the initial clustering methods producing the OTU tables that are dissimilarity-analysed;
    also usable to generate the internal-OTU match list
- name: SWARM
  role: Initial clustering method producing OTU tables for the dissimilarity/Mantel
    comparison
- name: DADA2
  role: Initial OTU-definition method producing OTU tables for the dissimilarity/Mantel
    comparison
- name: CROP
  role: Initial clustering method producing OTU tables for the dissimilarity/Mantel
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:229c7e8870b7b1daeabe1ae6fc9e5672dd2bedc80ea53080b05e6b05a0926d04
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bray-curtis-dissimilarity-computation

## Summary

Compute Bray-Curtis dissimilarity matrices among sites from OTU tables (binary presence/absence and Hellinger-transformed abundance variants) and from paired field survey data, then compare matrices with Mantel tests (Pearson correlation, 999 permutations) to quantify how OTU curation affects community dissimilarity.

## When to use

Use when you have a site-by-OTU abundance table (e.g., 130 Danish sites x 2425 OTUs) or paired ground-truth survey data and need to quantify beta-diversity and test whether OTU curation changes dissimilarity structure.

## When NOT to use

- The input is already a dissimilarity/distance matrix, not a raw OTU table with counts
- The goal is single rare-species identification — the article notes LULU curation may not be suitable for identifying single rare species in community data sets, so curated-table beta-diversity should not be over-interpreted at species level
- Sample metadata or site pairing is unavailable, so a valid uncurated-vs-curated or OTU-vs-plant Mantel comparison cannot be constructed

## Inputs

- OTU table (samples as columns, OTUs as rows, unique OTU ids as row names), e.g. otutable_test.txt with 2425 OTUs over 130 sites/samples
- LULU-curated OTU tables produced by the lulu() function from uncurated tables plus a match list
- Paired vascular plant field survey presence/absence or abundance data for the 130 Danish sites

## Outputs

- Bray-Curtis dissimilarity matrices (130 x 130) for each OTU table variant (binary and Hellinger-transformed abundance)
- Bray-Curtis dissimilarity matrix for the plant survey data
- mantel_results.csv with Mantel r statistics and permutation p-values for uncurated-vs-curated pairs and OTU-vs-plant comparisons

## How to apply

Load the OTU table variants (uncurated and LULU-curated tables produced by VSEARCH, SWARM, CROP and DADA2) with samples as columns and unique OTU ids as rows. Convert each table to a binary presence/absence matrix and a Hellinger-transformed abundance matrix. Compute Bray-Curtis dissimilarity matrices among the 130 sites for every OTU-table variant and for the plant field survey data (564 species, per-site richness 6-93). Run Mantel tests with Pearson correlation and 999 permutations between each uncurated-vs-curated pair of dissimilarity matrices and between each OTU-table matrix and the plant survey matrix. The rationale is that curation should preserve or improve agreement between molecular dissimilarity and survey-based ground truth while merging erroneous daughter OTUs into parents.

## Related tools

- **LULU** (Post-clustering curation of OTU tables whose curated vs uncurated Bray-Curtis dissimilarity matrices are compared; produces the curated tables via lulu(otutable, matchlist) in R) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the initial clustering methods producing the OTU tables that are dissimilarity-analysed; also usable to generate the internal-OTU match list)
- **SWARM** (Initial clustering method producing OTU tables for the dissimilarity/Mantel comparison)
- **DADA2** (Initial OTU-definition method producing OTU tables for the dissimilarity/Mantel comparison)
- **CROP** (Initial clustering method producing OTU tables for the dissimilarity/Mantel comparison)

## Examples

```
mantel(D_otu_curated, D_plant, method="pearson", permutations=999)  # Bray-Curtis matrices from binary and Hellinger-transformed tables
```

## Evaluation signals

- Uncurated-vs-curated binary Mantel r values fall within the reported 0.761-0.993 range with all p < 0.001
- OTU-vs-plant Mantel r values fall within the reported 0.57-0.78 range
- For all 20 OTU tables, curation yields dissimilarity matrices with the same or slightly improved correlation with plant data
- Dissimilarity matrices are 130 x 130 with zero diagonal and symmetric off-diagonal entries
- Hellinger-transformed abundance matrices preserve total read counts after LULU parent-merging, so site totals remain comparable

## Limitations

- Mantel r for uncurated-vs-curated matrices is high but never exactly 1, so curation does measurably change beta-diversity structure even when the overall correlation is strong
- CROP OTU picking appeared suboptimal compared to the other methods investigated (low similarity to best GenBank match, poor redundancy and beta-diversity measures)
- The 'cost of curation' is a small loss of 'real species' (rare, low-abundance OTUs), which can affect dissimilarity estimates

## Evidence

- [results] Mantel tests showed all un-curated vs curated dissimilarity matrices were highly correlated (mantel r 0.761-0.993 for binary data, all p < 0.001): [evidence span withheld — anchor status: not_found]
- [intro] OTU tables produced by representative clustering methods: "OTU tables were produced from the sequence data using a set of representative clustering methods: VSEARCH22, SWARM23, CROP24 and DADA213"
- [methods] Run Mantel tests with Pearson correlation and 999 permutations between each uncurated-vs-curated pair: [evidence span withheld — anchor status: not_found]
- [readme] README on LULU purpose and merging of erroneous OTUs: "By evaluating the co-occurence patterns of OTUs among samples **LULU** identifies OTUs that consistently satisfy some user selected criteria for being errors of more abundant OTUs and merges these."
