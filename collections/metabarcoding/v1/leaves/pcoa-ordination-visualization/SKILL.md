---
name: pcoa-ordination-visualization
description: Use when you have a rarefied OTU/feature table and sample metadata with
  a grouping column, and need to visualize between-sample beta diversity and test
  group separation (e.g., bray_curtis.txt ordination with Group-colored samples).
when_to_use_negative:
- Input is a raw, non-rarefied OTU table — distance metrics require equal-sampling
  standardization (vegan resampling) first
- The goal is alpha (within-sample) diversity rather than between-sample beta diversity
  ordination
- Only a single sample group exists — pairwise between-group p-value comparison is
  undefined
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: EasyAmplicon
  role: Pipeline providing the beta_pcoa.R script used for PCoA ordination and the
    beta_pcoa_stat.txt pairwise p-value statistics
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: USEARCH
  role: Builds the OTU tree (cluster_agg) and computes beta-diversity distance matrices
    (beta_div) that feed the PCoA ordination
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:707ab28bf7fc5cb964795c4b335f593e573e94a5018d6c06c63bae05798bdf12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - USEARCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pcoa-ordination-visualization

## Summary

Perform PCoA ordination on beta-diversity distance matrices (e.g., Bray-Curtis) within the EasyAmplicon pipeline, producing a colored PCoA plot PDF and a pairwise group-comparison p-value table.

## When to use

Use when you have a rarefied OTU/feature table and sample metadata with a grouping column, and need to visualize between-sample beta diversity and test group separation (e.g., bray_curtis.txt ordination with Group-colored samples).

## When NOT to use

- Input is a raw, non-rarefied OTU table — distance metrics require equal-sampling standardization (vegan resampling) first
- The goal is alpha (within-sample) diversity rather than between-sample beta diversity ordination
- Only a single sample group exists — pairwise between-group p-value comparison is undefined

## Inputs

- Bray-Curtis (or other beta-diversity) distance matrix, e.g., result/beta/bray_curtis.txt from usearch -beta_div
- Rarefied OTU/feature table (result/otutab_rare.txt) for distance computation
- Sample metadata / design file with a grouping column (result/metadata.txt, --group Group)
- Phylogenetic tree built from OTU representative sequences (result/otus.tree), required for UniFrac

## Outputs

- PCoA ordination plot PDF colored by group (result/beta/bray_curtis.pcoa.pdf)
- Pairwise between-group comparison p-values table (result/beta/beta_pcoa_stat.txt)
- Additional distance matrices (Bray-Curtis, Euclidean, UniFrac, etc.) under result/beta/

## How to apply

First build the phylogenetic tree from representative sequences with USEARCH hierarchical average-linkage clustering (usearch -cluster_agg result/otus.fa -treeout result/otus.tree), then compute inter-sample distance matrices from the rarefied feature table (usearch -beta_div result/otutab_rare.txt -tree result/otus.tree -filename_prefix result/beta/), which yields Bray-Curtis, Euclidean, UniFrac and related matrices. Ordinate the Bray-Curtis matrix using the EasyAmplicon R script beta_pcoa.R, passing the distance matrix as --input, result/metadata.txt as --design, the grouping column (e.g., Group) to color samples by, and an output PDF path. Finally, compute pairwise between-group significance p-values from the ordination/distance analysis and write them to result/beta/beta_pcoa_stat.txt. Rarefaction/equal-sampling standardization of the table (R/vegan resampling) should precede distance computation since beta metrics are sensitive to sequencing depth.

## Related tools

- **EasyAmplicon** (Pipeline providing the beta_pcoa.R script used for PCoA ordination and the beta_pcoa_stat.txt pairwise p-value statistics) — https://github.com/YongxinLiu/EasyAmplicon
- **USEARCH** (Builds the OTU tree (cluster_agg) and computes beta-diversity distance matrices (beta_div) that feed the PCoA ordination)

## Examples

```
Rscript ${db}/script/beta_pcoa.R --input result/beta/bray_curtis.txt --design result/metadata.txt --group Group --output result/beta/bray_curtis.pcoa.pdf
```

## Evaluation signals

- Distance matrix column count equals the number of samples in result/metadata.txt
- bray_curtis.pcoa.pdf is generated in result/beta/
- beta_pcoa_stat.txt contains a p-value for each pairwise group comparison defined by the Group column
- Distance matrices (bray_curtis.txt, unifrac, euclidean, etc.) are written under the result/beta/ filename prefix
- Tree file result/otus.tree exists and is used consistently with the rarefied table for UniFrac distances

## Limitations

- PCoA output depends on prior rarefaction; beta metrics are sensitive to sequencing-depth differences unless normalized via vegan resampling
- UniFrac requires a valid phylogenetic tree (result/otus.tree) built from the same representative sequences (result/otus.fa); without it, phylogenetic distances cannot be computed
- The workflow is tied to EasyAmplicon's expected directory layout (result/beta/) and metadata design file; deviating inputs require adapting the script arguments

## Evidence

- [methods] Ordinate the Bray-Curtis matrix with the R script beta_pcoa.R: [evidence span withheld — anchor status: not_found]
- [methods] Normalize for differences in sequencing depth between samples: "Normalize for differences in sequencing depth between samples. Use the R package vegan script to resample"
- [abstract] diversity analysis, and data visualization: "including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis, diversity analysis, and data visualization"
- [readme] EasyAmplicon pipeline is freely available on GitHub: "[Liu, et al. 2023. **iMeta**. EasyAmplicon: An easy-to-use, open-source, reproducible, and community-based pipeline for amplicon data analysis in microbiome research.](https://doi.org/10.1002/imt2.83)"
