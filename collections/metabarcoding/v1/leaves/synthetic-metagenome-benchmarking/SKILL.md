---
name: synthetic-metagenome-benchmarking
description: Use when you need to quantify and compare the accuracy of metagenomic
  taxonomic profilers at species level, and synthetic communities (CAMI 2, SynPhlAn)
  with known ground-truth abundances are available.
when_to_use_negative:
- The communities are real (non-synthetic) metagenomes with no ground-truth abundance
  profile, so OPAL precision/recall/RMSE against truth cannot be computed.
- The analysis goal is functional profiling or strain-level population genomics rather
  than species-level taxonomic abundance benchmarking.
- Only a single profiler is of interest with no alternative methods to compare against,
  making the benchmark table meaningless.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MetaPhlAn 4
  role: Primary profiler under evaluation; marker-based species-level profiling with
    SGB marker database covering 26,970 species-level genome bins
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: Baseline profiler (MetaPhlAn 3) run on the same 133 synthetic samples under
    default settings for comparison
  repo: https://github.com/biobakery/MetaPhlAn
- name: OPAL
  role: Benchmarking framework that computes per-sample precision, recall, F1, Bray-Curtis
    dissimilarity, RMSE, and TP/FP counts from predicted vs. ground-truth profiles
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:1197bb2d1bc05027ffee59924f0e08dff4cf26c6a942cc13d20d63f8ab292434
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - OPAL
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# synthetic-metagenome-benchmarking

## Summary

Benchmark species-level taxonomic profilers (MetaPhlAn 4 vs MetaPhlAn 3, mOTUs 2.6, Bracken 2.5) on synthetic metagenomes with known ground truth using the OPAL framework, comparing precision, recall, F1, Bray-Curtis dissimilarity, RMSE, and true/false-positive counts.

## When to use

Use when you need to quantify and compare the accuracy of metagenomic taxonomic profilers at species level, and synthetic communities (CAMI 2, SynPhlAn) with known ground-truth abundances are available.

## When NOT to use

- The communities are real (non-synthetic) metagenomes with no ground-truth abundance profile, so OPAL precision/recall/RMSE against truth cannot be computed.
- The analysis goal is functional profiling or strain-level population genomics rather than species-level taxonomic abundance benchmarking.
- Only a single profiler is of interest with no alternative methods to compare against, making the benchmark table meaningless.

## Inputs

- 128 CAMI 2 synthetic metagenome shotgun sequencing samples
- 5 SynPhlAn-nonhuman synthetic metagenome samples (133 synthetic samples total)
- Synthetic community ground-truth species relative-abundance profiles
- Species-level profiles from MetaPhlAn 4, MetaPhlAn 3, mOTUs 2.6, and Bracken 2.5 (run under default settings for baselines)

## Outputs

- Per-sample OPAL metrics: species-level precision, recall, F1, Bray-Curtis dissimilarity, RMSE on relative abundances, true/false-positive counts
- Aggregated benchmark table comparing all four profilers: opal_benchmark_metrics.csv (mean ± standard deviation per profiler)

## How to apply

Run MetaPhlAn 4 (marker-based profiling with the SGB marker database covering 26,970 species-level genome bins) on each of the 128 CAMI 2 and 5 SynPhlAn synthetic metagenomes to produce species-relative-abundance profiles. Run the baseline profilers (MetaPhlAn 3, mOTUs 2.6, Bracken 2.5) on the same 133 synthetic samples under default settings for comparability. Feed each predicted profile plus the synthetic ground truth into OPAL to compute per-sample species-level precision, recall, F1, Bray-Curtis dissimilarity, RMSE on relative abundances, and true/false-positive counts. Aggregate metrics per profiler as mean ± standard deviation; in the source study MetaPhlAn 4 achieved avg Bray-Curtis 0.13±0.07 and RMSE 0.016±0.019, outperforming MetaPhlAn 3 (avg BC 0.19±0.12; RMSE 0.019±0.018), with 96.65±66.08 true positives and 16.09±17.65 false positives versus 85.32±61.95 true positives for MetaPhlAn 3. Write the aggregated comparison to opal_benchmark_metrics.csv and validate that MetaPhlAn 4 means fall within one standard deviation of the reported anchors.

## Related tools

- **MetaPhlAn 4** (Primary profiler under evaluation; marker-based species-level profiling with SGB marker database covering 26,970 species-level genome bins) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (Baseline profiler (MetaPhlAn 3) run on the same 133 synthetic samples under default settings for comparison) — https://github.com/biobakery/MetaPhlAn
- **OPAL** (Benchmarking framework that computes per-sample precision, recall, F1, Bray-Curtis dissimilarity, RMSE, and TP/FP counts from predicted vs. ground-truth profiles)

## Examples

```
metaphlan sample.py --input_type fastq --bowtie2db metaphlan_databases/mpa_vJan21_CHOCOPhlAnSGB_202103 > profile.txt; opal.py -m profile.txt -o opal_out -g ground_truth.txt
```

## Evaluation signals

- opal_benchmark_metrics.csv exists and contains F1, Bray-Curtis, RMSE, TP, and FP fields for all four profilers (MetaPhlAn 4, MetaPhlAn 3, mOTUs 2.6, Bracken 2.5)
- MetaPhlAn 4 mean Bray-Curtis falls within one standard deviation of the reported anchor 0.13±0.07
- MetaPhlAn 4 mean RMSE falls within one standard deviation of the reported anchor 0.016±0.019
- MetaPhlAn 4 true-positive count is within one standard deviation of 96.65±66.08 and exceeds MetaPhlAn 3's 85.32±61.95 true positives
- All 133 synthetic samples (128 CAMI 2 + 5 SynPhlAn) are represented in the per-sample OPAL metrics

## Limitations

- MetaPhlAn 4's extended SGB and marker set still leaves under-characterized habitats poorly profiled: 'further work remains to better profile under-characterized habitats.'
- Environmental, nonhost-associated, and under-studied communities remain highly enriched for sequences not captured even by current uSGBs.
- Synthetic benchmark performance may not transfer to real communities; the study also notes a remaining sensitivity/specificity tradeoff when applying quality control to identify real but rare taxa.

## Evidence

- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins from a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins"
- [abstract] MetaPhlAn 4 is more accurate than alternatives on synthetic evaluations: "proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates"
- [results] On synthetic metagenomes MetaPhlAn 4 achieved avg Bray-Curtis 0.13±0.07 and RMSE 0.016±0.019, better than MetaPhlAn 3 (avg. BC 0.19±0.12; avg. RMSE 0.019±0.018): [evidence span withheld — anchor status: not_found]
- [other] Workflow: run MetaPhlAn 4 and baselines on 128 CAMI 2 and 5 SynPhlAn synthetic metagenomes, then feed predicted profiles plus ground truth into OPAL: [evidence span withheld — anchor status: not_found]
- [discussion] Even with the extended SGB and marker set, under-characterized habitats remain poorly profiled: "Notably, even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats."
- [readme] MetaPhlAn is a computational tool for species-level microbial profiling from metagenomic shotgun sequencing data: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
