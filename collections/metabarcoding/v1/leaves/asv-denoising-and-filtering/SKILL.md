---
name: asv-denoising-and-filtering
description: Use when raw demultiplexed metabarcoding FASTQ files must be converted
  to a filtered ASV table and false positives removed using negative controls, mock
  communities, and replicates, or when benchmarking sensitivity/precision against
  other denoising pipelines.
when_to_use_negative:
- When the input is already a curated ASV/feature table rather than raw demultiplexed
  FASTQ reads — the denoising and filtering commands expect sequence data
- When the study design lacks negative controls, mock samples, or replicates, since
  VTAM's parametrization explicitly uses these experimental elements to set filtering
  thresholds
- When benchmarking 16S bacterial data against commercial mock communities only (bacteria/fungi
  mocks exist, but metazoa/plants require custom mock construction)
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3941
tools:
- name: VTAM
  role: 'Primary pipeline: merge, sortreads, filter, optimize, make_known_occurrences,
    and taxassign commands produce the control-validated ASV table'
  repo: https://github.com/aitgon/vtam
- name: DADA2
  role: Denoising engine of the DALU comparison pipeline (v1.12.1) with marker-specific
    truncation lengths
- name: LULU
  role: Curation step (v0.1.0) in the DALU pipeline to merge spurious ASVs
- name: OBITools3
  role: obi uniq/grep/clean denoising (v3.0.1b19) in the OBIbaR comparison pipeline
- name: metabaR
  role: Filtering (v1.0.0) and replicate pooling in the OBIbaR pipeline
- name: cutadapt
  role: Dependency installed with VTAM for read processing
- name: vsearch
  role: Dependency used by VTAM (e.g., chimera handling) installed via bioconda
- name: Snakemake
  role: Workflow manager orchestrating the vtam_benchmark Snakemake files (01–05)
  repo: https://snakemake.readthedocs.io/en/stable/getting_started/installation.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:f10a08684a9d1e8b23a51f98256a086e7e07dcbae7b518225d892b8436c7a2db
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aitgon/vtam_benchmark
  repo_url: https://github.com/aitgon/vtam
  tools:
  - DADA2
  - LULU
  - OBITools3
  - Python
  - R
  - Snakemake
  - VTAM
  - cutadapt
  - metabaR
  - vsearch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# ASV denoising and filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Denoise metabarcoding FASTQ reads into ASVs and rigorously filter false positives (PCR/sequencing errors, tag-jump, chimeras, pseudogenes) using control samples, benchmarking VTAM's control-based parametrization against DADA2-based (DALU) and OBITools3-based (OBIbaR) pipelines on bat, fish, and shark COI/16S datasets.

## When to use

Use when raw demultiplexed metabarcoding FASTQ files must be converted to a filtered ASV table and false positives removed using negative controls, mock communities, and replicates, or when benchmarking sensitivity/precision against other denoising pipelines.

## When NOT to use

- When the input is already a curated ASV/feature table rather than raw demultiplexed FASTQ reads — the denoising and filtering commands expect sequence data
- When the study design lacks negative controls, mock samples, or replicates, since VTAM's parametrization explicitly uses these experimental elements to set filtering thresholds
- When benchmarking 16S bacterial data against commercial mock communities only (bacteria/fungi mocks exist, but metazoa/plants require custom mock construction)

## Inputs

- Demultiplexed, trimmed FASTQ/FASTA files for bat (357 fecal pellet samples), fish (35 Zingel asper excrement samples), and shark (63 gut content samples) datasets
- COInr COI reference database and RDP Classifier 16S trainsetNo18 formatted for VTAM
- Negative control, mock community, and technical replicate sample metadata (Fasta/Tsv inputs for VTAM filter/optimize and make_known_occurrences)
- Snakemake workflow files (01snkfl_prep.yml through 05snkfl_min_readcount.yml) from the vtam_benchmark repository

## Outputs

- VTAM ASV table with taxonomic assignments (asvtable_taxa output via taxassign)
- out/summary_min_readcount_XX/control_counts.tsv with true/false positive occurrence counts from add_filters.pl count_false_occ
- Sensitivity (TP/(TP+FN)) and precision (TP/(TP+FP)) values per pipeline and read count cut-off, plotted by scripts/plt_sensitivity_precision.py

## How to apply

