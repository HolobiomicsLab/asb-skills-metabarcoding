---
name: microbial-taxa-relative-abundance-quantification
description: Use when an OTU/species abundance table with taxonomic assignments and
  sample metadata needs relative-abundance normalization, group-wise mean/SEM summarization,
  and significance testing between sample groups.
when_to_use_negative:
- Input is raw FASTQ rather than a classified OTU abundance table — run the SILVAngs
  pipeline (BBmerge, BBduk, cd-hit-est, BLASTn) first
- Goal is alpha/beta diversity or Mantel-test analysis rather than taxon-level abundance
  comparison (use the Simpson/Bray-Curtis workflow instead)
- ITS/eukaryotic data classified with the Warcup V2 'Classify Sequences' tool at 90%
  similarity — this skill targets the 16S SILVAngs BLASTn-based prokaryotic table
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0622
tools:
- name: SILVAngs
  role: Online automatic pipeline producing the OTU abundance table with taxonomic
    classifications that relative abundances are computed from
- name: cd-hit-est v3.1.2
  role: Clusters quality-filtered reads into OTUs at identity criteria 1.00 and 0.98
    before abundance quantification
- name: BLASTn
  role: Local search against SILVA SSU ref release 132 assigning taxonomy to OTUs;
    weak hits ((% identity + % coverage)/2 <= 93) stay unclassified
- name: XLSTAT 2017
  role: Two-sample one-tail t-tests on relative abundances between I-source and P-source
    groups
- name: BBduk
  role: Upstream quality trimming (Q 30) and discarding reads below 50 bp that feed
    the OTU table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:106510fc238e8bb20c805d36da90ee3bddbc6d245a8ade758c5b5cdd2106a0e6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBduk
  - BLASTn
  - SILVAngs
  - XLSTAT 2017
  - cd-hit-est v3.1.2
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# microbial-taxa-relative-abundance-quantification

## Summary

Compute per-taxon relative abundances from a SILVAngs-derived 16S OTU table (cd-hit-est clustered, BLASTn-classified against SILVA SSU ref 132), aggregate mean abundance and SEM per sample group (e.g. inflow- vs pond-water), and test species-level differences with two-sample t-tests in XLSTAT 2017.

## When to use

Use when an OTU/species abundance table with taxonomic assignments and sample metadata needs relative-abundance normalization, group-wise mean/SEM summarization, and significance testing between sample groups.

## When NOT to use

- Input is raw FASTQ rather than a classified OTU abundance table — run the SILVAngs pipeline (BBmerge, BBduk, cd-hit-est, BLASTn) first
- Goal is alpha/beta diversity or Mantel-test analysis rather than taxon-level abundance comparison (use the Simpson/Bray-Curtis workflow instead)
- ITS/eukaryotic data classified with the Warcup V2 'Classify Sequences' tool at 90% similarity — this skill targets the 16S SILVAngs BLASTn-based prokaryotic table

## Inputs

- SILVAngs-derived OTU abundance table with taxonomic assignments (species/phylum/family/genus columns)
- Sample metadata mapping samples to groups (inflow I-source vs pond-water P-source, three trout farm locations)
- Sample-to-FASTQ provenance from MiSeq 3000 runs (V3-V4 16S amplicons)

## Outputs

- Per-taxon relative abundance matrix grouped by sample group
- Table of significantly different species with phylum, family, genus, mean relative abundance and SEM per group, and p value (CSV, e.g. table1_significant_species_ttest.csv)

## How to apply

Load the SILVAngs-derived OTU abundance table, produced by clustering reads with cd-hit-est v3.1.2 at identity criteria 1.00 and 0.98 and classifying via local BLASTn against the SILVA SSU ref dataset release 132 (reads whose (% sequence identity + % alignment coverage)/2 did not exceed 93 remain unclassified), together with metadata distinguishing inflow- (I-source) and pond-water (P-source) samples. Convert counts to per-taxon relative abundances and group samples by water source. For each species-level taxon, calculate the mean relative abundance and the standard error of the mean (SEM) overall and per group. Run two-sample one-tail t-tests on relative abundances between groups in XLSTAT 2017 (Addinsoft, Paris, France), as used for the paper's statistical testing. Retain taxa with significant p values and assemble a results table with phylum, family, genus, mean abundance and SEM per group, and p value (in the paper this yielded 15 significantly different species from Bacteroidetes and Proteobacteria).

## Related tools

- **SILVAngs** (Online automatic pipeline producing the OTU abundance table with taxonomic classifications that relative abundances are computed from)
- **cd-hit-est v3.1.2** (Clusters quality-filtered reads into OTUs at identity criteria 1.00 and 0.98 before abundance quantification)
- **BLASTn** (Local search against SILVA SSU ref release 132 assigning taxonomy to OTUs; weak hits ((% identity + % coverage)/2 <= 93) stay unclassified)
- **XLSTAT 2017** (Two-sample one-tail t-tests on relative abundances between I-source and P-source groups)
- **BBduk** (Upstream quality trimming (Q 30) and discarding reads below 50 bp that feed the OTU table)

## Examples

```
from scipy.stats import ttest_ind; stat, p = ttest_ind(i_source_relab, p_source_relab, alternative='less')
```

## Evaluation signals

- Output table contains exactly the expected number of significantly different taxon rows (15 species in the paper) with phylum, family, genus, mean/SEM and p value populated
- Significant taxa match the paper's reported genera (Marinoscillum, Dysgonomonas, Paludibacter, Mucilaginibacter, Pedobacter, Lacihabitans, Empedobacter, Simplicispira, Amaricoccus, Thioclava, Alicycliphilus, Caenimonas, Orientia, Sphingomonadaceae bacterium) restricted to Bacteroidetes and Proteobacteria
- Up-representation directions are consistent: Bacteroidetes species enriched in P-source; Proteobacteria species split between I-source (Simplicispira, Amaricoccus, Thioclava) and P-source (Alicycliphilus, Caenimonas, Orientia, Sphingomonadaceae bacterium)
- Relative abundances per sample sum to 1 and mean/SEM values are reproducible from the grouped per-taxon abundances
- Unclassified OTUs (BLAST score <= 93) are excluded or flagged rather than counted as species

## Limitations

- Statistical testing was performed with XLSTAT 2017, a GUI tool, so exact test settings (tail specification, variance assumptions) must be documented separately for reproducibility
- Some OTUs remained unclassified or unassigned due to weak BLAST hits, reducing species-level completeness of the abundance table
- Exact farm locations were withheld at farmers' request, limiting geographic reproducibility of the group comparison
- One-tail t-tests on relative abundances of 15 selected species did not mention multiple-testing correction

## Evidence

- [discussion] SILVAngs pipeline produced classified OTU table: "SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline based on the SILVA rDNA database"
- [discussion] OTU clustering with cd-hit-est v3.1.2: "clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2"
- [discussion] BLASTn classification against SILVA SSU ref 132: "based on local BLASTn search against the non-redundant version of SILVA SSU ref dataset (release no. 132)"
- [discussion] XLSTAT 2017 used for the paper's statistical testing: [evidence span withheld — anchor status: not_found]
- [abstract] Prokaryotic composition dominated by Proteobacteria and Bacteroidetes: "more than 99% of the detected prokaryotic OTUs (Operational Taxonomic Unit identification) were bacteria as of ~ 75.57% were Proteobacteria, and ~ 14.4% were Bacteroidetes"
