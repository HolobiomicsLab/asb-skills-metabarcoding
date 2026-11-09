---
name: mantel-correlation-testing
description: Use when you have paired OTU tables (e.g. uncurated and LULU-curated
  variants) and an independent survey matrix over the same samples, and you need to
  test correlation of their dissimilarity matrices.
when_to_use_negative:
- The goal is validating alpha-diversity or species-level identity of individual OTUs
  — Mantel correlation only assesses matrix-level turnover/composition, not single
  rare species
- The OTU tables do not share a common sample set with the ground-truth survey data
  (Mantel requires paired, identically ordered samples)
- Only one OTU table is available with no uncurated/curated counterpart or external
  reference matrix to correlate against
tools:
- name: VSEARCH
  role: Produces uncurated OTU tables whose dissimilarity matrices are compared to
    LULU-curated counterparts via Mantel tests
  repo: https://github.com/tobiasgf/lulu
- name: SWARM
  role: Produces uncurated OTU tables whose dissimilarity matrices are compared to
    LULU-curated counterparts via Mantel tests
- name: DADA2
  role: Produces uncurated OTU tables whose dissimilarity matrices are compared to
    LULU-curated counterparts via Mantel tests
- name: CROP
  role: Produces uncurated OTU tables whose dissimilarity matrices are compared to
    LULU-curated counterparts via Mantel tests
- name: LULU
  role: Post-clustering curation that generates the curated OTU tables; Mantel tests
    evaluate whether curation preserves or improves Bray-Curtis correlation with plant
    survey dissimilarity
  repo: https://github.com/tobiasgf/lulu
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:521462de29f0946b9e667f44ba46176cec61d2c43e3572c53855188ef66a4ccd
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
n_claims: 13
---

# mantel-correlation-testing

## Summary

Runs Mantel tests (Pearson correlation with 999 permutations) between Bray-Curtis dissimilarity matrices computed from uncurated vs LULU-curated OTU tables and against vascular plant field survey data, to quantify whether LULU post-clustering curation changes community dissimilarity and whether curation improves OTU-vs-ground-truth beta-diversity correlation.

## When to use

Use when you have paired OTU tables (e.g. uncurated and LULU-curated variants) and an independent survey matrix over the same samples, and you need to test correlation of their dissimilarity matrices.

## When NOT to use

- The goal is validating alpha-diversity or species-level identity of individual OTUs — Mantel correlation only assesses matrix-level turnover/composition, not single rare species
- The OTU tables do not share a common sample set with the ground-truth survey data (Mantel requires paired, identically ordered samples)
- Only one OTU table is available with no uncurated/curated counterpart or external reference matrix to correlate against

## Inputs

- Uncurated and LULU-curated OTU table variants from VSEARCH, SWARM, CROP and DADA2 (samples as columns, OTUs as rows, unique OTU ids as row names)
- Vascular plant field survey data (species presence/richness per site) for the same 130 Danish field sites
- Plant ITS2 metabarcoding soil data context (130 sites in Denmark)

## Outputs

- Bray-Curtis dissimilarity matrices among the 130 sites for each OTU table variant (binary and Hellinger-transformed abundance) and for the plant survey data
- mantel_results.csv with Mantel r statistics and permutation p-values for each uncurated-vs-curated pair and each OTU-vs-plant comparison

## How to apply

Load the OTU table variants (produced by VSEARCH, SWARM, CROP and DADA2, each uncurated and LULU-curated) and the paired vascular plant field survey data for the 130 Danish sites in R. Convert each OTU table to both a binary presence/absence matrix and a Hellinger-transformed abundance matrix, then compute Bray-Curtis dissimilarity matrices among the 130 sites for every variant and for the plant survey data. Run Mantel tests with Pearson correlation and 999 permutations between each uncurated-vs-curated pair of dissimilarity matrices and between each OTU-table matrix and the plant survey matrix. Rationale: the article's validation compares OTU-based beta diversity against thorough botanical survey data (564 species, per-site richness 6-93) as ground truth, so matrix-level correlation is the appropriate metric for turnover/composition, while alpha-diversity is assessed separately. Assemble all Mantel r statistics and permutation p-values into mantel_results.csv.

## Related tools

- **VSEARCH** (Produces uncurated OTU tables whose dissimilarity matrices are compared to LULU-curated counterparts via Mantel tests) — https://github.com/tobiasgf/lulu
- **SWARM** (Produces uncurated OTU tables whose dissimilarity matrices are compared to LULU-curated counterparts via Mantel tests)
- **DADA2** (Produces uncurated OTU tables whose dissimilarity matrices are compared to LULU-curated counterparts via Mantel tests)
- **CROP** (Produces uncurated OTU tables whose dissimilarity matrices are compared to LULU-curated counterparts via Mantel tests)
- **LULU** (Post-clustering curation that generates the curated OTU tables; Mantel tests evaluate whether curation preserves or improves Bray-Curtis correlation with plant survey dissimilarity) — https://github.com/tobiasgf/lulu

## Examples

```
mantel(otuD, plantD, method="pearson", permutations=999)  # Bray-Curtis dissimilarity matrices from uncurated/curated OTU tables vs plant survey, in R
```

## Evaluation signals

- Uncurated-vs-curated binary Mantel r values fall within the reported 0.761-0.993 range, all with p < 0.001
- OTU-vs-plant survey Mantel r values fall within the reported 0.57-0.78 range
- For all 20 OTU tables, curation yields dissimilarity matrices with the same or slightly improved correlation with plant data (never worse)
- All dissimilarity matrices are computed over the same 130 sites, so the Mantel test input matrices have identical dimensions and ordering
- mantel_results.csv contains both an r statistic and a permutation p-value for every uncurated-vs-curated pair and every OTU-vs-plant comparison

## Limitations

- Mantel correlation with plant data validates beta-diversity/composition only; it does not show that individual rare species are correctly resolved — the authors note LULU curation may not be suitable for identifying single rare species
- The 'cost of curation' is a small loss of 'real species' (rare, low-abundance taxa), which matrix correlation will not detect
- CROP OTU picking appeared suboptimal compared with the other clustering methods, so its Mantel results should be interpreted with that caveat
- Validation was performed only for vascular plants; relevance to other organism groups is inferred, not directly tested

## Evidence

- [results] Mantel tests showed all un-curated vs curated dissimilarity matrices were highly correlated (mantel r 0.761-0.993 for binary data, all p < 0.001): [evidence span withheld — anchor status: not_found]
- [results] for all 20 OTU tables curation resulted in dissimilarity matrices with the same or slightly improved correlation with plant data: [evidence span withheld — anchor status: not_found]
- [abstract] we use a unique data set of high quality survey data of vascular plants paired with plant ITS2 metabarcoding data of DNA extracted from soil from 130 sites in Denmark: "we use a unique data set of high quality survey data of vascular plants paired with plant ITS2 metabarcoding data of DNA extracted from soil from 130 sites in Denmark"
- [intro] OTU tables were produced from the sequence data using a set of representative clustering methods: VSEARCH22, SWARM23, CROP24 and DADA213: "OTU tables were produced from the sequence data using a set of representative clustering methods: VSEARCH22, SWARM23, CROP24 and DADA213"
- [methods] The OTU table contains 2425 OTUs destributed over 130 sites/samples.: "The OTU table contains 2425 OTUs destributed over 130 sites/samples."
- [intro] current approaches are likely to retain only dominant species, which may be adequate for assessing composition and turnover, but much less so for α-diversity: "current approaches are likely to retain only dominant species, which may be adequate for assessing composition and turnover, but much less so for α-diversity"
