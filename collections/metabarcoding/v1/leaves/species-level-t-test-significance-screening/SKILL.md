---
name: species-level-t-test-significance-screening
description: Use when a SILVAngs-derived OTU abundance table with taxonomic assignments
  must be compared between two sample groups (e.g., inflow- vs. pond-water) to identify
  significantly different species-level taxa.
when_to_use_negative:
- The input is raw FASTQ data rather than an OTU abundance table — pre-processing
  (BBmerge, BBduk, SILVAngs classification) must happen first.
- The comparison spans more than two sample groups or uses paired samples; a simple
  two-sample t-test on I- vs. P-source relative abundances is not the right design.
- The goal is beta-diversity or geographic-distance testing (Mantel test with IBD
  v1.52 and XLSTAT 2017) rather than species-level abundance differences.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: XLSTAT 2017
  role: Runs the two-sample t-tests on species-level relative abundances between inflow-
    and pond-water groups (Addinsoft, Paris, France)
- name: SILVAngs
  role: Online automatic pipeline producing the OTU abundance table and taxonomic
    classifications that are the input to the t-tests
- name: cd-hit-est v3.1.2
  role: Clusters reads into OTUs at identity criteria 1.00 and 0.98 before taxonomic
    classification
- name: BLASTn
  role: Assigns taxonomy via local search against the non-redundant SILVA SSU ref
    dataset (release no. 132)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:1916f60803bccacbb8537b47dc2ca77a776207961abe2b0e39fa3d35cd0269ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - SILVAngs
  - XLSTAT 2017
  - cd-hit-est v3.1.2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# species-level-t-test-significance-screening

## Summary

Screens species-level OTU relative abundances for significant differences between inflow- (I-source) and pond-water (P-source) trout-farm samples using two-sample t-tests in XLSTAT 2017, reproducing the 15 significantly different taxa reported in the paper's Table 1.

## When to use

Use when a SILVAngs-derived OTU abundance table with taxonomic assignments must be compared between two sample groups (e.g., inflow- vs. pond-water) to identify significantly different species-level taxa.

## When NOT to use

- The input is raw FASTQ data rather than an OTU abundance table — pre-processing (BBmerge, BBduk, SILVAngs classification) must happen first.
- The comparison spans more than two sample groups or uses paired samples; a simple two-sample t-test on I- vs. P-source relative abundances is not the right design.
- The goal is beta-diversity or geographic-distance testing (Mantel test with IBD v1.52 and XLSTAT 2017) rather than species-level abundance differences.

## Inputs

- SILVAngs-derived OTU abundance table with taxonomic assignments (cd-hit-est identity 1.00/0.98; BLASTn vs SILVA SSU ref release 132)
- Sample metadata distinguishing inflow- (I-source) from pond-water (P-source) samples across the three trout farm locations

## Outputs

- table1_significant_species_ttest.csv with 15 taxon rows containing phylum, family, genus, mean relative abundance and SEM per group, and p value

## How to apply

Load the SILVAngs-derived OTU table (reads clustered with cd-hit-est v3.1.2 at identity criteria 1.00/0.98 and classified via local BLASTn against SILVA SSU ref release 132) together with sample metadata distinguishing I-source from P-source samples. Compute per-taxon relative abundances and group samples by water source, calculating the mean relative abundance and SEM per group. Run two-sample t-tests on relative abundances for each species-level taxon between the inflow- and pond-water groups using XLSTAT 2017 (Addinsoft, Paris, France), matching the paper's statistical testing. Select the taxa with significant p values — expected to yield 15 species from two phyla (Bacteroidetes and Proteobacteria) and 12 families — and assemble a table with phylum, family, genus, mean relative abundance and SEM per group, and p value. Write the result as a CSV (table1_significant_species_ttest.csv) and verify it contains exactly 15 taxon rows consistent with the paper's reported taxa (Bacteroidetes such as Marinoscillum, Dysgonomonas, Paludibacter, Mucilaginibacter, Pedobacter, Lacihabitans, Empedobacter up-represented in P-source; Simplicispira, Amaricoccus, Thioclava up-represented in I-source; Alicycliphilus, Caenimonas, Orientia, Sphingomonadaceae bacterium up-represented in P-source).

## Related tools

- **XLSTAT 2017** (Runs the two-sample t-tests on species-level relative abundances between inflow- and pond-water groups (Addinsoft, Paris, France))
- **SILVAngs** (Online automatic pipeline producing the OTU abundance table and taxonomic classifications that are the input to the t-tests)
- **cd-hit-est v3.1.2** (Clusters reads into OTUs at identity criteria 1.00 and 0.98 before taxonomic classification)
- **BLASTn** (Assigns taxonomy via local search against the non-redundant SILVA SSU ref dataset (release no. 132))

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(i_source_abund, p_source_abund)  # two-sample t-test per taxon, as in XLSTAT 2017 workflow
```

## Evaluation signals

- Output table exists and contains exactly 15 taxon rows with phylum, family, genus, abundance/SEM, and p-value fields populated.
- Only two phyla appear among significant taxa (Bacteroidetes and Proteobacteria), spanning 12 families, as reported in the paper.
- Direction of difference is consistent with the paper: Bacteroidetes taxa up-represented in P-source; Simplicispira, Amaricoccus, and Thioclava up-represented in I-source; Alicycliphilus, Caenimonas, Orientia, and Sphingomonadaceae bacterium up-represented in P-source.
- Per-group mean relative abundance and SEM values are computed on relative (not raw count) abundances grouped by water source.
- Taxon names are traceable to SILVAngs/BLASTn assignments (reads with BLAST score (% identity + % alignment coverage)/2 ≤ 93 should have been unclassified and excluded).

## Limitations

- Exact farm locations were withheld at farmers' request, limiting geographic reproducibility of the three-location comparison.
- Some OTUs remained unclassified or unassigned because weak BLAST hits (score not exceeding 93) could not be classified, so species-level coverage is incomplete.
- The paper reports biodiversity and microbial enrichment in pond-water as more related to location than water source, so location effects may confound the I- vs. P-source contrast.

## Evidence

- [abstract] more than 99% of the detected prokaryotic OTUs (Operational Taxonomic Unit identification) were bacteria as of ~ 75.57% were Proteobacteria, and ~ 14.4% were Bacteroidetes: "more than 99% of the detected prokaryotic OTUs (Operational Taxonomic Unit identification) were bacteria as of ~ 75.57% were Proteobacteria, and ~ 14.4% were Bacteroidetes"
- [discussion] clustered and assigned as OTUs using cd-hit-est v3.1.2: "clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2;52)"
- [discussion] based on local BLASTn search against the non-redundant version of SILVA SSU ref dataset (release no. 132): "based on local BLASTn search against the non-redundant version of SILVA SSU ref dataset (release no. 132)"
- [discussion] tested using a Mantel test and performed using IBD v1.5259 and XLSTAT 2017 (Addinsoft, Paris, France) software: "tested using a Mantel test and performed using IBD v1.5259 and XLSTAT 2017 (Addinsoft, Paris, France) software"
