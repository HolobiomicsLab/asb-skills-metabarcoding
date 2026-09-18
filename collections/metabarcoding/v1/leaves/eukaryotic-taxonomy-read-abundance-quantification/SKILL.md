---
name: eukaryotic-taxonomy-read-abundance-quantification
description: Use when representative sequences (e.g. from a DADA2 paired-end run on
  16S rRNA data) need taxonomic assignment against PR2 and you must record Eukaryota-assigned
  read counts to judge which parameters highest-ranked capture target eukaryotic diversity.
when_to_use_negative:
- Input is already a feature/taxonomy table or a downstream diversity result — taxonomy
  assignment requires representative sequences from a repseqs run
- The analysis targets bacteria/archaea reads only and no eukaryotic (Eukaryota) abundance
  metric is needed — PR2 is a eukaryotic reference database
- Single-end Deblur denoising is required (Deblur is single-end), while this skill's
  workflow uses DADA2 paired-end truncation settings
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: Tourmaline
  role: Runs the qaqc, repseqs, and taxonomy steps; distinct run_name directories
    make DADA2 truncation-length runs comparable
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Executes the DADA2 denoising and taxonomic classification of representative
    sequences wrapped by Tourmaline
- name: Snakemake
  role: Workflow management system that Tourmaline uses to manage the step commands,
    inputs, and outputs
- name: DADA2
  role: Denoising algorithm producing representative sequences whose truncation lengths
    (100–250 bp) are swept before Eukaryota counting
  repo: https://github.com/benjjneb/dada2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:1b4ada8129bdae95ae6989e3721654270a1297e8a4779bf9808fa6fca5bc3fd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - DADA2
  - QIIME 2
  - Snakemake
  - Tourmaline
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# eukaryotic-taxonomy-read-abundance-quantification

## Summary

Quantify how many reads in an amplicon run are assigned to Eukaryota by running the Tourmaline taxonomy step on DADA2 representative sequences against a eukaryotic reference database (database_name: PR2), enabling comparison of eukaryotic read capture across parameter settings such as DADA2 truncation lengths.

## When to use

Use when representative sequences (e.g. from a DADA2 paired-end run on 16S rRNA data) need taxonomic assignment against PR2 and you must record Eukaryota-assigned read counts to judge which parameters best capture target eukaryotic diversity.

## When NOT to use

- Input is already a feature/taxonomy table or a downstream diversity result — taxonomy assignment requires representative sequences from a repseqs run
- The analysis targets bacteria/archaea reads only and no eukaryotic (Eukaryota) abundance metric is needed — PR2 is a eukaryotic reference database
- Single-end Deblur denoising is required (Deblur is single-end), while this skill's workflow uses DADA2 paired-end truncation settings

## Inputs

- QIIME 2 representative-sequences artifact from a Tourmaline repseqs step (asv_method: dada2pe, paired-end)
- Tourmaline config YAML with database_name set to PR2 and a distinct run_name per parameter setting
- Raw PRJNA679730 FASTQ files (2018 Lake Erie surface-water 16S rRNA amplicon data) processed via the qaqc step

## Outputs

- Taxonomy assignment for representative sequences (QIIME 2 taxonomy artifact/visualization)
- Per-run Eukaryota-assigned read counts
- Representative sequence length distributions from [run_name]-repseqs/stats/ sequence-length reports
- Aggregated summary table (e.g. truncation_sweep_summary.csv) with one row per truncation setting

## How to apply

After the Tourmaline repseqs step has produced representative sequences with asv_method: dada2pe, run the taxonomy step with a eukaryotic reference database by setting database_name: PR2 in the config YAML. For parameter sweeps, use a distinct run_name per setting (e.g. one per (dada2_trunc_len_f, dada2pe_trunc_len_r) pair such as (100,100), (150,150), (200,200), (250,250)) so runs are directly comparable in Tourmaline's directory structure. From each run, record the number of reads assigned to Eukaryota and extract the representative sequence length distributions from the sequence-length reports under each [run_name]-repseqs/stats/ directory. Optionally apply representative sequence filtering (taxonomy, length, abundance, prevalence, or ID; parameters repseq_min_length, repseq_max_length, repseq_min_abundance, repseq_min_prevalence default to 0) before or during taxonomy assignment. Aggregate per-run length distributions and Eukaryota read counts into a single table (e.g. truncation_sweep_summary.csv) with one row per parameter setting, then select truncation lengths capturing the largest diversity of target organisms.

## Related tools

- **Tourmaline** (Runs the qaqc, repseqs, and taxonomy steps; distinct run_name directories make DADA2 truncation-length runs comparable) — https://github.com/aomlomics/tourmaline
- **QIIME 2** (Executes the DADA2 denoising and taxonomic classification of representative sequences wrapped by Tourmaline)
- **Snakemake** (Workflow management system that Tourmaline uses to manage the step commands, inputs, and outputs)
- **DADA2** (Denoising algorithm producing representative sequences whose truncation lengths (100–250 bp) are swept before Eukaryota counting) — https://github.com/benjjneb/dada2

## Examples

```
conda activate snakemake-tour2; ./tourmaline.sh --step taxonomy --configfile config_01_qaqc.yaml --cores 8
```

## Evaluation signals

- truncation_sweep_summary.csv contains exactly one row per truncation setting with both the sequence length distribution summary and Eukaryota-assigned read counts
- Each truncation grid setting {(100,100), (150,150), (200,200), (250,250)} ran with a distinct run_name so per-run stats directories ([run_name]-repseqs/stats/) all exist
- Resulting trends reproduce the patterns reported in Supplementary Fig. S4A and S4B
- Per-run length distributions are extractable from the sequence-length reports and aggregate cleanly across runs
- Chosen truncation lengths capture a large diversity of target organisms in the 2018 Lake Erie 16S rRNA dataset

## Limitations

- Taxonomic results depend on the configured reference database (database_name must match the controlled vocabulary, e.g. PR2 or silva-138_1)
- Representative-sequence filter parameters (repseq_min_length, repseq_max_length, repseq_min_abundance, repseq_min_prevalence) default to 0 and must be set deliberately or no filtering occurs
- The Deblur alternative is single-end, so paired-end truncation sweeps with dada2pe are not directly comparable to Deblur runs
- Tourmaline 2 (vs. V1) changes how steps are invoked; the V1 branch must be checked out for legacy behavior

## Evidence

- [abstract] assigns taxonomy to the resulting representative sequences: "assigns taxonomy to the resulting representative sequences"
- [abstract] representative sequence filtering options: "representative sequence filtering (taxonomy, length, abundance, prevalence, or ID)"
- [intro] Truncation runs are comparable via run directories: "different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared"
- [readme] Tourmaline 2 usage via tourmaline.sh: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
