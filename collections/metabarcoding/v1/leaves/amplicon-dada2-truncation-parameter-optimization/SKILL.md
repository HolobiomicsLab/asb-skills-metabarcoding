---
name: amplicon-dada2-truncation-parameter-optimization
description: 'Use when processing paired-end amplicon FASTQ (e.g. 16S rRNA) with DADA2
  and the truncation lengths are undetermined or dataset-specific: quality profiles,
  the need to maximize read retention of target taxa (e.'
when_to_use_negative:
- The data are already denoised (an ASV/feature table exists) — truncation is a DADA2
  denoising-time parameter and cannot be applied post hoc
- The workflow uses a single-end denoising method such as Deblur, where the paired-end
  dada2pe trunc_len_f/trunc_len_r grid does not apply
- The goal is taxonomy classification or diversity analysis of an already-trimmed
  artifact rather than optimizing truncation parameters
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Tourmaline
  role: Snakemake-based workflow that runs the qaqc, repseqs, and taxonomy steps with
    per-run configuration files and distinct run_names so truncation settings can
    be compared
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Runs the DADA2 denoising plugin (dada2pe) and the taxonomy classifier; produces
    the demultiplexed artifact and representative sequence outputs
  repo: https://qiime2.org
- name: DADA2
  role: Denoising algorithm whose dada2_trunc_len_f and dada2pe_trunc_len_r parameters
    are the sweep variables
  repo: github.com/benjjneb/dada2
- name: Snakemake
  role: Workflow management system underlying Tourmaline's step execution (Snakefiles
    per step)
  repo: https://snakemake.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:2b948d27b3e78ebe21b808d4f48696300e9c16eb7599103fca3cd21252f8ec16
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - QIIME 2
  - Snakemake
  - Tourmaline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# amplicon-dada2-truncation-parameter-optimization

## Summary

A parameter-sweep skill for choosing DADA2 forward/reverse truncation lengths in an amplicon workflow (Tourmaline/QIIME 2) by running the repseqs step across a grid of truncation settings (e.g. 100–250 bp in 50-bp increments), each under a distinct run_name, and comparing representative-sequence length distributions plus Eukaryota-assigned read counts to identify the setting that best captures target organism diversity.

## When to use

Use when processing paired-end amplicon FASTQ (e.g. 16S rRNA) with DADA2 and the truncation lengths are undetermined or dataset-specific: quality profiles, the need to maximize read retention of target taxa (e.g. Eukaryota), or reproducibility of a reported truncation sweep (Supplementary Fig. S4A/S4B) trigger this skill.

## When NOT to use

- The data are already denoised (an ASV/feature table exists) — truncation is a DADA2 denoising-time parameter and cannot be applied post hoc
- The workflow uses a single-end denoising method such as Deblur, where the paired-end dada2pe trunc_len_f/trunc_len_r grid does not apply
- The goal is taxonomy classification or diversity analysis of an already-trimmed artifact rather than optimizing truncation parameters

## Inputs

- Raw paired-end FASTQ files downloaded from SRA (e.g. PRJNA679730, 2018 Lake Erie 16S rRNA surface water dataset)
- Tourmaline qaqc configuration file (config_01_qaqc.yaml) with raw_fastq_path, paired_end: true, and forward/reverse primer sequences
- Eukaryotic reference taxonomy database (PR2) for the taxonomy step
- Demultiplexed QIIME 2 artifact produced by the qaqc step

## Outputs

- Per-run representative sequence (ASV) QIIME 2 artifacts for each truncation setting
- Representative sequence length distributions from sequence-length reports under [run_name]-repseqs/stats/
- Taxonomy tables with Eukaryota-assigned read counts (PR2 classifier)
- Aggregated truncation_sweep_summary.csv with one row per truncation setting containing length distribution summary and Eukaryota read counts

## How to apply

