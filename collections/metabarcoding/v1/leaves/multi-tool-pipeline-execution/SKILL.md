---
name: multi-tool-pipeline-execution
description: Use when you need to apply several DA tools to the same ASV count table
  + two-column metadata grouping under identical filtering/rarefaction settings, and
  compare per-tool significant ASV calls.
when_to_use_negative:
- You want to use the maintained ANCOMBC R package (ancom/ancombc functions) instead
  of the archived, unmaintained standalone ANCOM scripts in FrederickHuangLin/ANCOM-Code-Archive
- Your metadata has more than 2 sample groupings — the two-column metadata file allows
  maximum 2 groupings
- You need results for a single tool only; running run_all_tools.sh incurs the overhead
  of all 14 tools (~20 min per dataset on a modern single-core CPU)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: ALDEx2
  role: DA tool invoked by run_all_tools.sh; version 1.18.0 pinned in the pipeline
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ANCOM
  role: Standalone ANCOM scripts (Ancom2_Script folder) called via ANCOM_DIR; archived
    code, no longer maintained
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: DESeq2
  role: DA tool invoked by run_all_tools.sh; negative-binomial based, version 1.26.0
    pinned
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: edgeR
  role: DA tool invoked by run_all_tools.sh; version 3.28.1 pinned
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: LEfSe
  role: DA tool run inside the required 'hackathon' conda environment (name changeable
    at line 58 of run_all_tools.sh)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: corncob
  role: DA tool invoked by run_all_tools.sh; version 0.1.0 pinned
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: metagenomeSeq
  role: DA tool invoked by run_all_tools.sh; version 1.28.2 pinned
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: limma voom
  role: DA tool invoked by run_all_tools.sh as part of the 14-tool comparison
- name: dockerdkd/hackathon2021
  role: Docker image containing correct versions of all tools for reproducible pipeline
    execution
  repo: https://hub.docker.com/r/dockerdkd/hackathon2021
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:ba021c558a00df3eac3f22cbff097c2ff3398801faf6fe6067e6b99d67aeebbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - ALDEx2
  - ANCOM
  - ANCOM-II
  - DESeq2
  - LEfSe
  - corncob
  - dockerdkd/hackathon2021
  - edgeR
  - limma voom
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# multi-tool-pipeline-execution

## Summary

Runs 14 differential abundance tools (ALDEx2, ANCOM, ANCOM-II, DESeq2, edgeR, LEfSe, corncob, metagenomeSeq, limma voom, etc.) uniformly across 16S rRNA ASV tables using the Dockerized bash pipeline from nearinj/Comparison_of_DA_microbiome_methods, so tool outputs are directly comparable across datasets and pre-processing conditions.

## When to use

Use when you need to apply several DA tools to the same ASV count table + two-column metadata grouping under identical filtering/rarefaction settings, and compare per-tool significant ASV calls.

## When NOT to use

- You want to use the maintained ANCOMBC R package (ancom/ancombc functions) instead of the archived, unmaintained standalone ANCOM scripts in FrederickHuangLin/ANCOM-Code-Archive
- Your metadata has more than 2 sample groupings — the two-column metadata file allows maximum 2 groupings
- You need results for a single tool only; running run_all_tools.sh incurs the overhead of all 14 tools (~20 min per dataset on a modern single-core CPU)

## Inputs

- ASV count table (tsv, features x samples) from the figshare 16S rRNA Microbiome Datasets deposit
- Two-column metadata tsv (column 1: sample names; column 2: sample grouping, maximum 2 groupings)
- config.sh with DATA_DIR, ANCOM_DIR, TOOL_DIR PATHs
- Rarefied ASV table (for -R unfiltered runs) or rarefaction depth and prevalence filter values (-D, -F)

## Outputs

- Per-dataset, per-tool significant ASV result tables in the specified OUTPUT_FOLDER
- False-positive analysis result tables (percent significant ASVs per tool/replicate) from run_all_FP_datasets_*.sh
- Aggregate results matching Fig. 4 (filtered and unfiltered conditions)

## How to apply

