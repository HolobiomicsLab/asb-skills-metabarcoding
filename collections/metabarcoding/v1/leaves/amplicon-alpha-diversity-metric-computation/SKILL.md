---
name: amplicon-alpha-diversity-metric-computation
description: Use when you have OTU tables (e.g. from VSEARCH, SWARM, CROP or DADA2)
  and need to quantify per-sample richness/α-diversity and judge whether post-clustering
  curation (LULU, dbotu3, singleton removal) brings molecular estimates closer to
  field-survey ground truth.
when_to_use_negative:
- The goal is identifying single rare species — the paper states 'LULU curation may
  thus not be suitable for identifying single rare species in community data sets'
  and the 'cost of curation' is a small loss of real rare species.
- The input is raw reads rather than an OTU/feature table with an accompanying match
  list — LULU requires an OTU table plus a precomputed match list.
- Composition/turnover is the only target and α-diversity is irrelevant — the paper
  notes rank-abundance-tail-discarding approaches 'may be adequate for assessing composition
  and turnover, but much less so for α-diversity'.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: LULU
  role: Post-clustering curation of OTU tables prior to α-diversity computation; removes
    erroneous OTUs by co-occurrence and sequence similarity, improving α-diversity
    estimates
  repo: https://github.com/tobiasgf/lulu
- name: dbotu3
  role: 'Benchmarked alternative to LULU: run both as one-step clustering and as post-clustering
    curation before computing α-diversity metrics'
- name: VSEARCH
  role: Initial OTU table production and match-list generation (--usearch_global --self)
    feeding the α-diversity comparison workflow
- name: SWARM
  role: Representative greedy clustering method producing OTU tables whose curated
    α-diversity is benchmarked
  repo: https://github.com/frederic-mahe/swarm
- name: CROP
  role: Representative clustering method producing OTU tables for the α-diversity
    benchmark (noted as suboptimal in the paper)
- name: DADA2
  role: Initial OTU definition method recommended by the authors before clustering
    and LULU curation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:fd2680a15839c456630267f4e5a1b4f78f2de41cf63623e2e0dd598b359989bd
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# amplicon-alpha-diversity-metric-computation

## Summary

Compute per-sample α-diversity (species richness) from curated or uncurated amplicon OTU tables and validate the estimates against independent ground-truth survey data, as done in the LULU paper where plant ITS2 metabarcoding of 130 Danish soil sites was compared to botanical field surveys.

## When to use

Use when you have OTU tables (e.g. from VSEARCH, SWARM, CROP or DADA2) and need to quantify per-sample richness/α-diversity and judge whether post-clustering curation (LULU, dbotu3, singleton removal) brings molecular estimates closer to field-survey ground truth.

## When NOT to use

- The goal is identifying single rare species — the paper states 'LULU curation may thus not be suitable for identifying single rare species in community data sets' and the 'cost of curation' is a small loss of real rare species.
- The input is raw reads rather than an OTU/feature table with an accompanying match list — LULU requires an OTU table plus a precomputed match list.
- Composition/turnover is the only target and α-diversity is irrelevant — the paper notes rank-abundance-tail-discarding approaches 'may be adequate for assessing composition and turnover, but much less so for α-diversity'.

## Inputs

- OTU table (samples as columns, OTUs as rows, unique OTU ids) from VSEARCH, SWARM, CROP or DADA2
- Optionally curated OTU tables produced by LULU or dbotu3 (genetic dissimilarity 0.16, abundance criteria 0 and 10)
- Ground-truth field survey / inventory data paired with the same sites (e.g. vascular plant survey of 130 Danish sites, 564 species)
- Match list (OTU1, OTU2, % similarity) produced by BLASTn or VSEARCH for curation steps

## Outputs

- Per-sample α-diversity (richness) estimates per OTU-table variant (uncurated, LULU-curated, dbotu3-curated, singleton-removed)
- Benchmark table comparing molecular α-diversity against field-survey ground truth (mirroring Supplementary Figs 8–15)
- Curated OTU tables with preserved total read counts

## How to apply

Start from the OTU table with samples as columns and unique OTU ids as rows (any producer algorithm, e.g. VSEARCH, SWARM, CROP or DADA2 on plant ITS2 soil amplicon data). For each table variant — uncurated, LULU-curated (via a BLASTn or VSEARCH match list with, e.g. --id .84, qcov 80/90), dbotu3-curated (genetic dissimilarity 0.16, abundance criteria 0 and 10), or singleton-removed — compute per-sample OTU counts as α-diversity estimates. Compare these estimates against paired ground-truth inventory data (the paper's botanical survey: 564 species, per-site richness 6–93 across 130 Danish sites) using metrics such as OTU counts, α-diversity agreement, taxonomic redundancy and best-GenBank-match similarity. Correct application means the curated table retains read totals (errors are merged into parents, preserving total read count) while moving per-site richness estimates closer to the survey-based values.

## Related tools

- **LULU** (Post-clustering curation of OTU tables prior to α-diversity computation; removes erroneous OTUs by co-occurrence and sequence similarity, improving α-diversity estimates) — https://github.com/tobiasgf/lulu
- **dbotu3** (Benchmarked alternative to LULU: run both as one-step clustering and as post-clustering curation before computing α-diversity metrics)
- **VSEARCH** (Initial OTU table production and match-list generation (--usearch_global --self) feeding the α-diversity comparison workflow)
- **SWARM** (Representative greedy clustering method producing OTU tables whose curated α-diversity is benchmarked) — https://github.com/frederic-mahe/swarm
- **CROP** (Representative clustering method producing OTU tables for the α-diversity benchmark (noted as suboptimal in the paper))
- **DADA2** (Initial OTU definition method recommended by the authors before clustering and LULU curation)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name); alpha_diversity <- rowSums(curated_result$curated_table > 0)
```

## Evaluation signals

- Curated tables preserve total read counts because errors are merged with their parent OTUs rather than discarded.
- α-diversity estimates from curated tables move closer to the paired field-survey richness (ground truth: 6–93 species per site across 130 sites).
- LULU curation consistently improves α-diversity estimates and other biodiversity metrics relative to the uncurated table.
- Benchmark tables (mirroring Supplementary Figs 8–15) show LULU outperforming dbotu3 across curation metrics except a few comparisons, e.g. average best match for CROP tables.
- OTU table schema is valid: samples as columns, unique OTU ids as row names matching the sequence/match-list ids.

## Limitations

- The 'cost of curation' is a small loss of 'real species' (rare, low-abundance OTUs), so α-diversity may be slightly underestimated after curation.
- CROP OTU picking appeared suboptimal (low similarity to best GenBank match, poor redundancy and β-diversity measures), affecting downstream α-diversity benchmarks.
- Validation was performed only for vascular plants with a paired inventory; relevance to other organism groups is inferred but not directly tested.

## Evidence

- [abstract] LULU curation consistently improves α-diversity estimates and other biodiversity metrics: "LULU curation consistently improves α-diversity estimates and other biodiversity metrics"
- [intro] field survey ground truth dataset: "A total of 564 plant species (approximately one third of the naturally occurring flora) were recorded in the study, with field site species richness per site ranging from 6 to 93"
- [discussion] curation preserves reads: "After identification of these extra OTUs, they can be merged with their parent OTUs in order to preserve the total read count and reduce the OTU number of the table"
- [readme] README invocation: "curated_result <- lulu(otutable_name, matchlist_name)"
- [discussion] match list production: [evidence span withheld — anchor status: not_found]
