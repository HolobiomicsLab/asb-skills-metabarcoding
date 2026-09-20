---
name: genus-level-taxonomic-abundance-collapsing
description: Use when preparing ASV tables from multiple 16S rRNA datasets for cross-study
  DA consistency analysis, where significant features must be compared at the genus
  level rather than ASV level.
when_to_use_negative:
- When the analysis targets ASV-level features rather than cross-study genus-level
  consistency — the paper's main comparison of 14 DA tools on 38 datasets operates
  on ASV tables
- When the disease/environment datasets are not appropriate for cross-data consistency
  comparison; the paper cautions that 'not all environments and datasets are appropriate
  for this comparison'
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: run_all_tools.sh
  role: Pipeline script that runs all 14 DA tools on each collapsed genus-level dataset
    with -D 2000 rarefaction and -F 0.1 prevalence filtering
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ALDEx2
  role: One of the DA tools run on the collapsed genus-level tables; showed 1.35-fold
    increased consistency across diarrhea datasets
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ANCOM
  role: ANCOM/ANCOM-II scripts run on the collapsed genus tables; ANCOM_DIR must point
    to the Ancom2_Script folder
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: DESeq2
  role: Negative-binomial DA tool (v1.26.0) run on the collapsed genus tables
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: edgeR
  role: Negative-binomial DA tool (v3.28.1) run on the collapsed genus tables; 1.105-fold
    consistency at p=0.002 in the diarrhea analysis
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: metagenomeSeq
  role: Zero-inflated Gaussian DA tool (v1.28.2) run on the collapsed genus tables
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:8b726a771f3a940dc86331ba6cdae27eeca9b8124cdd7a4c21b4697b3cc911ac
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
  - edgeR
  - limma voom
  - metagenomeSeq
  - run_all_tools.sh
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# genus-level-taxonomic-abundance-collapsing

## Summary

Collapse ASV-level 16S rRNA count tables to genus level before running differential abundance (DA) tools, so that cross-study consistency of significant genera can be compared across datasets of the same phenotype (e.g., the five diarrhea and five obesity datasets).

## When to use

Use when preparing ASV tables from multiple 16S rRNA datasets for cross-study DA consistency analysis, where significant features must be compared at the genus level rather than ASV level.

## When NOT to use

- When the analysis targets ASV-level features rather than cross-study genus-level consistency — the paper's main comparison of 14 DA tools on 38 datasets operates on ASV tables
- When the disease/environment datasets are not appropriate for cross-data consistency comparison; the paper cautions that 'not all environments and datasets are appropriate for this comparison'

## Inputs

- ASV-level 16S rRNA count tables from the figshare '16S rRNA Microbiome Datasets' archive (e.g., GEMS, dia_schneider, Human-C. diff 1, Human-C. diff 2, Human-Inf. and the five obesity datasets)
- Two-column TSV metadata files (sample names in the first column, sample groupings in the second, maximum 2 groupings)
- Taxonomic assignment/lineage information needed to aggregate ASVs to genus

## Outputs

- Genus-level collapsed count tables, one per dataset, formatted for run_all_tools.sh (ASV_TABLE.tsv / rarified table inputs)
- Downstream per-tool significance calls on the collapsed tables, used to count genera significant in common across all five datasets per phenotype

## How to apply

Download the raw 16S rRNA datasets from figshare (https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724), unzip them, and set DATA_DIR, ANCOM_DIR, and TOOL_DIR in config.sh as required by the pipeline. Collapse each ASV table to genus level for the target datasets — the five diarrhea datasets (GEMS, dia_schneider, Human-C. diff 1, Human-C. diff 2, Human-Inf.) and the five obesity datasets. Feed each collapsed genus-level table (with its two-column TSV metadata: sample names in column 1, sample groupings in column 2) into run_all_tools.sh with rarefaction depth -D 2000 and prevalence filtering -F 0.1 (10%), matching the manuscript's filtering settings. The rationale is that collapsing to genus aligns taxon identities across studies so that observed overlap of significant genera can be compared against a permutation null that assumes all genera are equally likely to be significant by chance.

## Related tools

- **run_all_tools.sh** (Pipeline script that runs all 14 DA tools on each collapsed genus-level dataset with -D 2000 rarefaction and -F 0.1 prevalence filtering) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ALDEx2** (One of the DA tools run on the collapsed genus-level tables; showed 1.35-fold increased consistency across diarrhea datasets) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ANCOM** (ANCOM/ANCOM-II scripts run on the collapsed genus tables; ANCOM_DIR must point to the Ancom2_Script folder) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **DESeq2** (Negative-binomial DA tool (v1.26.0) run on the collapsed genus tables) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **edgeR** (Negative-binomial DA tool (v3.28.1) run on the collapsed genus tables; 1.105-fold consistency at p=0.002 in the diarrhea analysis) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **metagenomeSeq** (Zero-inflated Gaussian DA tool (v1.28.2) run on the collapsed genus tables) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A genus_collapsed_table.tsv -G metadata.tsv -O output_folder -D 2000 -F 0.1
```

## Evaluation signals

- The collapsed tables must run cleanly through run_all_tools.sh with -D 2000 and -F 0.1, producing output for all 14 DA tools per dataset
- For each of the 14 tools, the observed count of genera significant in common across all five diarrhea datasets (and separately across the five obesity datasets) must be computable from the collapsed outputs
- Computed fold differences of observed vs. expected overlap must match Table 2 reference values, e.g., ALDEx2 fold diff 1.353 at p<0.001 and edgeR fold diff 1.105 at p=0.002
- The output CSV must contain one fold-difference row per DA tool
- The Docker image dockerdkd/hackathon2021 with R 3.6.3 and pinned packages (ALDEx2 1.18.0, DESeq2 1.26.0, edgeR 3.28.1, metagenomeSeq 1.28.2) should reproduce the same results as a local install

## Limitations

- The permutation null assumes all genera are equally likely to be significant by chance, which is simplistic and invalid due to prevalence differences between genera
- Consistency of significant genera across obesity datasets was no higher than expected by chance for most tools, indicating collapsing alone does not guarantee reproducible signals
- There is no assurance that the intersection of tool outputs is more reliable; tools could be picking up the same noise as significant
- Rarefying to depth -D 2000 reduces statistical power and is criticized for introducing biases and high false positive rates with standard tests

## Evidence

- [other] Collapse each ASV table to genus level for the five diarrhea datasets (GEMS, dia_schneider, Human-C. diff 1, Human-C. diff 2, Human-Inf.) and the five obesity datasets: [evidence span withheld — anchor status: not_found]
- [methods] run_all_tools.sh filtering invocation: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [readme] Prevalance filtering definition: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [discussion] Diarrhea consistency finding: "ANCOM-II performed better than average at identifying the same genera as significantly DA across five diarrhea-related datasets"
- [discussion] Null model caveat: [evidence span withheld — anchor status: not_found]
- [readme] figshare data source: "All the raw tables along with the correct folder structuring is available in figshare at the following link: https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724"
