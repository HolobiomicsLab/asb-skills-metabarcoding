---
name: curation-benchmark-against-field-survey-ground-truth
description: Use when you have OTU tables from multiple clustering methods (e.g. VSEARCH,
  SWARM, CROP, DADA2) plus an independent species inventory, and must decide which
  curation method yields realistic biodiversity metrics.
when_to_use_negative:
- There is no independent ground-truth inventory (paired survey data or comprehensive
  reference database) to validate against — curation effect cannot be evaluated as
  designed
- 'The goal is identifying single rare species: the article notes LULU curation may
  not be suitable for that, and carries a ''cost of curation'' — a small loss of real,
  rare, low-abundance species'
- The data is already a curated feature table and the question is downstream ecology
  rather than comparing curation methods
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: LULU
  role: Co-occurrence based post-clustering curation of OTU tables; benchmarked method
    whose curation effect is validated against field survey data
  repo: https://github.com/tobiasgf/lulu
- name: dbotu3
  role: Alternative algorithm run both for one-step clustering as intended and as
    post-clustering curation, benchmarked against LULU
- name: VSEARCH
  role: Produces initial OTU tables and can generate the LULU match list via --usearch_global
    --self
- name: SWARM
  role: One of the representative initial clustering methods whose tables are curated
    and benchmarked
  repo: https://github.com/frederic-mahe/swarm
- name: CROP
  role: One of the representative initial clustering methods whose tables are curated
    and benchmarked
- name: DADA2
  role: One of the representative initial clustering methods (recommended for initial
    OTU definition in the final workflow)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:ae1dfc871047274800c690cf70272534fa36dfa351006652140d9e1c183c6a2e
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
  - dbotu3
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# curation-benchmark-against-field-survey-ground-truth

## Summary

Benchmark post-clustering curation methods (LULU vs. dbotu3 as post-clustering curation, plus singleton removal) by curating OTU tables from representative clustering algorithms and evaluating curation effect against independent field survey ground truth.

## When to use

Use when you have OTU tables from multiple clustering methods (e.g. VSEARCH, SWARM, CROP, DADA2) plus an independent species inventory, and must decide which curation method yields realistic biodiversity metrics.

## When NOT to use

- There is no independent ground-truth inventory (paired survey data or comprehensive reference database) to validate against — curation effect cannot be evaluated as designed
- The goal is identifying single rare species: the article notes LULU curation may not be suitable for that, and carries a 'cost of curation' — a small loss of real, rare, low-abundance species
- The data is already a curated feature table and the question is downstream ecology rather than comparing curation methods

## Inputs

- OTU tables (samples as columns, unique OTU ids as rows) produced by VSEARCH, SWARM, CROP and/or DADA2
- File of representative OTU centroid sequences (e.g. centroids_test.txt, 2425 OTUs over 130 samples)
- Match list with columns query OTU id, target OTU id, percent similarity, produced by BLASTn or VSEARCH
- Independent ground-truth species inventory (paired field survey data, e.g. vascular plant records per site)
- GenBank reference data for taxonomic assignment of OTUs (optional validation layer)

## Outputs

- Curated OTU tables (LULU-curated, dbotu3-curated, singleton-removed)
- Benchmark table of curation validation metrics (OTU counts, α-diversity, taxonomic redundancy, best-match similarity) per clustering/curation combination
- Comparative figures mirroring Supplementary Figs 8–15 (LULU vs dbotu3)
- Conclusion on which curation method gives the most realistic biodiversity estimates

## How to apply

