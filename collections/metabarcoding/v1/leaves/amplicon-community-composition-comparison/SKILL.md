---
name: amplicon-community-composition-comparison
description: Use when you have paired amplicon OTU tables and independent community
  survey data and need to quantify whether a processing choice (e.g., LULU curation)
  alters or improves beta-diversity/composition correlations.
when_to_use_negative:
- The goal is assessing alpha-diversity (species richness) rather than community dissimilarity/turnover
  — LULU's authors note composition/turnover assessment is adequate but alpha-diversity
  needs different evaluation
- Amplicon data and survey data are not paired site-by-site, so a Mantel test of matrix
  correlation is not meaningful
- You need to identify single rare species; the paper cautions LULU curation may not
  be suitable for identifying single rare species in community data sets
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3305
tools:
- name: LULU
  role: Produces the curated OTU tables whose dissimilarity matrices are compared
    against the uncurated ones via Mantel tests
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the clustering methods used to produce OTU tables prior to LULU curation;
    also an option for generating the match list
- name: SWARM
  role: One of the representative clustering methods used to produce OTU tables prior
    to LULU curation
- name: DADA2
  role: One of the representative clustering methods used to produce OTU tables prior
    to LULU curation
- name: CROP
  role: One of the representative clustering methods used to produce OTU tables prior
    to LULU curation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:f1681aeaaf8512a570d6cd2d85e7bba22212c94a5f092cd5d05da98133f70c1f
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# amplicon-community-composition-comparison

## Summary

Compares community composition patterns between OTU tables (e.g., uncurated vs LULU-curated) and ground-truth survey data by computing Bray-Curtis dissimilarity matrices and assessing their agreement with Mantel tests (Pearson correlation, 999 permutations).

## When to use

Use when you have paired amplicon OTU tables and independent community survey data and need to quantify whether a processing choice (e.g., LULU curation) alters or improves beta-diversity/composition correlations.

## When NOT to use

- The goal is assessing alpha-diversity (species richness) rather than community dissimilarity/turnover — LULU's authors note composition/turnover assessment is adequate but alpha-diversity needs different evaluation
- Amplicon data and survey data are not paired site-by-site, so a Mantel test of matrix correlation is not meaningful
- You need to identify single rare species; the paper cautions LULU curation may not be suitable for identifying single rare species in community data sets

## Inputs

- OTU tables (samples as columns, OTUs as rows with unique OTU ids), uncurated and LULU-curated, from VSEARCH/SWARM/CROP/DADA2
- Paired vascular plant field survey data (community composition per site)
- Site/sample metadata aligning the 130 field sites across tables and survey

## Outputs

- Bray-Curtis dissimilarity matrices among sites for each OTU table variant and the plant survey data
- mantel_results.csv with Mantel r statistics and permutation p-values for each comparison
- Validation verdict comparing observed Mantel r to reported ranges (binary 0.761–0.993; OTU-vs-plant 0.57–0.78)

## How to apply

Load the OTU table variants (uncurated and LULU-curated, produced by VSEARCH, SWARM, CROP or DADA2) plus the paired field survey data in R. Convert each table to both a binary presence/absence matrix and a Hellinger-transformed abundance matrix, then compute Bray-Curtis dissimilarity matrices among the 130 sites for every variant and for the plant survey data. Run Mantel tests with Pearson correlation and 999 permutations between each uncurated-vs-curated pair of dissimilarity matrices and between each OTU-table matrix and the plant survey matrix. Accept the validation only if uncurated-vs-curated binary Mantel r falls within the reported 0.761–0.993 range (all p < 0.001) and OTU-vs-plant r falls within the reported 0.57–0.78 range; curation should yield the same or slightly improved correlation with plant data for all 20 OTU tables.

## Related tools

- **LULU** (Produces the curated OTU tables whose dissimilarity matrices are compared against the uncurated ones via Mantel tests) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the clustering methods used to produce OTU tables prior to LULU curation; also an option for generating the match list)
- **SWARM** (One of the representative clustering methods used to produce OTU tables prior to LULU curation)
- **DADA2** (One of the representative clustering methods used to produce OTU tables prior to LULU curation)
- **CROP** (One of the representative clustering methods used to produce OTU tables prior to LULU curation)

## Examples

```
R: mantel_r <- vegan::mantel(dist_uncurated, dist_curated, method="pearson", permutations=999)
```

## Evaluation signals

- Uncurated-vs-curated binary-data Mantel r values fall within the reported 0.761–0.993 range with all p < 0.001
- OTU-table-vs-plant Mantel r values fall within the reported 0.57–0.78 range
- For all 20 OTU tables, curation yields dissimilarity matrices with the same or slightly improved correlation with plant survey data
- All Mantel tests use Pearson correlation with 999 permutations, and results are assembled into mantel_results.csv

## Limitations

- Validation was performed only for vascular plants; relevance to other organism groups is inferred but not directly tested
- CROP OTU picking appeared suboptimal (low similarity to best GenBank match, poor redundancy and beta-diversity measures)
- Amplicon beta-diversity was adequate for assessing composition and turnover but this does not guarantee accurate alpha-diversity

## Evidence

- [full_text] Mantel tests showed all un-curated vs curated dissimilarity matrices were highly correlated (mantel r 0.761-0.993 for binary data, all p < 0.001): [evidence span withheld — anchor status: not_found]
- [intro] current approaches are likely to retain only dominant species, which may be adequate for assessing composition and turnover, but much less so for α-diversity: "current approaches are likely to retain only dominant species, which may be adequate for assessing composition and turnover, but much less so for α-diversity"
- [discussion] the OTU picking of CROP seemed to be suboptimal compared to the other methods investigated: "the OTU picking of CROP seemed to be suboptimal compared to the other methods investigated"
- [readme] The requirements of LULU are that the OTU table has samples as columns and OTUs as rows, and that it has unique OTU id's as row names.: "The requirements of **LULU** are that the OTU table has samples as columns and OTUs as rows, and that it has unique OTU id's as row names."