Download the raw 16S rRNA ASV count tables and two-column metadata (sample name + grouping, max 2 groups) from the figshare deposit (14531724) and point config.sh's DATA_DIR, ANCOM_DIR, and TOOL_DIR parameters at the unzipped Hackathon data and the repo's Ancom2_Script/Tool_scripts folders. Run the key script run_all_tools.sh per dataset, passing -A (ASV table tsv), -G (metadata tsv), -O (output folder), and either -R (rarefied ASV table) for unfiltered runs or -D 2000 -F 0.1 (rarefaction depth 2000, 10% prevalence filter) for filtered runs. Batch execution uses run_0.1_filt_datasets.sh (38 filtered datasets), run_no_filt_datasets.sh (38 unfiltered), run_all_FP_datasets_filt.sh / run_all_FP_datasets_unfilt.sh (false-positive analyses), and run_FP_unfilt_EXT.sh (90 extra unfiltered FP replicates, all tools except ANCOM, ALDEx2, corncob). Execute tools inside the docker image dockerdkd/hackathon2021 (R 3.6.3 with pinned package versions such as ALDEx2 1.18.0, edgeR 3.28.1, DESeq2 1.26.0, LEfSe in a conda env named 'hackathon') to guarantee version consistency. Validate by comparing your output to the shipped Expected_MALL_output for the Human-ALL (MALL) dataset (~20 min on a single core).

## Related tools

- **ALDEx2** (DA tool invoked by run_all_tools.sh; version 1.18.0 pinned in the pipeline) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ANCOM** (Standalone ANCOM scripts (Ancom2_Script folder) called via ANCOM_DIR; archived code, no longer maintained) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **DESeq2** (DA tool invoked by run_all_tools.sh; negative-binomial based, version 1.26.0 pinned) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **edgeR** (DA tool invoked by run_all_tools.sh; version 3.28.1 pinned) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **LEfSe** (DA tool run inside the required 'hackathon' conda environment (name changeable at line 58 of run_all_tools.sh)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **corncob** (DA tool invoked by run_all_tools.sh; version 0.1.0 pinned) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **metagenomeSeq** (DA tool invoked by run_all_tools.sh; version 1.28.2 pinned) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **limma voom** (DA tool invoked by run_all_tools.sh as part of the 14-tool comparison)
- **dockerdkd/hackathon2021** (Docker image containing correct versions of all tools for reproducible pipeline execution) — https://hub.docker.com/r/dockerdkd/hackathon2021

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Run the Human-ALL (MALL) dataset and confirm output matches the shipped Expected_MALL_output in Misc_datafiles (expect ~20 minutes on a modern single-core CPU)
- Verify the filtered run used exactly -D 2000 and -F 0.1 (2000-read rarefaction depth, 10% prevalence filter) as in the manuscript
- Confirm all 38 datasets complete for both run_0.1_filt_datasets.sh and run_no_filt_datasets.sh with per-tool outputs in each OUTPUT_FOLDER
- For the false-positive extension, check run_FP_unfilt_EXT.sh covers 90 replicates and excludes ANCOM, ALDEx2, and corncob
- Check resulting per-tool significance percentages fall in manuscript ranges (e.g., edgeR on filtered data mean 0.69–27.9% significant ASVs) and flag outliers

## Limitations

- The archived ANCOM scripts in FrederickHuangLin/ANCOM-Code-Archive are no longer maintained; ANCOMBC's ancom/ancombc functions are recommended instead
- The randomized approach for estimating FDR is not a perfect representation of real data, since real groupings may contain systematic abundance differences; few significant ASVs under randomization is not proof of low practical FDR
- The 10% prevalence-filter cut-off was arbitrarily selected and the optimal threshold is unknown
- Metadata must be a two-column tsv with at most 2 groupings; the LEfSe conda environment must be named 'hackathon' (or line 58 of run_all_tools.sh edited)

## Evidence

- [readme] The key script is "run_all_tools.sh" for running all tools on a given dataset.: "The key script is "run_all_tools.sh" for running all tools on a given dataset."
- [readme] Filtered run command: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [readme] Metadata format constraint: "the metadatafile should be a two column tsv table with the first column having the sample names and the second column having the sample groupings.(maximum 2 groupings)"
- [readme] Batch FP scripts: "run_all_FP_datasets_filt.sh --> runs the false postive analysis for filtered datasets"
- [readme] FP extension exclusions: "run_FP_unfilt_EXT.sh --> runs an additional 90 replicates for the unfilt FP analysis on all tools except for ANCOM, ALDEx2, and corncob."
- [methods] Docker image availability: "A docker image with the correct versions of all the tools is available at https://hub.docker.com/r/dockerdkd/hackathon2021"
- [methods] Figshare data deposit: "available in figshare at the following link: https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724"
- [readme] Expected output for validation: "We have included the expected results from running the Human-ALL dataset (MALL dataset). This can be found in the Misc_datafiles/Expected_MALL_output folder."
