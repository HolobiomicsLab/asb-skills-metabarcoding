---
name: metabarcoding-diversity-figure-reproduction
description: Use when a practitioner needs to reproduce or extend the assessment of
  how denoising vs. clustering pipeline choices and sequence filtering regimes alter
  per-sample alpha diversity (Hill Numbers 0/1/2) in amplicon data.
when_to_use_negative:
- When the goal is to select a single best metabarcoding pipeline — the paper states
  its aim is not to present a single best pathway, only to illustrate how processes
  are affected by program or parameter choices
- When the input is already a rarefied, filtered feature table with diversity values
  computed — this skill reproduces the full pipeline from raw reads through denoising,
  filtering, and rarefaction
- When data are not COI amplicon libraries (e.g., 16S/ITS) with reference databases
  like UNITE, Greengenes, or SILVA, since host/mock COI removal and the mock-guided
  'miss' subtraction are COI-specific
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: QIIME 2
  role: Environment for importing raw reads and running DADA2/Deblur denoising pipelines
    (v2018.11)
- name: DADA2
  role: Error-model denoising pipeline producing one of the three representative-sequence
    sets compared
- name: Deblur
  role: Denoising pipeline run with non-default parameters --p-min-reads 2 and --p-min-size
    1 so only singletons are discarded
- name: VSEARCH
  role: Paired-end merging, dereplication, 98%/97% identity OTU clustering, and de
    novo chimera filtering
  repo: https://github.com/torognes/vsearch
- name: Phyloseq
  role: R package used to compute Hill Numbers 0, 1, and 2 from rarefied feature tables
- name: Vegan
  role: R package used with Phyloseq for alpha diversity estimation
- name: Tidyverse
  role: R framework for running Kruskal–Wallis and Dunn's tests and producing Figure
    3 / S2-style plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:70ee4d1f27a539bb1d564827a9cd283a333c7ac53606d1e827c73ed6d29c1cb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - DADA2
  - Deblur
  - Phyloseq
  - QIIME 2
  - Tidyverse
  - VSEARCH
  - Vegan
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabarcoding-diversity-figure-reproduction

## Summary

Reproduces the O'Rourke et al. (2020) 'tidybug' analysis showing that Hill-number alpha diversity of bat guano COI metabarcoding data remains sensitive to the choice of denoising program (DADA2, Deblur, VSEARCH) and filtering regime, even after rarefaction to 5,000 reads per sample.

## When to use

Use when a practitioner needs to reproduce or extend the assessment of how denoising vs. clustering pipeline choices and sequence filtering regimes alter per-sample alpha diversity (Hill Numbers 0/1/2) in amplicon data.

## When NOT to use

- When the goal is to select a single best metabarcoding pipeline — the paper states its aim is not to present a single best pathway, only to illustrate how processes are affected by program or parameter choices
- When the input is already a rarefied, filtered feature table with diversity values computed — this skill reproduces the full pipeline from raw reads through denoising, filtering, and rarefaction
- When data are not COI amplicon libraries (e.g., 16S/ITS) with reference databases like UNITE, Greengenes, or SILVA, since host/mock COI removal and the mock-guided 'miss' subtraction are COI-specific

## Inputs

- Raw paired-end COI amplicon FASTQ reads from SRA PRJNA518082 (bat guano and mock community libraries)
- Denoised/clustered feature tables from DADA2, Deblur (--p-min-reads 2 --p-min-size 1), and VSEARCH (98% then 97% identity clustering) pipelines
- Mock community feature table for deriving the per-library 'miss' subtraction integer used in the extra filter
- R environment with Phyloseq, Vegan, and Tidyverse packages

## Outputs

- Rarefied (5,000 reads/sample) filtered feature tables per denoising program and filtering regime
- Per-sample Hill Number 0, 1, and 2 (observed richness, Shannon's Entropy, Simpson's Index) values for every guano sample
- Kruskal–Wallis test statistics and Dunn's post hoc pairwise comparisons among denoising programs per filtering regime
- Plots reproducing the Figure 3 / Figure S2 pattern of alpha diversity by denoising program and filtering regime

## How to apply

Import raw paired-end guano reads from SRA PRJNA518082 into QIIME 2, trim adapters with Cutadapt, and generate representative sequences via three routes: DADA2 error-model denoising, Deblur with '--p-min-reads 2 --p-min-size 1', and VSEARCH OTU clustering (merge, dereplicate, cluster at 98% then 97% identity with de novo chimera filtering). After removing host/mock COI sequences, apply three filtering regimes to each feature table: basic (none), standard (drop variants seen in only one sample; retain samples with ≥5,000 total filtered reads), and extra (standard plus subtracting the maximum count of a 'miss' mock sequence variant from each table element). Rarefy to 5,000 reads per sample and compute Hill Numbers 0, 1, and 2 per guano sample with Phyloseq/Vegan in R, then test among-program differences with Kruskal–Wallis tests per filtering regime and Hill Number followed by Dunn's post hoc pairwise tests, and plot per-sample estimates to match Figures 3 and S2. The rationale is that the paper found denoising program and filtering regime significantly influenced diversity estimates whether read-abundance or presence/absence information was used.

## Related tools

- **QIIME 2** (Environment for importing raw reads and running DADA2/Deblur denoising pipelines (v2018.11))
- **DADA2** (Error-model denoising pipeline producing one of the three representative-sequence sets compared)
- **Deblur** (Denoising pipeline run with non-default parameters --p-min-reads 2 and --p-min-size 1 so only singletons are discarded)
- **VSEARCH** (Paired-end merging, dereplication, 98%/97% identity OTU clustering, and de novo chimera filtering) — https://github.com/torognes/vsearch
- **Phyloseq** (R package used to compute Hill Numbers 0, 1, and 2 from rarefied feature tables)
- **Vegan** (R package used with Phyloseq for alpha diversity estimation)
- **Tidyverse** (R framework for running Kruskal–Wallis and Dunn's tests and producing Figure 3 / S2-style plots)

## Examples

```
vsearch --cluster_fast guano_merged.fasta --id 0.97 --centroids otus97.fasta
```

## Evaluation signals

- Kruskal–Wallis tests identify significant differences in Hill-number alpha diversity among DADA2, Deblur, and VSEARCH samples for each filtering regime, with Hill Number 0 on basic-filtered data as the expected exception
- VSEARCH-processed samples repeatedly contain 2–3x more ASVs per sample than DADA2 or Deblur
- All rarefied tables are normalized to exactly 5,000 reads per sample before diversity computation
- Per-sample diversity plots match the qualitative pattern reported in Figures 3 and S2 of the paper
- Dunn's post hoc tests confirm pairwise program differences consistent with the reported significant among-group comparisons

## Limitations

- Software versions for the denoising programs and classifiers are not reported in the results section
- Differences between denoising and clustering methods are largely mitigated by removing low-abundance sequence variants, so filtering regime choice strongly conditions the outcome
- The exact per-library 'miss' subtraction integer for the extra filter must be derived from the mock community data; it is not a fixed universal constant
- VSEARCH clustering parameters mirrored the VSEARCH Wiki GitHub pipeline, so results depend on those parameter choices

## Evidence

- [methods] a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset"
- [methods] extra filter definition: "an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table"
- [abstract] denoising vs clustering error rates: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
- [intro] study scope on diversity metrics: [evidence span withheld — anchor status: not_found]
- [readme] repo documentation pointer: "See [the docs folder of this repo](https://github.com/devonorourke/tidybug/tree/master/docs) for information on how each part of the project was performed"
