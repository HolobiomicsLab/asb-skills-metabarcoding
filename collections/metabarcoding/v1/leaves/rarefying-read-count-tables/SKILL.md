---
name: rarefying-read-count-tables
description: Use when running DA tools that operate on rarefied count tables (e.g.
  the rarefied/CLR test variants in a multi-tool comparison) and sample library sizes
  differ, requiring subsampling to a fixed depth such as 2000 reads.
when_to_use_negative:
- When the downstream analysis is not DA testing — the paper states 'whether rarefying
  is advisable for other analyses, such as prior to computing diversity metrics, is
  beyond the scope of this work'.
- 'When the goal is maximum statistical power or minimal FDR: rarefying has been criticized
  for reducing statistical power, introducing biases, and yielding high false positive
  rates with standard tests.'
- When comparing to the paper's unfiltered results — mixing rarefied and non-rarefied
  variants without reporting pre-processing confounds cross-tool comparisons, since
  'results depend on data pre-processing'.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: run_all_tools.sh
  role: Pipeline script that accepts the -D rarefaction depth and -R rarefied ASV
    table arguments to run all 14 DA tools on rarefied data
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ALDEx2
  role: One of the 14 DA tools executed on rarefied and unfiltered dataset variants
    for comparison
- name: LEfSe
  role: One of the 14 DA tools executed on rarefied and unfiltered dataset variants
    for comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:78236638634e088e1f0e14c663255e0639ff74af93dfc7ecc64d02e8589c08be
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# rarefying-read-count-tables

## Summary

Rarefies (subsamples) 16S rRNA ASV read count tables to a fixed depth so that differing library sizes across samples are corrected before differential abundance (DA) testing with rarefied tool variants. The paper applies this within its 14-tool DA comparison at a rarefaction depth of 2000, while acknowledging that rarefying is a heavily criticized, contentious preprocessing choice.

## When to use

Use when running DA tools that operate on rarefied count tables (e.g. the rarefied/CLR test variants in a multi-tool comparison) and sample library sizes differ, requiring subsampling to a fixed depth such as 2000 reads.

## When NOT to use

- When the downstream analysis is not DA testing — the paper states 'whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work'.
- When the goal is maximum statistical power or minimal FDR: rarefying has been criticized for reducing statistical power, introducing biases, and yielding high false positive rates with standard tests.
- When comparing to the paper's unfiltered results — mixing rarefied and non-rarefied variants without reporting pre-processing confounds cross-tool comparisons, since 'results depend on data pre-processing'.

## Inputs

- ASV/OTU read count table (TSV) from 16S rRNA data, e.g. ASV_TABLE.tsv
- Metadata file (two-column TSV: sample names and sample groupings), e.g. METADATA_FILE.tsv

## Outputs

- Rarefied ASV table (RARIFIED_ASV_TABLE.tsv) subsampled to the chosen depth (e.g. 2000 reads/sample)
- Per-tool DA results computed on the rarefied table (significant ASV counts per dataset)

## How to apply

Pass the rarefaction depth via the run_all_tools.sh -D parameter (e.g. -D 2000), which supplies a rarefied ASV table (-R RARIFIED_ASV_TABLE.tsv) alongside the raw ASV table so rarefied tool variants can be executed. In the paper's pipeline, rarefying at depth 2000 was combined with 10% prevalence filtering (-F 0.1) when rerunning all 14 DA tools (ALDEx2 1.18.0, ANCOM-II, corncob 0.1.0, DESeq2 1.26.0, edgeR 3.28.1, LEfSe, MaAsLin2 0.99.12, metagenomeSeq 1.28.2, limma voom) on the 38 16S rRNA datasets. The rationale is to correct for differing read depths across samples, but note the paper documents that this approach has been heavily criticized for reducing statistical power, introducing biases, and yielding high false positive rates with standard tests; results therefore depend on data pre-processing. Always report whether rarefying was applied, since the number of significant ASVs per tool changes between rarefied and unfiltered variants.

## Related tools

- **run_all_tools.sh** (Pipeline script that accepts the -D rarefaction depth and -R rarefied ASV table arguments to run all 14 DA tools on rarefied data) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ALDEx2** (One of the 14 DA tools executed on rarefied and unfiltered dataset variants for comparison)
- **LEfSe** (One of the 14 DA tools executed on rarefied and unfiltered dataset variants for comparison)

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -R RARIFIED_ASV_TABLE.tsv -D 2000 -F 0.1
```

## Evaluation signals

- The rarefied ASV table's per-sample library sizes must all equal the specified rarefaction depth (e.g. 2000 reads), except samples below the depth which are excluded.
- Running run_all_tools.sh on the Human-ALL (MALL) dataset with rarefaction must produce outputs matching Misc_datafiles/Expected_MALL_output.
- Per-tool percentages of significant ASVs computed on rarefied variants should be tracked separately from unfiltered variants so filtered-vs-unfiltered differences (e.g. ANCOM-II filtered mean 3.8% vs limma voom TMMwsp unfiltered mean 40.5%) remain reproducible.
- The -D parameter value in the invocation must match the rarefaction depth used in the manuscript analysis (2000).

## Limitations

- Rarefying has been heavily criticized for reducing statistical power, introducing biases, and yielding high false positive rates with standard tests.
- Whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work.
- Results depend on data pre-processing: the number and set of significant ASVs identified by each tool changes between rarefied and unfiltered runs.
- The companion 10% prevalence filter cut-off was arbitrarily selected; an optimal cut-off is unknown.

## Evidence

- [intro] One area of disagreement is whether read count tables should be rarefied (i.e., subsampled) to correct for differing read depths across samples: "One area of disagreement is whether read count tables should be rarefied (i.e., subsampled) to correct for differing read depths across samples3. This approach has been heavily criticized"
- [readme] -D represents the rarefaction depth: "-D represents the rarefaction depth"
- [readme] run_all_tools.sh invocation with rarefaction depth and filtering: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [discussion] rarefying out of scope for other analyses: "whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work"
- [abstract] results depend on pre-processing: "these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing"
