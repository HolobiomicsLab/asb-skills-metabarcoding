---
name: snakemake-workflow-parameter-sweep
description: Use when you need to compare how pipeline parameters (e.g., dada2 trunc_len_f/trunc_len_r
  values) affect outputs such as representative sequence length distributions or taxonomic
  read assignment, and must run one workflow repeatedly under controlled parameter
  settings.
when_to_use_negative:
- Input is already a denoised feature table or representative sequences — rerunning
  the sweep from raw FASTQ would discard existing ASVs
- The question concerns filtering thresholds (repseq_min_length, repseq_min_abundance,
  etc.) rather than denoising truncation parameters — a single repseqs run with varied
  filter settings is more appropriate
- Single-end data or non-DADA2 methods (Deblur), where paired-end truncation length
  grids (dada2pe) do not apply
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Tourmaline
  role: Containerized Snakemake-managed workflow whose qaqc, repseqs, and taxonomy
    steps are rerun with distinct run_names and configs for each parameter setting
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Runs the DADA2 denoising and taxonomy classification underlying each sweep
    run
- name: Snakemake
  role: Workflow management system that orchestrates each parameterized rerun of the
    pipeline
- name: DADA2
  role: Denoising algorithm whose trunc_len_f/trunc_len_r parameters are the swept
    variables
  repo: github.com/benjjneb/dada2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:2e30860ba05a9aa0594c0bcc61cb5ba83b256dccec6752f8ed546cf810fa3481
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# snakemake-workflow-parameter-sweep

## Summary

A method for systematically sweeping workflow parameters (e.g., DADA2 forward/reverse truncation lengths in 50-bp increments from 100 to 250 bp) by rerunning the Tourmaline/Snakemake amplicon pipeline with a distinct config file and run_name per setting, then aggregating per-run outputs (sequence length distributions and taxonomy-assigned read counts) into a comparison table.

## When to use

Use when you need to compare how pipeline parameters (e.g., dada2 trunc_len_f/trunc_len_r values) affect outputs such as representative sequence length distributions or taxonomic read assignment, and must run one workflow repeatedly under controlled parameter settings.

## When NOT to use

- Input is already a denoised feature table or representative sequences — rerunning the sweep from raw FASTQ would discard existing ASVs
- The question concerns filtering thresholds (repseq_min_length, repseq_min_abundance, etc.) rather than denoising truncation parameters — a single repseqs run with varied filter settings is more appropriate
- Single-end data or non-DADA2 methods (Deblur), where paired-end truncation length grids (dada2pe) do not apply

## Inputs

- Raw paired-end FASTQ files from SRA (e.g., PRJNA679730 Lake Erie 16S rRNA amplicon data)
- Tourmaline configuration YAML (config_01_qaqc.yaml) with raw_fastq_path, paired_end: true, primers, asv_method: dada2pe, dada2_trunc_len_f/dada2pe_trunc_len_r, and a distinct run_name per setting
- Taxonomic reference database (e.g., PR2 for Eukaryota assignment; SILVA 138.1 for 16S)
- Demultiplexed QIIME 2 artifact produced by the qaqc step

## Outputs

- Per-run QIIME 2 representative-sequence artifacts and sequence-length reports under [run_name]-repseqs/stats/
- Per-run taxonomy assignments with Eukaryota read counts
- Aggregated truncation_sweep_summary.csv with one row per truncation setting containing length-distribution summaries and Eukaryota-assigned read counts

## How to apply

Run the Tourmaline qaqc step once (./tourmaline.sh with config_01_qaqc.yaml specifying raw_fastq_path, paired_end: true, and primers) to produce the demultiplexed QIIME 2 artifact from PRJNA679730 FASTQ files. For each parameter setting in the grid — here truncation pairs {(100,100), (150,150), (200,200), (250,250)} — copy the Tourmaline directory with the helper script and set asv_method: dada2pe with dada2_trunc_len_f and dada2pe_trunc_len_r in the config, assigning a distinct run_name so runs are comparable. Run the repseqs step and then the taxonomy step per run (database_name: PR2 for eukaryotic 18S/Eukaryota assignment), recording Eukaryota-assigned reads. Extract representative sequence length distributions from the sequence-length reports under each [run_name]-repseqs/stats/ directory. Aggregate per-run length distributions and Eukaryota read counts into a single CSV (truncation_sweep_summary.csv) with one row per setting, and judge success by reproduction of the trends in Supplementary Fig. S4A/S4B and selection of truncation lengths that capture maximal target-organism diversity.

## Related tools

- **Tourmaline** (Containerized Snakemake-managed workflow whose qaqc, repseqs, and taxonomy steps are rerun with distinct run_names and configs for each parameter setting) — github.com/aomlomics/tourmaline
- **QIIME 2** (Runs the DADA2 denoising and taxonomy classification underlying each sweep run)
- **Snakemake** (Workflow management system that orchestrates each parameterized rerun of the pipeline)
- **DADA2** (Denoising algorithm whose trunc_len_f/trunc_len_r parameters are the swept variables) — github.com/benjjneb/dada2

## Examples

```
./tourmaline.sh --step repseqs --configfile config_02_repseqs.yaml --cores 8
```

## Evaluation signals

- truncation_sweep_summary.csv contains exactly one row per truncation setting, each with both a sequence length distribution summary and Eukaryota-assigned read counts
- Trends across the sweep reproduce the patterns reported in Supplementary Fig. S4A and S4B
- Each run is written to a distinct [run_name]-repseqs/stats/ directory so no two settings overwrite outputs
- Per-run QIIME 2 provenance (built-in tracking) records the specific dada2_trunc_len_f / dada2pe_trunc_len_r values used
- The chosen truncation set captures a large diversity of target organisms, evidenced by maximal Eukaryota read assignment without excessive sequence loss

## Limitations

- Only the tested grid (100–250 bp in 50-bp increments) was validated; other truncation ranges may behave differently
- Runs must use a distinct run_name (via the helper script copying the Tourmaline directory) or comparison across settings is compromised
- Tourmaline 2 differs from V1 (separate V1 branch); results and config semantics may not be interchangeable between versions
- No changelog found, so version-dependent behavior changes cannot be traced

## Evidence

- [intro] different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared, facilitated by a helper script that makes a new copy of the Tourmaline directory: "different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared, facilitated by a helper script that makes a new copy of the Tourmaline directory"
- [readme] ./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
- [abstract] Tourmaline was run on a Lake Erie 16S rRNA amplicon dataset demonstrating utility for parameter optimization and interactive visualization: [evidence span withheld — anchor status: not_found]
