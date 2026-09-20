---
name: permutation-based-false-positive-estimation
description: Use when comparing DA tools' false-positive control on 16S rRNA ASV count
  tables with two-group metadata, or when validating that a DA pipeline's significant-hit
  rates are not inflated under randomized (null) groupings.
when_to_use_negative:
- Real sample groupings with genuine biological differences — the randomized null
  is not a perfect representation of real data, so low significant hits here do not
  prove low practical FDR
- Analyses where the goal is consensus DA calling on true case/control contrasts rather
  than tool benchmarking (use the consensus approach of several methods instead)
- Datasets with more than two sample groupings — the run_all_tools.sh metadata format
  requires maximum 2 groupings
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: edgeR
  role: DA tool benchmarked; flagged as a false-positive outlier on filtered data
    (mean 0.69–27.9% significant ASVs)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: LEfSe
  role: DA tool benchmarked; flagged as a false-positive outlier on filtered data
    (mean 3.4–5.1% significant ASVs)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ALDEx2
  role: DA tool benchmarked; excluded from the 90-replicate unfiltered extension run
    (run_FP_unfilt_EXT.sh)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ANCOM
  role: DA tool benchmarked; excluded from the 90-replicate unfiltered extension run
    (run_FP_unfilt_EXT.sh); standalone code archived separately
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: ANCOM-II
  role: DA tool benchmarked; its preprocessing methodology handles different types
    of zeros in the archived ANCOM pipeline
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: DESeq2
  role: DA tool benchmarked under permuted labels
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: corncob
  role: DA tool benchmarked under permuted labels; excluded from the 90-replicate
    unfiltered extension run
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: metagenomeSeq
  role: DA tool benchmarked under permuted labels
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: limma voom
  role: DA tool benchmarked under permuted labels
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:e71a27562840c1ed4150b724925ceacb2e8d03ea30b81002a72545aacd058cf2
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# permutation-based-false-positive-estimation

## Summary

Benchmark microbiome differential abundance (DA) tools by randomly reassigning case/control labels within a dataset's most frequently sampled group, then measuring the percent of ASVs each tool calls significant — an empirical false-positive rate with no expected group differences. Used in Nearing et al. 2022 to show edgeR (mean 0.69–27.9%) and LEfSe (mean 3.4–5.1%) were outliers on prevalence-filtered 16S rRNA data.

## When to use

Use when comparing DA tools' false-positive control on 16S rRNA ASV count tables with two-group metadata, or when validating that a DA pipeline's significant-hit rates are not inflated under randomized (null) groupings.

## When NOT to use

- Real sample groupings with genuine biological differences — the randomized null is not a perfect representation of real data, so low significant hits here do not prove low practical FDR
- Analyses where the goal is consensus DA calling on true case/control contrasts rather than tool benchmarking (use the consensus approach of several methods instead)
- Datasets with more than two sample groupings — the run_all_tools.sh metadata format requires maximum 2 groupings

## Inputs

- Raw 16S rRNA ASV count table (tsv, taxa x samples) per dataset
- Two-column metadata tsv (sample names, sample groupings; maximum 2 groupings)
- Rarefied ASV table (e.g., rarefaction depth D=2000) where needed
- Prevalence-filtered count tables (10% prevalence filter, F=0.1)

## Outputs

- Per-tool, per-replicate percent of ASVs called significantly DA under randomized labels
- Aggregated false-positive results table across replicates and datasets (Fig. 4 layout, filtered and unfiltered conditions)
- Per-dataset DA tool output folders from run_all_tools.sh

## How to apply

Download raw 16S rRNA ASV count tables and two-column metadata groupings (sample name + group, max 2 groupings) from the figshare deposit (14531724). For each dataset, identify the most frequent sample group and randomly permute the case/control labels among its samples, since no group differences are then expected. Run each DA tool (ALDEx2, ANCOM, ANCOM-II, LEfSe, DESeq2, edgeR, corncob, metagenomeSeq, limma voom) via run_all_tools.sh with prevalence filtering F=0.1 (10%) and rarefaction depth D=2000 for the filtered condition, using run_all_FP_datasets_filt.sh or run_all_FP_datasets_unfilt.sh; for the extended unfiltered run use run_FP_unfilt_EXT.sh with 90 replicates, which applies to all tools except ANCOM, ALDEx2, and corncob. For every tool and replicate, compute the percent of ASVs called significantly differentially abundant, then aggregate across replicates and datasets into a Fig. 4-style results table for filtered and unfiltered conditions.

## Related tools

- **edgeR** (DA tool benchmarked; flagged as a false-positive outlier on filtered data (mean 0.69–27.9% significant ASVs)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **LEfSe** (DA tool benchmarked; flagged as a false-positive outlier on filtered data (mean 3.4–5.1% significant ASVs)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ALDEx2** (DA tool benchmarked; excluded from the 90-replicate unfiltered extension run (run_FP_unfilt_EXT.sh)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ANCOM** (DA tool benchmarked; excluded from the 90-replicate unfiltered extension run (run_FP_unfilt_EXT.sh); standalone code archived separately) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **ANCOM-II** (DA tool benchmarked; its preprocessing methodology handles different types of zeros in the archived ANCOM pipeline) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **DESeq2** (DA tool benchmarked under permuted labels) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **corncob** (DA tool benchmarked under permuted labels; excluded from the 90-replicate unfiltered extension run) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **metagenomeSeq** (DA tool benchmarked under permuted labels) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **limma voom** (DA tool benchmarked under permuted labels) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- edgeR on 10% prevalence-filtered data yields mean significant-ASV percentages within the reported 0.69–27.9% range
- LEfSe on filtered data yields mean rates within the reported 3.4–5.1% range
- ALDEx2 and the t-test on rarefied data consistently yield 0% significant ASVs on unfiltered data
- All per-tool/dataset percentages fall inside the bounds of Fig. 4b; flag any combination outside these ranges
- Permutation replicates (90 for the run_FP_unfilt_EXT.sh extension) are completed for every included tool/dataset combination

## Limitations

- The randomized approach for estimating FDR is not a perfect representation of real data; real sample groupings will likely contain some systematic differences in microbial abundances, so few significant ASVs under this approach is not proof of low practical FDR
- Reasons why some methods have much higher significance rates than other tools remain unclear
- ANCOM, ALDEx2, and corncob were excluded from the additional 90-replicate unfiltered extension (run_FP_unfilt_EXT.sh), limiting replicate coverage for those tools
- The standalone ANCOM functions in the FrederickHuangLin/ANCOM-Code-Archive repository are no longer maintained; users are directed to the ANCOMBC R package

## Evidence

- [results] edgeR and LEfSe outliers on filtered data: [evidence span withheld — anchor status: not_found]
- [readme] filtered run parameters: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [readme] FP scripts listed: "run_all_FP_datasets_filt.sh --> runs the false postive analysis for filtered datasets"
- [readme] extended replicates: "run_FP_unfilt_EXT.sh --> runs an additional 90 replicates for the unfilt FP analysis on all tools except for ANCOM, ALDEx2, and corncob."
- [discussion] FDR caveat: "our randomized approach for estimating FDR is not a perfect representation of real data; that is, real sample groupings will likely contain some systematic differences in microbial abundances"
- [readme] metadata format: "the metadatafile should be a two column tsv table with the first column having the sample names and the second column having the sample groupings.(maximum 2 groupings)"