Run VTAM's merge and sortreads (default values) to produce demultiplexed, primer/tag-trimmed FASTA inputs, then run the filter command with dataset-optimized parameters (bat: lfn_read_count_cutoff 60, pcr_error_var_prop 0.2, lfn_sample_replicate_cutoff 0.002, lfn_variant_cutoff 0.006; fish: 10, 0.1, 0.003, 0.006; shark: 40, 0.2, 0.001, 0.016, with FilterCodonStop and FilterIndel skipped), obtaining these settings via the optimize command applied to negative-control and mock samples. The filter applies FilterLNF, FilterPCRerror, FilterMinReplicateNumber, FilterRenkonen, FilterChimera, and (for COI markers) FilterCodonStop and FilterIndel to eliminate low-frequency noise, PCR errors, non-reproducible occurrences, whole low-diversity replicates, chimeras, and pseudogenes. For comparison, run DALU (DADA2 1.12.1 denoising with marker-specific truncation lengths, LULU 0.1.0 curation, occurrence acceptance in ≥2 replicates) and OBIbaR (OBITools3 3.0.1b19 obi uniq/grep/clean, metabaR 1.0.0 filters), assigning taxonomy for all pipelines with VTAM's taxassign (COInr COI database for COI; RDP Classifier 16S trainsetNo18 for 16S). Sweep a minimum read count cut-off over 0, 10, 40, and 60 (mirroring VTAM's suggested 10 for fish, 40 for shark, 60 for bat), classify occurrences as true/false positives using the count_false_occ option of add_filters.pl against mock and negative controls, and compute sensitivity TP/(TP+FN) and precision TP/(TP+FP), plotting with scripts/plt_sensitivity_precision.py.

## Related tools

- **VTAM** (Primary pipeline: merge, sortreads, filter, optimize, make_known_occurrences, and taxassign commands produce the control-validated ASV table) — https://github.com/aitgon/vtam
- **DADA2** (Denoising engine of the DALU comparison pipeline (v1.12.1) with marker-specific truncation lengths)
- **LULU** (Curation step (v0.1.0) in the DALU pipeline to merge spurious ASVs)
- **OBITools3** (obi uniq/grep/clean denoising (v3.0.1b19) in the OBIbaR comparison pipeline)
- **metabaR** (Filtering (v1.0.0) and replicate pooling in the OBIbaR pipeline)
- **cutadapt** (Dependency installed with VTAM for read processing)
- **vsearch** (Dependency used by VTAM (e.g., chimera handling) installed via bioconda)
- **Snakemake** (Workflow manager orchestrating the vtam_benchmark Snakemake files (01–05)) — https://snakemake.readthedocs.io/en/stable/getting_started/installation.html

## Examples

```
snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- VTAM attains precision 1.00 on the five marine fish samples versus 0.66 (OBIbaR) and 0.19 (DALU), while sensitivity is high (0.95–1) for all pipelines
- Precision of the DALU and OBIbaR pipelines increases monotonically with the minimum read count cut-off (0, 10, 40, 60)
- VTAM shows similar sensitivity but higher precision than DALU and OBIbaR across all three datasets and both markers (COI, 16S)
- control_counts.tsv is produced for each cut-off under out/summary_min_readcount_XX/ by add_filters.pl count_false_occ
- VTAM's per-dataset optimized parameters match the suggested cut-offs (10 for fish, 40 for shark, 60 for bat)

## Limitations

- FilterCodonStop and FilterIndel must be skipped for the shark dataset, so pseudogene/spurious-sequence filtering is not applied for that marker
- Metabarcoders studying metazoa or plants must construct adequate mock communities, as commercial mock samples only exist for bacteria and fungi
- At the highest cut-off (60), OBIbaR's precision was comparable to VTAM for the fish and bat datasets
- Software versions of DADA2, OBITools, VTAM, and metabaR, and optimized parameter values, are not always stated uniformly across sections; DALU and OBIbaR are referenced by name only in parts of the text

## Evidence

- [abstract] VTAM showed similar sensitivity, but a higher precision compared to two other pipelines using three datasets and two different markers (COI, 16S).: "VTAM showed similar sensitivity, but a higher precision compared to two other pipelines using three datasets and two different markers (COI, 16S)."
- [methods] Two filters eliminate non-reproducible occurrences: "Two filters eliminate non-reproducible occurrences (FilterMinReplicateNumber), or whole replicates (FilterRenkonen)"
- [methods] The FilterLNF (Filter Low Frequency Noise) and FilterPCRerror filters eliminate occurrences: "The FilterLNF (Filter Low Frequency Noise) and FilterPCRerror filters eliminate occurrences"
- [methods] These values were based on the suggested cut-offs optimized by VTAM (10 for fish, 40 for shark, 60 for bat): "These values were based on the suggested cut-offs optimized by VTAM (10 for fish, 40 for shark, 60 for bat)"
- [methods] assigned to taxa by the taxassign command using the COInr COI reference database: "assigned to taxa by the taxassign command using the COInr COI reference database"
- [readme] This is the code to reproduce the benchmark analysis of metabarcoding software in this publication: "This is the code to reproduce the benchmark analysis of metabarcoding software in this publication"