First run the Tourmaline qaqc step once to obtain the demultiplexed QIIME 2 artifact: configure config_01_qaqc.yaml with raw_fastq_path, paired_end: true, and the assay primers, then run ./tourmaline.sh -s qaqc -c config_01_qaqc.yaml. Then, for each truncation setting in the grid {(100,100), (150,150), (200,200), (250,250)}, run the repseqs step with asv_method: dada2pe, setting dada2_trunc_len_f and dada2pe_trunc_len_r to the paired values, and assign a distinct run_name per setting so runs remain comparable (Tourmaline's helper script creates a new copy of the Tourmaline directory for this purpose). For each repseqs run, run the taxonomy step against a eukaryotic reference database (database_name: PR2) and record the number of reads assigned to Eukaryota. Extract representative sequence length distributions from the sequence-length reports under each [run_name]-repseqs/stats/ directory. Finally, aggregate per-run length distributions and Eukaryota read counts into truncation_sweep_summary.csv with one row per truncation setting; the rationale is that truncation length trades off read retention against read-merging/sequence quality, and the sweep empirically identifies the setting capturing the largest diversity of target organisms.

## Related tools

- **Tourmaline** (Snakemake-based workflow that runs the qaqc, repseqs, and taxonomy steps with per-run configuration files and distinct run_names so truncation settings can be compared) — github.com/aomlomics/tourmaline
- **QIIME 2** (Runs the DADA2 denoising plugin (dada2pe) and the taxonomy classifier; produces the demultiplexed artifact and representative sequence outputs) — https://qiime2.org
- **DADA2** (Denoising algorithm whose dada2_trunc_len_f and dada2pe_trunc_len_r parameters are the sweep variables) — github.com/benjjneb/dada2
- **Snakemake** (Workflow management system underlying Tourmaline's step execution (Snakefiles per step)) — https://snakemake.readthedocs.io/en/stable/

## Examples

```
./tourmaline.sh --step repseqs --configfile config_02_repseqs.yaml --cores 8
```

## Evaluation signals

- truncation_sweep_summary.csv contains exactly one row per truncation setting in {(100,100), (150,150), (200,200), (250,250)}, each with both a sequence length distribution summary and Eukaryota-assigned read counts
- Each repseqs run used asv_method: dada2pe with the matching dada2_trunc_len_f / dada2pe_trunc_len_r pair and a distinct run_name (separate directories per setting)
- Sequence-length reports exist under every [run_name]-repseqs/stats/ directory and their length distributions reflect the corresponding truncation cutoff
- Eukaryota read counts come from a taxonomy step run with database_name: PR2
- The resulting trends (length distributions vs. truncation setting; Eukaryota read counts vs. truncation setting) reproduce the patterns reported in Supplementary Fig. S4A and S4B

## Limitations

- The sweep is restricted to the 50-bp-increment grid {(100,100), (150,150), (200,200), (250,250)}; finer truncation resolution or asymmetric forward/reverse settings are outside the demonstrated design
- Results are dataset-specific (2018 Lake Erie 16S rRNA data); trends depend on that run's quality profiles and cannot be assumed to transfer to other amplicon datasets or primer sets
- Validation is comparative (relative trends across settings, checked against Supplementary Fig. S4A/S4B), not an absolute correctness check of any single truncation choice
- Comparability across settings requires a distinct run_name/new Tourmaline directory copy per run; reusing one run directory mixes parameters and invalidates the comparison

## Evidence

- [intro] different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared, facilitated by a helper script that makes a new copy of the Tourmaline directory: "different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared, facilitated by a helper script that makes a new copy of the Tourmaline directory"
- [methods] Step 1. Sequence quality assurance and quality control: "Step 1. Sequence quality assurance and quality control"
- [abstract] We ran Tourmaline on a 16S ribosomal RNA amplicon data set from Lake Erie surface water: "We ran Tourmaline on a 16S ribosomal RNA amplicon data set from Lake Erie surface water"
- [readme] ./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
