---
name: post-clustering-curation-comparison
description: Decision support for choosing between discarding rare OTUs (singletons)
  and co-occurrence-based post-clustering curation when validating amplicon alpha-diversity
  estimates against independent ground-truth richness data.
when_to_use_negative:
- When the goal is assessing composition or turnover rather than alpha-diversity —
  the article notes that discarding the abundance tail may be adequate for composition
  but 'much less so for alpha-diversity'
- When the aim is identifying single rare species in a community dataset — LULU curation
  may not be suitable and carries a small 'cost of curation' in real rare species
- When no paired independent ground-truth richness data (e.g. field survey) exist
  to compute correspondence metrics such as R2
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3305
tools:
- name: LULU
  role: R package performing the co-occurrence-based post-clustering curation whose
    improvement is benchmarked against singleton removal
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the initial clustering methods producing OTU tables, and also used
    to build the self match list for LULU (--usearch_global --self --id .84)
- name: SWARM
  role: Greedy clustering method producing one of the OTU tables compared; improvement
    by LULU was most pronounced for VSEARCH and SWARM
  repo: https://github.com/frederic-mahe/swarm
- name: CROP
  role: Clustering method producing one of the OTU tables compared (its OTU picking
    appeared suboptimal in the study)
- name: DADA2
  role: Initial OTU-definition method producing one of the OTU tables compared; part
    of the recommended workflow with VSEARCH clustering and LULU curation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:82b576ecefe1a4afc2e41a7d23e16b9cb8343a4b69f1319f000bffecb8331d41
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
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# post-clustering-curation-comparison

## Summary

A benchmarking skill that quantifies how singleton removal compares to LULU post-clustering curation for improving the correspondence between molecular OTU richness and field-survey plant richness. In the source study, singleton removal gave a mean R2 gain of only 0.03 versus 0.27 for LULU across VSEARCH, SWARM, CROP and DADA2 OTU tables from 130 Danish sites.

## When to use

Decision support for choosing between discarding rare OTUs (singletons) and co-occurrence-based post-clustering curation when validating amplicon alpha-diversity estimates against independent ground-truth richness data.

## When NOT to use

- When the goal is assessing composition or turnover rather than alpha-diversity — the article notes that discarding the abundance tail may be adequate for composition but 'much less so for alpha-diversity'
- When the aim is identifying single rare species in a community dataset — LULU curation may not be suitable and carries a small 'cost of curation' in real rare species
- When no paired independent ground-truth richness data (e.g. field survey) exist to compute correspondence metrics such as R2

## Inputs

- Initial OTU tables (samples as columns, OTUs as rows, unique OTU ids as row names) produced by VSEARCH, SWARM, CROP and/or DADA2
- Representative OTU centroid/sequence FASTA files matching the OTU table ids
- Match list of internal OTU matches (query id, target id, percent similarity) from BLASTn or VSEARCH
- Paired ground-truth field-survey richness data per site (e.g. vascular plant species counts for the same 130 sampling sites)

## Outputs

- Singleton-filtered OTU tables (OTUs with total read count of one removed)
- LULU-curated OTU tables with statistics (curated_result$curated_table)
- Per-method R2 values for OTU richness vs plant richness under uncurated, singleton-removed, and LULU-curated conditions
- Comparison table (e.g. comparison_table.csv) of mean R2 improvements: singleton removal (~0.03) vs LULU curation (~0.27)

## How to apply

Load the initial OTU tables produced with representative clustering methods (VSEARCH, SWARM, CROP and DADA2) plus paired field-survey richness data (in the source study: plant ITS2 soil metabarcoding from 130 Danish sites with 564 field-recorded species, per-site richness 6–93). First filter out singleton OTUs (total read count of one) from each table, compute per-site OTU richness, and regress it against per-site plant species richness to obtain baseline-corrected R2 values, alongside the uncurated-table baseline. Then curate the same tables with LULU in R using minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, and minimum_relative_cooccurence=0.95, preceded by building a match list (e.g. VSEARCH --usearch_global --self --id .84 with query and target coverage cutoffs, or BLASTn -perc_identity 84 -qcov_hsp_perc 80). Recompute the richness-vs-plant-richness R2 on the LULU-curated tables and tabulate the R2 improvements for singleton removal versus LULU curation (e.g. write comparison_table.csv). The rationale is that LULU merges co-occurring, sequence-similar 'daughter' OTUs into more abundant 'parent' OTUs, preserving rare but real taxa that singleton removal would discard.

