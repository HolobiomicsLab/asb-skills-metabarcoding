---
name: significant-feature-tabulation-per-tool
description: Use when you have DA tool output tables for multiple 16S rRNA datasets
  and need to quantify and compare how many features each method calls significant,
  e.g. to reproduce per-tool significance-rate comparisons across studies.
when_to_use_negative:
- 'You need to judge which tool''s calls are biologically correct rather than how
  many features each calls — the paper states ''for the majority of this work we don''t
  necessarily know the right answer: we can mainly just say which tools are more similar
  to others'''
- Your goal is estimating false positive rates or cross-study consistency of specific
  genera — use the separate FP analysis scripts (run_all_FP_datasets_filt.sh) or consistency
  analysis instead
- Your data has more than two sample groupings — the metadata file must have 'maximum
  2 groupings'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: run_all_tools.sh
  role: Key pipeline script that drives all 14 DA tools on a given dataset with options
    -A (ASV table), -G (metadata), -O (output), -D (rarefaction depth 2000), -F (prevalence
    filter 0.1)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ALDEx2
  role: One of the 14 DA tools applied to each dataset; among the most stringent/most
    consistent methods
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ANCOM-II
  role: Most stringent DA method on 10% prevalence-filtered data (mean 3.8% significant
    ASVs, SD 5.9%); run via the Ancom2_Script folder (ANCOM_DIR)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: DESeq2
  role: Negative-binomial DA tool applied to each dataset
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: edgeR
  role: Negative-binomial DA tool; identified the most significant ASVs on filtered
    data (mean 32.5%)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: LEfSe
  role: DA tool run in the 'hackathon' conda environment; second-highest significance
    rate on filtered data (mean 27.5%)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: corncob
  role: Beta-binomial DA tool applied to each dataset
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: metagenomeSeq
  role: Zero-inflated Gaussian DA tool applied to each dataset
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: Maaslin2
  role: Multivariable DA tool (v0.99.12) applied to each dataset
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: limma voom
  role: DA tool; limma voom TMMwsp unfiltered mean 40.5% significant ASVs used as
    a validation anchor
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:7b94475dcb12ce4071ed7da982d7387e38bc28619f637dac83938bf49ee2bb9e
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
  - edgeR
  - limma voom
  - metagenomeSeq
  - run_all_tools.sh
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# significant-feature-tabulation-per-tool

## Summary

Computes the percentage of significant ASVs identified by each of 14 differential abundance (DA) tools (e.g. ALDEx2, ANCOM-II, DESeq2, edgeR, LEfSe, corncob, metagenomeSeq, MaAsLin2, limma voom) per 16S rRNA dataset, both unfiltered and after 10% prevalence filtering, to reproduce the per-tool-per-dataset table behind Fig. 1a/b of Nearing et al. 2022.

## When to use

Use when you have DA tool output tables for multiple 16S rRNA datasets and need to quantify and compare how many features each method calls significant, e.g. to reproduce per-tool significance-rate comparisons across studies.

## When NOT to use

- You need to judge which tool's calls are biologically correct rather than how many features each calls — the paper states 'for the majority of this work we don't necessarily know the right answer: we can mainly just say which tools are more similar to others'
- Your goal is estimating false positive rates or cross-study consistency of specific genera — use the separate FP analysis scripts (run_all_FP_datasets_filt.sh) or consistency analysis instead
- Your data has more than two sample groupings — the metadata file must have 'maximum 2 groupings'

## Inputs

- ASV count tables (ASV_TABLE.tsv) and two-column metadata TSVs (sample names and groupings, max 2 groups) for the 38 16S rRNA datasets from figshare 14531724
- Per-tool DA output tables from the 14 methods (ALDEx2 1.18.0, ANCOM-II, corncob 0.1.0, DESeq2 1.26.0, edgeR 3.28.1, LEfSe in the 'hackathon' conda env, MaAsLin2 0.99.12, metagenomeSeq 1.28.2, limma voom, rarefied/CLR test variants)
- config.sh with DATA_DIR, ANCOM_DIR, TOOL_DIR paths

## Outputs

- significant_asv_percentages.tsv: per-tool-per-dataset table of percentage of significant ASVs for both filtered and unfiltered variants
- Per-tool mean percentages across datasets (e.g. ANCOM-II filtered mean 3.8%, edgeR mean 32.5%, LEfSe mean 27.5%)

## How to apply

Run the repo pipeline (config.sh pointing DATA_DIR at the figshare 16S rRNA Microbiome Datasets 14531724 and ANCOM_DIR/TOOL_DIR at the repo's Ancom2_Script and Tool_scripts folders) via run_no_filt_datasets.sh on the 38 unfiltered datasets and run_0.1_filt_datasets.sh with -F 0.1 prevalence filtering and -D 2000 rarefaction depth for the rarefied variants. For each tool's output table, count the significant ASVs per dataset and divide by the total number of tested ASVs to get the percentage of significant ASVs per tool per dataset. Aggregate these percentages into a per-tool-per-dataset table written to significant_asv_percentages.tsv, matching Fig. 1a/b. The rationale is that different DA tools identify drastically different numbers of significant ASVs and results depend on pre-processing (10% prevalence filtering, rarefaction), so tabulation must be done per tool, per dataset, per filtering regime.

## Related tools

- **run_all_tools.sh** (Key pipeline script that drives all 14 DA tools on a given dataset with options -A (ASV table), -G (metadata), -O (output), -D (rarefaction depth 2000), -F (prevalence filter 0.1)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ALDEx2** (One of the 14 DA tools applied to each dataset; among the most stringent/most consistent methods) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ANCOM-II** (Most stringent DA method on 10% prevalence-filtered data (mean 3.8% significant ASVs, SD 5.9%); run via the Ancom2_Script folder (ANCOM_DIR)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **DESeq2** (Negative-binomial DA tool applied to each dataset) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **edgeR** (Negative-binomial DA tool; identified the most significant ASVs on filtered data (mean 32.5%)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **LEfSe** (DA tool run in the 'hackathon' conda environment; second-highest significance rate on filtered data (mean 27.5%)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **corncob** (Beta-binomial DA tool applied to each dataset) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **metagenomeSeq** (Zero-inflated Gaussian DA tool applied to each dataset) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **Maaslin2** (Multivariable DA tool (v0.99.12) applied to each dataset) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **limma voom** (DA tool; limma voom TMMwsp unfiltered mean 40.5% significant ASVs used as a validation anchor) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Per-tool mean percentages must reproduce reported values: limma voom TMMwsp unfiltered mean 40.5%, ANCOM-II filtered mean 3.8% (SD 5.9%), edgeR filtered mean 32.5%, LEfSe filtered mean 27.5%
- Individual tool outputs on the Human-ALL (MALL) dataset must match the deposited expected results in Misc_datafiles/Expected_MALL_output
- The aggregated table must match Fig. 1a/b structure: one percentage per tool per dataset, for both 10%-filtered and unfiltered variants
- Each metadata file used must be a two-column TSV (sample names, sample groupings) with a maximum of 2 groupings
- Run time sanity check: the MALL dataset should take ~20 minutes on a modern single-core CPU

## Limitations

- Results depend heavily on data pre-processing: the 10% prevalence filter cut-off was arbitrarily selected and 'more work is needed to estimate an optimal cut-off'
- 'it is unclear why some methods have much higher significance rates than other tools' — high percentages (e.g. edgeR 32.5%) cannot be straightforwardly interpreted as better detection
- For most datasets the 'right answer' is unknown; the tabulation only shows which tools are more similar to others, not which are correct
- The randomized FDR-estimation approach is 'not a perfect representation of real data', so few significant ASVs under it is not proof of low practical FDR
- Standalone ANCOM code in the ANCOM-Code-Archive is unmaintained; users are directed to the ANCOMBC R package

## Evidence

- [results] On 10% prevalence-filtered data, ANCOM-II was the most stringent DA method (mean 3.8% significant ASVs, SD 5.9%), while edgeR (mean 32.5%) and LEfSe (mean 27.5%) identified the most.: [evidence span withheld — anchor status: not_found]
- [readme] README describes the core invocation and filter parameters.: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [readme] README defines the prevalence filter flag.: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [methods] Methods text on the runner scripts for 38 datasets.: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [readme] README on expected outputs for validation.: "We have included the expected results from running the Human-ALL dataset (MALL dataset). This can be found in the Misc_datafiles/Expected_MALL_output folder."
- [abstract] Abstract motivating the tabulation: tools differ drastically.: "these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing"
