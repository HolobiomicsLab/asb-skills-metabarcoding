---
name: rarefaction-and-prevalence-filtering-preprocessing
description: Use when preparing ASV/OTU count tables and metadata for multi-tool DA
  comparison, and read depths differ across samples or rare taxa could inflate tool-specific
  significance calls.
when_to_use_negative:
- When rarefying for other analyses such as computing diversity metrics, which is
  beyond the scope of this work
- When a taxon's prevalence means filtering would bias the test statistic — filtering
  should be independent of the test statistic, and the optimal cut-off is unknown
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3308
tools:
- name: run_all_tools.sh
  role: Pipeline entry point that applies rarefaction (-D) and prevalence filtering
    (-F) before running all 14 DA tools
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ALDEx2
  role: Downstream DA tool whose calls depend on the rarefied, prevalence-filtered
    table
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ANCOM
  role: Downstream DA tool applied to the filtered tables via the ANCOM-Code-Archive
    preprocessing (zero handling) pipeline
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:c63fb74bff5953f2d6cf1f4eecd6088ee1635449ef726c367efe1d3d1d78795f
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# rarefaction-and-prevalence-filtering-preprocessing

## Summary

Standardize 16S rRNA count tables before differential abundance (DA) testing by rarefying to a fixed sequencing depth (-D) and filtering taxa by prevalence (-F), matching the Nearing et al. 2022 cross-tool comparison settings.

## When to use

Use when preparing ASV/OTU count tables and metadata for multi-tool DA comparison, and read depths differ across samples or rare taxa could inflate tool-specific significance calls.

## When NOT to use

- When rarefying for other analyses such as computing diversity metrics, which is beyond the scope of this work
- When a taxon's prevalence means filtering would bias the test statistic — filtering should be independent of the test statistic, and the optimal cut-off is unknown

## Inputs

- ASV/OTU count table in TSV format (features x samples, absolute abundance)
- Two-column metadata TSV (sample names, group labels; maximum 2 groupings)
- Rarefaction depth parameter (manuscript value: 2000)
- Prevalence filter threshold (manuscript value: 0.1, i.e., 10%)

## Outputs

- Rarefied ASV table at the specified depth (-D 2000)
- Prevalence-filtered feature tables per dataset (10% threshold) consumed by run_all_tools.sh
- Per-dataset, per-tool DA output folders produced from the filtered and unfiltered variants

## How to apply

Pass the ASV table, two-column metadata TSV (sample names and groupings), and output folder to run_all_tools.sh with rarefaction depth -D 2000 and prevalence filtering -F 0.1 (a taxon is kept only if present in >=10% of samples), replicating the manuscript's filtered analysis of the 38 16S rRNA datasets. Rationale: rarefaction corrects for differing read depths across samples, and the article recommends prevalence filtering (e.g., at 10%) prior to DA testing to reduce spurious calls on rare features, while noting the 10% cut-off was arbitrarily selected. Run with R 3.6.3 and the pinned package versions (e.g., ALDEx2 1.18.0, DESeq2 1.26.0, edgeR 3.28.1, metagenomeSeq 1.28.2) or via the dockerdkd/hackathon2021 Docker image to keep preprocessing identical across tools. For cross-study consistency analyses (diarrhea/obesity datasets), apply the same -D 2000 -F 0.1 settings to genus-collapsed tables via the run_diarrhea_*.sh and run_obesity_*.sh driver scripts. Optionally run the unfiltered variant (-R RARIFIED_ASV_TABLE.tsv only) to assess sensitivity of DA results to pre-processing, since results are known to depend on it.

## Related tools

- **run_all_tools.sh** (Pipeline entry point that applies rarefaction (-D) and prevalence filtering (-F) before running all 14 DA tools) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ALDEx2** (Downstream DA tool whose calls depend on the rarefied, prevalence-filtered table) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ANCOM** (Downstream DA tool applied to the filtered tables via the ANCOM-Code-Archive preprocessing (zero handling) pipeline) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- The command runs with -D 2000 and -F 0.1 and completes for each of the 38 filtered datasets via run_0.1_filt_datasets.sh
- The output folder contains one result set per DA tool (14 tools), as produced for the expected Human-ALL (MALL) output in Misc_datafiles/Expected_MALL_output
- Prevalence filtering removes features present in fewer than 10% of samples before any tool is invoked
- Downstream consistency metrics (e.g., Table 2 ALDEx2 fold diff 1.353, p<0.001) are only reproducible with the same filtered, rarefied inputs

## Limitations

- The 10% prevalence cut-off was arbitrarily selected; more work is needed to estimate an optimal threshold
- Rarefaction is criticized for reducing statistical power, introducing biases, and yielding high false positive rates with standard tests
- DA results depend on data pre-processing, so filtered vs. unfiltered runs can give drastically different numbers and sets of significant ASVs

## Evidence

- [methods] run_all_tools.sh on a single dataset with filtering: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [readme] Parameter meaning: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [discussion] Recommended prevalence filtering: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing"
- [intro] Rarefaction debate: "One area of disagreement is whether read count tables should be rarefied (i.e., subsampled) to correct for differing read depths across samples"