## Related tools

- **LULU** (R package performing the co-occurrence-based post-clustering curation whose improvement is benchmarked against singleton removal) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the initial clustering methods producing OTU tables, and also used to build the self match list for LULU (--usearch_global --self --id .84))
- **SWARM** (Greedy clustering method producing one of the OTU tables compared; improvement by LULU was most pronounced for VSEARCH and SWARM) — https://github.com/frederic-mahe/swarm
- **CROP** (Clustering method producing one of the OTU tables compared (its OTU picking appeared suboptimal in the study))
- **DADA2** (Initial OTU-definition method producing one of the OTU tables compared; part of the recommended workflow with VSEARCH clustering and LULU curation)

## Examples

```
R: library(lulu); curated_result <- lulu(otutable_name, matchlist_name, minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95); curated_table <- curated_result$curated_table
```

## Evaluation signals

- Mean R2 improvement over uncurated tables should be approximately 0.03 for singleton removal and approximately 0.27 for LULU curation across clustering methods
- Singleton-filtered tables must contain no OTU with total read count equal to one, and per-site OTU richness is recomputed only on retained OTUs
- LULU curation must merge flagged errors with their parents preserving total read counts (curated_result$curated_table), and R2 for curated tables should exceed both uncurated and singleton-removed values
- Match list must have three columns (query OTU id, matching OTU id, percent similarity) with ids matching the OTU table row names, produced with minimum_match 84
- Comparison table should cover all clustering methods (VSEARCH, SWARM, CROP, DADA2) with per-method and mean R2 improvements for both curation strategies

## Limitations

- Singleton removal yields only marginal gains (~0.03 mean R2 improvement) compared to LULU curation (~0.27), so it is a weak alternative for alpha-diversity improvement
- The 'cost of curation' is a small loss of real rare/low-abundance species, and LULU may not be suitable for identifying single rare species
- Validation was performed only on vascular plants from 130 Danish sites; relevance to other organism groups is inferred but not directly tested
- CROP OTU picking appeared suboptimal (low similarity to best GenBank match, poor redundancy and beta-diversity measures), which can affect the comparison for that method

## Evidence

- [results] Singleton removal offered only marginal gains compared to LULU curation: the mean improvement in the coefficient of determination (R2) for OTU richness vs plant richness was 0.03 for singleton removal versus 0.27 for LULU curation.: [evidence span withheld — anchor status: not_found]
- [intro] We also tested the dbotu3 algorithm, both for 'one-step' clustering as intended, and as an alternative to LULU for post-clustering curation: "Furthermore, we evaluated the effect of singleton removal in comparison to curation"
- [intro] The algorithm is independent of a reference database, and can thus be applied to any OTU table produced for any set of samples produced by any initial OTU definition algorithm: "The algorithm is independent of a reference database, and can thus be applied to any OTU table produced for any set of samples produced by any initial OTU definition algorithm"
- [intro] OTU tables were produced from the sequence data using a set of representative clustering methods: VSEARCH22, SWARM23, CROP24 and DADA213: "OTU tables were produced from the sequence data using a set of representative clustering methods: VSEARCH22, SWARM23, CROP24 and DADA213"
- [methods] LULU was run with minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95 per the task workflow on tables from the 130 Danish sites.: "If the number of samples where the 'potential parent' has a positive presence is below the `minimum_relative_cooccurence` (default 95%, meaning that 1 in 20 samples are allowed to have no parent"
- [readme] The only requirements is that the match list has three columns with pair wise similarity scores for the OTUs.: "The only requirements is that the match list has three columns with pair wise similarity scores for the OTUs."
- [readme] VSEARCH match-list command: vsearch --usearch_global OTU_sequences.fasta --db OTU_sequences.fasta --self --id .84 --iddef 1 --userout match_list.txt -userfields query+target+id --maxaccepts 0 --query_cov .9 --maxhits 10: "vsearch --usearch_global OTU_sequences.fasta --db OTU_sequences.fasta --self --id .84 --iddef 1 --userout match_list.txt -userfields query+target+id --maxaccepts 0 --query_cov .9 --maxhits 10"
