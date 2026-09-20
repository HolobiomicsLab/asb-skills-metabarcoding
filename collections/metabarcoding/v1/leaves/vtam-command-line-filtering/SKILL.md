---
name: vtam-command-line-filtering
description: Use when demultiplexed FASTQ-derived ASV occurrences (ASV i in sample
  j of replicate k with read count Nijk) and merged fasta files must be stringently
  filtered for PCR errors, low-frequency noise, chimeras, pseudogenes, and non-reproducible
  occurrences using control-aware thresholds.
when_to_use_negative:
- Input is raw, undemultiplexed FASTQ that has not yet been processed by VTAM's merge
  and sortreads commands — the filter step expects their output fasta files as input
- The goal is denoising/clustering of reads into ASVs (Swarm, Unoise, Deblur, DADA2)
  rather than validation filtering of already-called ASVs — VTAM's filter operates
  downstream of variant generation
- The dataset has no negative controls, mock samples, or replicates, so control-based
  threshold optimization (optimize/make_known_occurrences) cannot be meaningfully
  parameterized
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: VTAM
  role: Command-line metabarcoding package whose 'filter' command implements the variant-based
    filtering (FilterLNF, FilterPCRerror, FilterChimera, FilterMinReplicateNumber,
    FilterRenkonen, FilterCodonStop, FilterIndel) and 'optimize' determines filtering
    thresholds
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM (installed via python3 -m pip install vtam
    in a conda python=3.9 environment)
  repo: https://github.com/aitgon/vtam
- name: WopMars
  role: Database-driven workflow manager framework underlying VTAM command execution
  repo: github.com/aitgon/wopmars
- name: vsearch
  role: Conda-installed dependency of VTAM used by its sequence-processing commands
- name: cutadapt
  role: Pip-installed dependency of VTAM for read trimming in the upstream steps feeding
    the filter
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:f5907a29aa9fcb2134187cd68cd5217522a5e7c5a7604d9e2d5beac726a3cc9b
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
  - Python
  - VTAM
  - WopMars
  - cutadapt
  - vsearch
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# vtam-command-line-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the VTAM command-line 'filter' step, a variant-based filtering procedure that applies dedicated filters (FilterLNF, FilterPCRerror, FilterChimera, FilterMinReplicateNumber, FilterRenkonen, FilterCodonStop, FilterIndel) to ASV occurrence tables and merged fasta files, explicitly using negative controls, mock samples, and replicates to minimize false negatives and false positives.

## When to use

Use when demultiplexed FASTQ-derived ASV occurrences (ASV i in sample j of replicate k with read count Nijk) and merged fasta files must be stringently filtered for PCR errors, low-frequency noise, chimeras, pseudogenes, and non-reproducible occurrences using control-aware thresholds.

## When NOT to use

- Input is raw, undemultiplexed FASTQ that has not yet been processed by VTAM's merge and sortreads commands — the filter step expects their output fasta files as input
- The goal is denoising/clustering of reads into ASVs (Swarm, Unoise, Deblur, DADA2) rather than validation filtering of already-called ASVs — VTAM's filter operates downstream of variant generation
- The dataset has no negative controls, mock samples, or replicates, so control-based threshold optimization (optimize/make_known_occurrences) cannot be meaningfully parameterized

## Inputs

- Demultiplexed, tag- and primer-trimmed FASTQ amplicon reads (e.g. fish, bat, or shark datasets from Dryad/OSF)
- Merged fasta sequences produced by VTAM merge/sortreads steps (input to the filter step)
- ASV occurrence table (ASV i in sample j of replicate k with read count Nijk)
- VTAM filter parameter values (lfn_sample_replicate_cutoff, pcr_error_var_prop, lfn_variant_cutoff, lfn_read_count_cutoff)

## Outputs

- Filtered occurrence table with erroneous, chimeric, pseudogene, and non-reproducible occurrences removed (exportable as TSV)
- Output fasta files of validated ASVs (input for downstream make_known_occurrences / taxassign)
- Chimera-flag list / list of removed ASV IDs from FilterChimera

## How to apply

First run VTAM's 'merge' and 'sortreads' commands on the demultiplexed FASTQ data to produce the merged fasta files that serve as input to the filter step. Then run the VTAM 'filter' command (implemented in Python within the VTAM/WopMars framework), which applies FilterLNF and FilterPCRerror to eliminate low-frequency noise and occurrences where an ASV closely matches a much more abundant ASV, FilterChimera to remove chimeric ASVs by comparison against more abundant ASVs, FilterMinReplicateNumber and FilterRenkonen to eliminate non-reproducible occurrences or whole replicates, and FilterCodonStop and FilterIndel to discard pseudogenes and spurious sequences. Thresholds such as lfn_sample_replicate_cutoff, pcr_error_var_prop, lfn_variant_cutoff, and lfn_read_count_cutoff can be optimized with the 'optimize' command (e.g. bat dataset optimum: lfn_sample_replicate_cutoff 0.002, pcr_error_var_prop 0.2, lfn_variant_cutoff 0.006, lfn_read_count_cutoff 60) and applied to real samples based on clean control samples. Validation can be done with 'make_known_occurrences' prepared from the filter output, and the resulting filtered occurrence table can be exported (e.g. as TSV) after confirming no flagged chimeric ASV remains.

## Related tools

- **VTAM** (Command-line metabarcoding package whose 'filter' command implements the variant-based filtering (FilterLNF, FilterPCRerror, FilterChimera, FilterMinReplicateNumber, FilterRenkonen, FilterCodonStop, FilterIndel) and 'optimize' determines filtering thresholds) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM (installed via python3 -m pip install vtam in a conda python=3.9 environment)) — https://github.com/aitgon/vtam
- **WopMars** (Database-driven workflow manager framework underlying VTAM command execution) — github.com/aitgon/wopmars
- **vsearch** (Conda-installed dependency of VTAM used by its sequence-processing commands)
- **cutadapt** (Pip-installed dependency of VTAM for read trimming in the upstream steps feeding the filter)

## Examples

```
snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- No ASV flagged as a chimera by FilterChimera remains in the filtered occurrence table
- The set of removed ASV IDs exactly matches the chimera-flag list extracted from the filter output
- Filter step was run on the output fasta files of VTAM merge/sortreads (i.e. inputs are merged, demultiplexed, tag- and primer-trimmed reads)
- Negative control and mock samples in the filter output are clean at the chosen thresholds (e.g. lfn_read_count_cutoff 60 for the bat dataset), confirming FN/FP-minimizing parameters
- Occurrences removed by FilterPCRerror each have a closely matching ASV with a significantly higher read count, and non-reproducible occurrences fail FilterMinReplicateNumber

## Limitations

- Denoising/clustering algorithms alone cannot account for inter-sample contamination, tag-jump, and chimeras, so VTAM filtering must be combined with such tools rather than replacing them
- Commercial mock samples are only available for some taxonomic groups (bacteria and fungi); metabarcoders of metazoa or plants must construct adequate mock samples for threshold optimization
- Recommended read-count cut-offs are dataset-specific (10 for fish, 40 for shark, 60 for bat) and must be re-optimized per dataset using controls

## Evidence

- [readme] VTAM processes FASTQ data and produces a table of ASVs assigned to taxonomic groups: [evidence span withheld — anchor status: external_doc]
