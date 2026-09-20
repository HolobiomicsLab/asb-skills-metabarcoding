---
name: asv-sequence-length-distribution-analysis
description: Use when comparing multiple Tourmaline/DADA2 runs that differ only in
  dada2 truncation lengths (e.g., 100/150/200/250 bp) and you need per-run ASV length
  distributions plus taxonomy-assigned read counts to choose optimal truncation parameters.
when_to_use_negative:
- Data has not been through the Tourmaline qaqc step — no demultiplexed QIIME 2 artifact
  exists yet to run repseqs on
- 'The goal is OTU clustering rather than DADA2 ASV inference — the sweep relies on
  asv_method: dada2pe truncation parameters'
- The analysis targets single-end data — the sweep grid here uses dada2pe with paired
  forward and reverse truncation lengths
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: Tourmaline
  role: Workflow that runs the qaqc, repseqs, and taxonomy steps via tourmaline.sh,
    with distinct run_name per truncation setting for comparison
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Runs the DADA2 denoising and taxonomy classification commands underlying the
    repseqs and taxonomy steps
- name: Snakemake
  role: Workflow management system through which Tourmaline manages commands, inputs,
    and outputs for each step
- name: DADA2
  role: 'Denoising algorithm (asv_method: dada2pe) whose truncation-length parameters
    are swept and whose representative sequences are analyzed'
  repo: https://github.com/benjjneb/dada2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:2d321a7e59cc39045fbfe4dece190278d88bb445cf10275587240d0b0d0c4576
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - DADA2
  - QIIME 2
  - Snakemake
  - Tourmaline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# asv-sequence-length-distribution-analysis

## Summary

Extracts and aggregates representative sequence (ASV) length distributions and Eukaryota read-assignment counts across DADA2 truncation-length sweep runs (100–250 bp) produced by the Tourmaline workflow, enabling selection of truncation lengths that best capture target organism diversity in the 2018 Lake Erie 16S rRNA dataset.

## When to use

Use when comparing multiple Tourmaline/DADA2 runs that differ only in dada2 truncation lengths (e.g., 100/150/200/250 bp) and you need per-run ASV length distributions plus taxonomy-assigned read counts to choose optimal truncation parameters.

## When NOT to use

- Data has not been through the Tourmaline qaqc step — no demultiplexed QIIME 2 artifact exists yet to run repseqs on
- The goal is OTU clustering rather than DADA2 ASV inference — the sweep relies on asv_method: dada2pe truncation parameters
- The analysis targets single-end data — the sweep grid here uses dada2pe with paired forward and reverse truncation lengths

## Inputs

- Raw demultiplexed paired-end FASTQ files (e.g., SRA PRJNA679730 Lake Erie surface water 16S rRNA data)
- Tourmaline qaqc output: demultiplexed QIIME 2 artifact
- config_01_qaqc.yaml with raw_fastq_path, paired_end: true, primer settings
- PR2 eukaryotic reference database for taxonomy assignment
- Per-run sequence-length reports under [run_name]-repseqs/stats/

## Outputs

- truncation_sweep_summary.csv with one row per truncation setting containing the sequence length distribution summary and Eukaryota-assigned read counts
- Per-run representative sequence length distributions from [run_name]-repseqs/stats/
- Per-run Eukaryota read assignment counts

## How to apply

Run the Tourmaline qaqc step once on the demultiplexed paired-end FASTQ data (config_01_qaqc.yaml with raw_fastq_path, paired_end: true, and the appropriate primers). For each truncation setting in the grid {(100,100), (150,150), (200,200), (250,250)}, run the repseqs step with asv_method: dada2pe, setting dada2_trunc_len_f and dada2pe_trunc_len_r accordingly, using a distinct run_name per setting so runs can be compared. For each repseqs run, run the taxonomy step with the PR2 eukaryotic reference database (database_name: PR2) and record the number of reads assigned to Eukaryota. Extract representative sequence length distributions from the sequence-length reports under each [run_name]-repseqs/stats/ directory, then aggregate per-run length distributions and Eukaryota read counts into a single truncation_sweep_summary.csv with one row per truncation setting.

## Related tools

- **Tourmaline** (Workflow that runs the qaqc, repseqs, and taxonomy steps via tourmaline.sh, with distinct run_name per truncation setting for comparison) — https://github.com/aomlomics/tourmaline
- **QIIME 2** (Runs the DADA2 denoising and taxonomy classification commands underlying the repseqs and taxonomy steps)
- **Snakemake** (Workflow management system through which Tourmaline manages commands, inputs, and outputs for each step)
- **DADA2** (Denoising algorithm (asv_method: dada2pe) whose truncation-length parameters are swept and whose representative sequences are analyzed) — https://github.com/benjjneb/dada2

## Examples

```
./tourmaline.sh --step repseqs --configfile config_02_repseqs.yaml --cores N
```

## Evaluation signals

- truncation_sweep_summary.csv contains exactly one row per truncation setting in the grid {(100,100), (150,150), (200,200), (250,250)}
- Each summary row includes both a sequence length distribution summary and Eukaryota-assigned read counts (no missing values)
- Per-run length distributions are present under every [run_name]-repseqs/stats/ directory for all four run_names
- Eukaryota read counts come from the taxonomy step run with database_name: PR2, not another reference database
- Resulting trends reproduce the patterns reported in Supplementary Fig. S4A and S4B of the Tourmaline paper

## Limitations

- Truncation grid evaluated is limited to 100–250 bp in 50-bp increments; other lengths were not tested in this task
- Eukaryota assignment depends on the PR2 reference database; alternative databases (e.g., SILVA 138.1) would give different counts
- Tourmaline is designed for Illumina amplicon data and was demonstrated on a Lake Erie surface-water 16S rRNA dataset; other platforms/targets may need adaptation

## Evidence

- [intro] different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared: "different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared, facilitated by a helper script that makes a new copy of the Tourmaline directory"
- [abstract] Tourmaline utility for parameter optimization: "showing its utility for parameter optimization and the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer"
- [abstract] it uses QIIME 2 to run either the DADA2 or Deblur denoising algorithm: "it uses QIIME 2 to run either the DADA2 or Deblur denoising algorithm"
- [methods] repseq_min_length config keys: "repseq_min_length: 0
repseq_max_length: 0"
- [readme] tourmaline.sh usage: "Instead of interacting with Snakemake rules directly, the main way to run Tourmaline 2 is through the `tourmaline.sh` script."
