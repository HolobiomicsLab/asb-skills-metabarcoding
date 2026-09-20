---
name: batch-pipeline-execution-across-datasets
description: Use when a comparative benchmark requires the identical multi-tool pipeline
  to be executed over many datasets (e.g., 38 16S rRNA ASV tables) under two preprocessing
  regimes, with per-tool outputs collected for aggregation.
when_to_use_negative:
- You need ANCOM on new data — the ANCOM-Code-Archive scripts are archive-only and
  unmaintained; use the ancombc R package instead
- You only need a single tool or a single dataset — running the full 14-tool batch
  adds ~20 min per dataset (e.g., the MALL dataset) of overhead without benefit
- Your datasets have more than two sample groupings — run_all_tools.sh metadata supports
  a maximum of 2 groupings
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: run_all_tools.sh
  role: Key pipeline script that executes all 14 DA tools on a given dataset with
    optional -D rarefaction and -F prevalence filtering
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: dockerdkd/hackathon2021
  role: Docker image pinning correct versions of all tools for reproducible batch
    execution
  repo: https://hub.docker.com/r/dockerdkd/hackathon2021
- name: ANCOM-II
  role: DA tool executed via the Ancom2_Script folder referenced by ANCOM_DIR in config.sh
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: ALDEx2
  role: One of the 14 DA tools run on every dataset by run_all_tools.sh
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: LEfSe
  role: DA tool run inside the 'hackathon' conda environment (name changeable at line
    58 of run_all_tools.sh)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:b55c3ce9fefa38ca7899111eaf4a37c7e38c157cd5451ef7efab5c3c75b335ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - ALDEx2
  - ANCOM-II
  - DESeq2
  - LEfSe
  - Maaslin2
  - corncob
  - dockerdkd/hackathon2021
  - edgeR
  - limma voom
  - metagenomeSeq
  - run_all_tools.sh
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# batch-pipeline-execution-across-datasets

## Summary

Run a fixed battery of 14 microbiome differential abundance tools (ALDEx2, ANCOM-II, DESeq2, edgeR, corncob, metagenomeSeq, MaAsLin2, LEfSe, limma voom, and rarefied/CLR variants) uniformly across all 38 16S rRNA datasets, in both unfiltered and 10% prevalence-filtered variants, via config-driven bash driver scripts.

## When to use

Use when a comparative benchmark requires the identical multi-tool pipeline to be executed over many datasets (e.g., 38 16S rRNA ASV tables) under two preprocessing regimes, with per-tool outputs collected for aggregation.

## When NOT to use

- You need ANCOM on new data — the ANCOM-Code-Archive scripts are archive-only and unmaintained; use the ancombc R package instead
- You only need a single tool or a single dataset — running the full 14-tool batch adds ~20 min per dataset (e.g., the MALL dataset) of overhead without benefit
- Your datasets have more than two sample groupings — run_all_tools.sh metadata supports a maximum of 2 groupings

## Inputs

- Unzipped figshare 16S rRNA Microbiome Datasets (14531724) with 38 ASV tables and folder structure
- Two-column TSV metadata files (sample IDs in column 1, sample groupings in column 2, max 2 groupings)
- Rarefied ASV table for the rarefied tool variants (-R argument)

## Outputs

- Per-tool differential abundance output tables for each of the 38 datasets, unfiltered
- Per-tool differential abundance output tables for each of the 38 datasets, 10% prevalence-filtered (and rarefied at depth 2000)
- Inputs for downstream aggregation into per-tool-per-dataset percentages of significant ASVs (Fig. 1a/b)

## How to apply

Configure config.sh with DATA_DIR pointed at the unzipped figshare 16S rRNA Microbiome Datasets (14531724) Hackathon folder, and set ANCOM_DIR and TOOL_DIR to the repo's Ancom2_Script and Tool_scripts folders. Run run_no_filt_datasets.sh to execute run_all_tools.sh on all 38 unfiltered datasets, then run_0.1_filt_datasets.sh with -D 2000 (rarefaction depth) and -F 0.1 (10% prevalence filter) for the filtered run. The metadata file must be a two-column TSV (sample names, group labels, maximum 2 groupings). Use the docker image dockerdkd/hackathon2021 (R 3.6.3, ALDEx2 1.18.0, DESeq2 1.26.0, edgeR 3.28.1, corncob 0.1.0, metagenomeSeq 1.28.2, Maaslin2 0.99.12, LEfSe in the 'hackathon' conda environment) to guarantee version consistency across the batch. The rationale is that DA results depend on data pre-processing, so every tool must see identical inputs under each regime before per-dataset percentages of significant ASVs are computed.

## Related tools

- **run_all_tools.sh** (Key pipeline script that executes all 14 DA tools on a given dataset with optional -D rarefaction and -F prevalence filtering) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **dockerdkd/hackathon2021** (Docker image pinning correct versions of all tools for reproducible batch execution) — https://hub.docker.com/r/dockerdkd/hackathon2021
- **ANCOM-II** (DA tool executed via the Ancom2_Script folder referenced by ANCOM_DIR in config.sh) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **ALDEx2** (One of the 14 DA tools run on every dataset by run_all_tools.sh) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **LEfSe** (DA tool run inside the 'hackathon' conda environment (name changeable at line 58 of run_all_tools.sh)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Reproduce the MALL (Human-ALL) dataset outputs and diff them against Misc_datafiles/Expected_MALL_output
- Computed limma voom TMMwsp unfiltered mean percentage of significant ASVs must equal 40.5%; ANCOM-II filtered mean must equal 3.8%
- edgeR (mean 32.5%) and LEfSe (mean 27.5%) should emerge as the tools identifying the most significant ASVs on 10% prevalence-filtered data
- Per-tool mean percentages across the 38 datasets must match the deposited expected results before writing significant_asv_percentages.tsv

## Limitations

- The standalone ANCOM functions in FrederickHuangLin/ANCOM-Code-Archive are no longer maintained; users are directed to ancom/ancombc in the ANCOMBC package
- Prevalence filtering at 10% was arbitrarily selected; the optimal cut-off is unknown
- Running a full dataset (e.g., MALL) takes ~20 minutes on a modern single-core CPU, so the 38-dataset batch is compute-intensive

## Evidence

- [readme] The key script is "run_all_tools.sh" for running all tools on a given dataset.: "The key script is "run_all_tools.sh" for running all tools on a given dataset."
- [readme] run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [readme] -F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [methods] A docker image with the correct versions of all the tools is available at https://hub.docker.com/r/dockerdkd/hackathon2021: "A docker image with the correct versions of all the tools is available at https://hub.docker.com/r/dockerdkd/hackathon2021"
- [abstract] these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing: "these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing"
