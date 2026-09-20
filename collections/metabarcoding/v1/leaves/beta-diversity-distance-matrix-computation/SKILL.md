---
name: beta-diversity-distance-matrix-computation
description: Use when a rarefied feature/OTU table (result/otutab_rare.txt), sample
  metadata with a Group column, and an OTU tree (result/otus.tree) are available and
  the goal is beta-diversity distance matrices, a PCoA plot, and pairwise group p-values.
when_to_use_negative:
- The feature table has not been rarefied/standardized to equal sampling depth — normalize
  for sequencing depth differences first (e.g., vegan resampling in EasyAmplicon)
- The analysis goal is alpha diversity or taxonomic composition rather than inter-sample
  dissimilarity and ordination
- UniFrac is required but no phylogenetic tree of representative sequences has been
  or can be built
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3307
tools:
- name: USEARCH
  role: Builds the OTU tree (-cluster_agg) and computes inter-sample beta-diversity
    distance matrices (-beta_div) from the rarefied OTU table
- name: EasyAmplicon
  role: Pipeline providing the beta_pcoa.R script for PCoA ordination and the beta_pcoa_stat.txt
    pairwise significance testing, all writing to result/beta/
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Companion database/scripts package supplying the R scripts used by EasyAmplicon
    for diversity analysis
  repo: https://github.com/YongxinLiu/EasyMicrobiome
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:240592690f1b254a1e907fb14850753429da9baa036d6018f5aeb23dc2d4bc0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - USEARCH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# beta-diversity-distance-matrix-computation

## Summary

Compute inter-sample beta-diversity distance matrices (Bray-Curtis, Euclidean, UniFrac) from a rarefied OTU/feature table using USEARCH, then ordinate the Bray-Curtis matrix via PCoA and test pairwise group differences as implemented in the EasyAmplicon pipeline.

## When to use

Use when a rarefied feature/OTU table (result/otutab_rare.txt), sample metadata with a Group column, and an OTU tree (result/otus.tree) are available and the goal is beta-diversity distance matrices, a PCoA plot, and pairwise group p-values.

## When NOT to use

- The feature table has not been rarefied/standardized to equal sampling depth — normalize for sequencing depth differences first (e.g., vegan resampling in EasyAmplicon)
- The analysis goal is alpha diversity or taxonomic composition rather than inter-sample dissimilarity and ordination
- UniFrac is required but no phylogenetic tree of representative sequences has been or can be built

## Inputs

- Rarefied OTU/feature table (result/otutab_rare.txt) with samples as columns
- Representative OTU sequences FASTA (result/otus.fa) for tree building
- Sample metadata with a Group column (result/metadata.txt)
- Phylogenetic tree of OTUs (result/otus.tree) for UniFrac distances

## Outputs

- Distance matrices (Bray-Curtis, Euclidean, UniFrac, etc.) written under result/beta/
- PCoA ordination plot colored by Group (beta/pcoa_bray_curtis.pdf / bray_curtis.pcoa.pdf)
- beta_pcoa_stat.txt with pairwise between-group comparison p-values

## How to apply

First build the phylogenetic tree from representative OTU sequences with USEARCH hierarchical average-linkage clustering (`usearch -cluster_agg result/otus.fa -treeout result/otus.tree`), since UniFrac distances require it. Then generate inter-sample distance matrices from the rarefied feature table with `usearch -beta_div result/otutab_rare.txt -tree result/otus.tree -filename_prefix result/beta/`, which writes Bray-Curtis, Euclidean, UniFrac and related matrices under the result/beta/ prefix. Ordinate the Bray-Curtis matrix with the EasyAmplicon R script beta_pcoa.R, passing the distance matrix, result/metadata.txt, the Group column for coloring, and an output PDF path. Finally compute pairwise between-group significance p-values from the ordination/distance analysis into result/beta/beta_pcoa_stat.txt. Validate that the distance matrix dimensionality matches the sample count in the metadata before trusting downstream plots.

## Related tools

- **USEARCH** (Builds the OTU tree (-cluster_agg) and computes inter-sample beta-diversity distance matrices (-beta_div) from the rarefied OTU table)
- **EasyAmplicon** (Pipeline providing the beta_pcoa.R script for PCoA ordination and the beta_pcoa_stat.txt pairwise significance testing, all writing to result/beta/) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Companion database/scripts package supplying the R scripts used by EasyAmplicon for diversity analysis) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
usearch -beta_div result/otutab_rare.txt -tree result/otus.tree -filename_prefix result/beta/
```

## Evaluation signals

- The column (and row) count of each distance matrix equals the number of samples in result/metadata.txt
- bray_curtis.pcoa.pdf is generated in result/beta/ and samples are colored by the metadata Group column
- beta_pcoa_stat.txt exists and contains a p-value for every pairwise group comparison of the Group factor
- result/otus.tree exists and USEARCH -beta_div produced multiple matrices (Bray-Curtis, Euclidean, UniFrac) under the result/beta/ filename prefix
- The input table is the rarefied otutab_rare.txt (equal-depth standardized), not the raw otutab.txt

## Limitations

- UniFrac distances depend on the quality of the OTU tree built by USEARCH hierarchical average-linkage clustering; errors in tree building propagate to UniFrac matrices
- Distance matrices are only comparable to metadata if the sample order/names match result/metadata.txt exactly
- The R packages/scripts must be installed from the EasyMicrobiome/pre-packaged R library archive; missing dependencies break beta_pcoa.R

## Evidence

- [methods] beta_div workflow with tree: [evidence span withheld — anchor status: not_found]
- [results] PCoA outputs and pairwise p-values: [evidence span withheld — anchor status: not_found]
- [methods] Tree building step: [evidence span withheld — anchor status: not_found]
- [abstract] Diversity analysis in pipeline scope: "including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis, diversity analysis, and data visualization"
- [methods] Rarefaction precondition: [evidence span withheld — anchor status: not_found]