Load the initial OTU tables (e.g. 20 tables from VSEARCH, SWARM, CROP and DADA2 on the plant ITS2 soil dataset of 130 Danish sites) and, where required, generate a match list with BLASTn or VSEARCH (e.g. vsearch --usearch_global --self --id .84 --iddef 1 --userfields query+target+id --maxaccepts 0 --query_cov .9 --maxhits 10). Curate each table with the LULU R function and, in parallel, run dbotu3 both in its one-step clustering mode and as post-clustering curation (genetic dissimilarity 0.16, abundance criteria 0 and 10), and evaluate singleton removal for comparison. For every curated table compute curation validation metrics — OTU counts, α-diversity estimates, taxonomic redundancy, β-diversity, and similarity of OTUs to best GenBank matches — against the field survey ground truth (564 species, per-site richness 6–93). Assemble the metrics into comparative benchmark tables/figures mirroring Supplementary Figs 8–15 and judge success by whether curation improves the metrics relative to the uncurated tables and whether LULU outperforms the alternative.

## Related tools

- **LULU** (Co-occurrence based post-clustering curation of OTU tables; benchmarked method whose curation effect is validated against field survey data) — https://github.com/tobiasgf/lulu
- **dbotu3** (Alternative algorithm run both for one-step clustering as intended and as post-clustering curation, benchmarked against LULU)
- **VSEARCH** (Produces initial OTU tables and can generate the LULU match list via --usearch_global --self)
- **SWARM** (One of the representative initial clustering methods whose tables are curated and benchmarked) — https://github.com/frederic-mahe/swarm
- **CROP** (One of the representative initial clustering methods whose tables are curated and benchmarked)
- **DADA2** (One of the representative initial clustering methods (recommended for initial OTU definition in the final workflow))

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name)
```

## Evaluation signals

- Curation should reduce OTU counts while improving α-diversity estimates relative to the field survey ground truth (per-site richness 6–93, 564 species overall)
- Curated tables should show reduced taxonomic redundancy and higher similarity of OTUs to best GenBank matches than uncurated tables
- dbotu3 as post-clustering curation should improve metrics for both abundance settings (0 and 10), yet LULU should outperform it in all comparisons except a few metrics such as average best match for the CROP tables
- Match list schema check: three columns of query OTU id, matching OTU id, and percent similarity score
- The curated table's total read count is preserved because errors are merged (not discarded) with their parent OTUs

## Limitations

- Validation was performed only on vascular plants; relevance to other organism groups is inferred but not directly tested
- Curation's 'cost' is a small loss of real (rare, low-abundance) species, and LULU may not be suitable for identifying single rare species
- No validation on mock communities with known genetic content has been performed (noted as future work), and mock communities lack the complexity of real soil samples
- CROP OTU picking appeared suboptimal (low best-GenBank-match similarity, poor redundancy and β-diversity), which can bias method comparisons on CROP tables

## Evidence

- [abstract] LULU curation consistently improves α-diversity estimates and other biodiversity metrics: "LULU curation consistently improves α-diversity estimates and other biodiversity metrics"
- [abstract] OTU tables produced with several algorithms then curated and validated against field survey data: "OTU tables are produced with several different OTU definition algorithms and subsequently curated with LULU, and validated against field survey data"
- [intro] dbotu3 tested as alternative curation method: "We also tested the dbotu3 algorithm, both for ‘one-step’ clustering as intended, and as an alternative to LULU for post-clustering curation"
- [intro] Representative clustering methods for initial OTU tables: "OTU tables were produced from the sequence data using a set of representative clustering methods: VSEARCH22, SWARM23, CROP24 and DADA213"
- [discussion] LULU merge mechanism preserving read counts: "After identification of these extra OTUs, they can be merged with their parent OTUs in order to preserve the total read count and reduce the OTU number of the table"
- [discussion] Match list production with external algorithm: [evidence span withheld — anchor status: not_found]
- [readme] README VSEARCH command for match list: "vsearch --usearch_global OTU_sequences.fasta --db OTU_sequences.fasta --self --id .84 --iddef 1 --userout match_list.txt -userfields query+target+id --maxaccepts 0 --query_cov .9 --maxhits 10"
- [readme] README requirement of OTU table and matchlist: "The required input of **LULU** is an OTU table and a corresponding matchlist with all the internal matches of OTUs."
