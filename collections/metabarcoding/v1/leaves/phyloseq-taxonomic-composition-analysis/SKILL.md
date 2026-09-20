---
name: phyloseq-taxonomic-composition-analysis
description: Use when you have per-sample OTU/ASV tables with taxonomies (e.g., from
  PEMA, LotuS, or DADA2) plus a metadata.csv of sample habitats and you need taxonomic
  profiles, beta-diversity, or PERMANOVA testing of habitat differentiation.
when_to_use_negative:
- Input is raw .fastq.gz reads rather than an OTU/ASV abundance table — the metabarcoding
  pipeline must run first
- No sample metadata available — phyloseq downstream analysis in PEMA requires the
  metadata.csv file
- Marker genes outside 16S/18S rRNA, ITS, COI, or 12S rRNA scope of PEMA's supported
  pipelines
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_3307
tools:
- name: PEMA
  role: Containerized pipeline whose phyloseq step invokes the phyloseq_in_PEMA.R
    script on the generated OTU table
  repo: https://github.com/hariszaf/pema
- name: phyloseq
  role: R package performing taxonomic composition, alpha- and beta-diversity analysis,
    and statistical comparisons
  repo: http://joey711.github.io/phyloseq/index.html
- name: vegan
  role: R package whose functions (e.g., PERMANOVA) PEMA additionally invokes for
    downstream statistics
  repo: https://cran.r-project.org/web/packages/vegan/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:1a7f9b0a4979e0b61c14e7136c64197b34d0c51055e2c69c3a775df8314838fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BDS
  - LotuS
  - PEMA
  - QIIME 2
  - mothur
  - phyloseq
  - vegan
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# phyloseq-taxonomic-composition-analysis

## Summary

Run downstream ecological analysis of metabarcoding OTU/ASV tables through the phyloseq R package, computing taxonomic composition, alpha- and beta-diversity, and PERMANOVA tests on community composition (as PEMA does for 16S rRNA data from the Pavloudi et al. salinity-gradient dataset).

## When to use

Use when you have per-sample OTU/ASV tables with taxonomies (e.g., from PEMA, LotuS, or DADA2) plus a metadata.csv of sample habitats and you need taxonomic profiles, beta-diversity, or PERMANOVA testing of habitat differentiation.

## When NOT to use

- Input is raw .fastq.gz reads rather than an OTU/ASV abundance table — the metabarcoding pipeline must run first
- No sample metadata available — phyloseq downstream analysis in PEMA requires the metadata.csv file
- Marker genes outside 16S/18S rRNA, ITS, COI, or 12S rRNA scope of PEMA's supported pipelines

## Inputs

- OTU or ASV table with taxa and per-sample abundances produced by PEMA (e.g., 16S rRNA data)
- metadata.csv describing samples (comma-separated; e.g., river, lagoon, open sea habitats and salinity)
- phyloseq_in_PEMA.R R script from the PEMA analysis directory
- OTU tables from comparison pipelines (LotuS, DADA2) for cross-pipeline PERMANOVA

## Outputs

- Taxonomic composition profiles per sample
- Alpha- and beta-diversity statistics and visualizations
- PERMANOVA statistics comparing PEMA, LotuS, and DADA2 community compositions
- permanova_comparison.tsv with all statistics and OTU-number-vs-salinity tabulation
- FASTQC-based read quality statistics per sample (from the enclosing PEMA run)

## How to apply

Place the phyloseq_in_PEMA.R script and a comma-separated metadata.csv in the PEMA analysis directory alongside parameters.tsv and mydata/, then run PEMA (./pema_latest.bds) so the phyloseq step executes on the resulting OTU table. The script computes taxonomic composition profiles, alpha- and beta-diversity, statistical comparisons, and correlations between samples via phyloseq (with select vegan functions). For the cross-pipeline comparison, run PERMANOVA on the community compositions of PEMA, LotuS, and DADA2 to test habitat differentiation among river, lagoon, and open sea samples, and tabulate per-sample OTU numbers against salinity. Validate by reproducing PEMA F.Model = 7.0718, P < 0.001 and LotuS F.Model = 6.5901, P < 0.001, with OTU counts decreasing as salinity increases.

## Related tools

- **PEMA** (Containerized pipeline whose phyloseq step invokes the phyloseq_in_PEMA.R script on the generated OTU table) — https://github.com/hariszaf/pema
- **phyloseq** (R package performing taxonomic composition, alpha- and beta-diversity analysis, and statistical comparisons) — http://joey711.github.io/phyloseq/index.html
- **vegan** (R package whose functions (e.g., PERMANOVA) PEMA additionally invokes for downstream statistics) — https://cran.r-project.org/web/packages/vegan/index.html

## Examples

```
./pema_latest.bds  # with analysis_directory containing parameters.tsv, mydata/, metadata.csv, phyloseq_in_PEMA.R; or Rscript phyloseq_in_PEMA.R
```

## Evaluation signals

- PERMANOVA on PEMA results must reproduce F.Model = 7.0718 with P < 0.001; LotuS must give F.Model = 6.5901, P < 0.001
- Per-sample OTU numbers must show a decreasing trend with increasing salinity (river > lagoon > open sea)
- metadata.csv must be comma-separated and named exactly as required or PEMA's phyloseq step fails
- Statistics are written to permanova_comparison.tsv for the PEMA/LotuS/DADA2 comparison
- Taxonomic profiles and beta-diversity computed by the provided phyloseq R script should show comparable quality to established pipelines (mothur, QIIME 2)

## Limitations

- PEMA will fail unless the mandatory files (parameters.tsv, mydata/, metadata.csv, phyloseq_in_PEMA.R) are named exactly as described
- Only some parts of phyloseq_in_PEMA.R are user-adjustable; the main features must stay the same for the script to execute as part of PEMA
- Parameter file version must match the PEMA version used (e.g., v.2.1.4 parameters for pema:v.2.1.4)

## Evidence

- [intro] In addition, ecological and phylogenetic analysis are facilitated via the “phyloseq” R package [27].: "In addition, ecological and phylogenetic analysis are facilitated via the “phyloseq” R package [27]."
- [methods] PERMANOVA on PEMA, LotuS and DADA2 results: "when the PERMANOVA analysis was run for the results of PEMA, LotuS, and DADA2"
- [methods] alpha/beta diversity and taxonomic composition: "This includes α- and β-diversity analysis, taxonomic composition, statistical comparisons, and calculation of correlations between samples"
- [readme] PEMA performs all the basic functions of the phyloseq R package: [evidence span withheld — anchor status: not_found]
- [readme] phyloseq_in_PEMA.R is an R script implemented with fixed main features: [evidence span withheld — anchor status: not_found]
