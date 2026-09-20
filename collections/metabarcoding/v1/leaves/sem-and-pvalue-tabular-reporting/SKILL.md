---
name: sem-and-pvalue-tabular-reporting
description: Use when a species/taxon abundance table has samples split into two groups
  (e.g., I-source inflow vs. P-source pond water) and you must report which taxa differ
  significantly with mean ± SEM and p-values in a table.
when_to_use_negative:
- Input is raw FASTQ rather than a processed taxon abundance table — run the SILVAngs
  pipeline (QC, clustering, BLASTn classification) first
- The analysis goal is beta-diversity or geographic distance effects (use the Mantel
  test / IBD workflow instead of per-taxon t-tests)
- Samples cannot be assigned to a two-group design (I-source vs. P-source); the t-test
  framing does not apply
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: XLSTAT 2017
  role: Statistical add-in used to run the two-sample (one-tail) t-tests on per-taxon
    relative abundances between inflow- and pond-water groups
- name: SILVAngs
  role: Upstream pipeline that produced the OTU abundance table and taxonomic assignments
    that feed the SEM/p-value table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:1d01c77f70fddc3a76ba9d0e31893d8be5b752b134177ca1780df1bd84337750
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SILVAngs
  - XLSTAT 2017
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sem-and-pvalue-tabular-reporting

## Summary

Compute per-taxon mean relative abundance and SEM per water-source group (inflow vs. pond) and attach two-sample one-tail t-test p-values, then assemble the results as a species-level significance table (as in Table 1 of the trout-farm metabarcoding study).

## When to use

Use when a species/taxon abundance table has samples split into two groups (e.g., I-source inflow vs. P-source pond water) and you must report which taxa differ significantly with mean ± SEM and p-values in a table.

## When NOT to use

- Input is raw FASTQ rather than a processed taxon abundance table — run the SILVAngs pipeline (QC, clustering, BLASTn classification) first
- The analysis goal is beta-diversity or geographic distance effects (use the Mantel test / IBD workflow instead of per-taxon t-tests)
- Samples cannot be assigned to a two-group design (I-source vs. P-source); the t-test framing does not apply

## Inputs

- SILVAngs-derived OTU relative-abundance table with species-level taxonomic assignments (phylum/family/genus)
- Sample metadata mapping each sample to inflow- (I-source) or pond-water (P-source) group across the three trout farm locations

## Outputs

- CSV significance table (e.g., table1_significant_species_ttest.csv) with one row per significantly different taxon and columns: phylum, family, genus, mean relative abundance and SEM per group, p-value
- Set of significantly different species with direction of enrichment (up-represented in I-source or P-source)

## How to apply

Load the SILVAngs-derived OTU abundance table (reads clustered with cd-hit-est at identity criteria 1.00/0.98 and classified via local BLASTn against SILVA SSU ref release 132) together with metadata distinguishing I-source from P-source samples. Convert counts to per-taxon relative abundances and group samples by water source. For each species-level taxon, compute the mean relative abundance and the standard error of the mean (SEM) overall and separately for the I-source and P-source groups. Run two-sample t-tests on the relative abundances between inflow- and pond-water groups using XLSTAT 2017 (Addinsoft, Paris, France), as used for the paper's statistical testing. Retain only taxa with significant p-values, keep their taxonomic classification (phylum, family, genus), and write the table (here: table1_significant_species_ttest.csv) with columns for phylum, family, genus, group-wise mean and SEM, and p-value.

## Related tools

- **XLSTAT 2017** (Statistical add-in used to run the two-sample (one-tail) t-tests on per-taxon relative abundances between inflow- and pond-water groups)
- **SILVAngs** (Upstream pipeline that produced the OTU abundance table and taxonomic assignments that feed the SEM/p-value table)

## Examples

```
R> res <- t.test(relabun[taxon, I_source_samples], relabun[taxon, P_source_samples], alternative="greater")
```

## Evaluation signals

- Output CSV contains exactly 15 taxon rows for the Table 1 replication, each with phylum, family, genus, group-wise mean/SEM, and p-value populated
- Retained species belong to only two phyla (Bacteroidetes and Proteobacteria) spanning 12 families, matching the reported result
- Directionality check: Bacteroidetes species (e.g., Marinoscillum, Dysgonomonas, Paludibacter, Mucilaginibacter, Pedobacter, Lacihabitans, Empedobacter) are up-represented in P-source, while Simplicispira, Amaricoccus, and Thioclava are up-represented in I-source
- SEM values are non-negative and consistent with per-group relative abundances; p-values correspond to one-tail two-sample t-tests
- Taxon names in the output match the taxa reported as significantly different between inflow- and pond-water samples in the paper

## Limitations

- Exact farm locations were withheld at farmers' request, limiting geographic reproducibility of the grouping
- Some OTUs remained unclassified or unassigned when the BLASTn score '(% sequence identity + % alignment coverage)/2' did not exceed 93, so species-level rows can be incomplete
- Only significant taxa are retained; non-significant taxa and their effect sizes are not part of the output table

## Evidence

- [other] t-test on grouped relative abundances: [evidence span withheld — anchor status: not_found]
- [other] findings of the significant species: [evidence span withheld — anchor status: not_found]
- [discussion] XLSTAT used for statistics in this study: "tested using a Mantel test and performed using IBD v1.5259 and XLSTAT 2017 (Addinsoft, Paris, France) software"
- [discussion] OTU clustering and classification upstream of the abundance table: "clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2;52)"
