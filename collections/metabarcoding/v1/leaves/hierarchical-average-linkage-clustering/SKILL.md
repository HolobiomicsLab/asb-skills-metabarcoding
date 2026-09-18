---
name: hierarchical-average-linkage-clustering
description: Use when you have a plant-species (or site) x arthropod-OTU count table
  from eDNA metabarcoding and need to visualise/quantify whether community composition
  clusters by sample grouping via average-linkage dendrograms.
when_to_use_negative:
- The question concerns gradient-based relationships with environmental predictors
  better handled by ordination/RDA (the article used RDA separately for that purpose).
- The count table is dominated by high-abundance taxa you do not want down-weighted
  and a presence/absence or untransformed metric is preferred; Raup-Crick on cube-root-transformed
  counts is tuned to incidence-driven metabarcoding data.
- Data are continuous trait measurements rather than community count data, where Raup-Crick
  dissimilarity is undefined or inappropriate.
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: vegan
  role: vegdist with the 'raupcrick' method computes the pairwise Raup-Crick dissimilarity
    matrix from the (cube-root-transformed) community table
- name: pheatmap
  role: renders the clustered sample dendrogram/heatmap image for the figure
- name: R
  role: analysis environment in which the transformation, clustering and heatmap rendering
    are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:ab57ac3950368b84a1ac6f8dfc6108a4ba6989230fc46f9446d5daccbd70f944
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pheatmap
  - vegan
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# hierarchical-average-linkage-clustering

## Summary

Cluster arthropod eDNA community samples (e.g. flower samples by plant species) using Raup-Crick dissimilarity and UPGMA (average-linkage) hierarchical clustering, rendered as a pheatmap dendrogram, to test whether communities segregate by plant species.

## When to use

Use when you have a plant-species (or site) x arthropod-OTU count table from eDNA metabarcoding and need to visualise/quantify whether community composition clusters by sample grouping via average-linkage dendrograms.

## When NOT to use

- The question concerns gradient-based relationships with environmental predictors better handled by ordination/RDA (the article used RDA separately for that purpose).
- The count table is dominated by high-abundance taxa you do not want down-weighted and a presence/absence or untransformed metric is preferred; Raup-Crick on cube-root-transformed counts is tuned to incidence-driven metabarcoding data.
- Data are continuous trait measurements rather than community count data, where Raup-Crick dissimilarity is undefined or inappropriate.

## Inputs

- plant-species x arthropod-OTU community count table (e.g. CSV/TSV of eDNA read counts per sample)
- sample metadata mapping samples to plant species (Tanacetum, Echium, Angelica, etc.)

## Outputs

- average-linkage (UPGMA) sample dendrogram / cluster heatmap (e.g. figure5_cluster_dendrogram.png)
- Raup-Crick dissimilarity matrix between samples

## How to apply

Load the plant-species x arthropod-OTU community count table into R and apply a cube-root (n^(1/3)) transformation to down-weight dominant taxa. Compute pairwise dissimilarities with vegan::vegdist using the 'raupcrick' method, which is incidence-based and appropriate for metabarcoding presence/absence-dominated data. Build the dendrogram with hclust(method='average') (UPGMA) on the dissimilarity matrix and render it with pheatmap, e.g. saving as figure5_cluster_dendrogram.png. Judge correctness by checking that samples from the same plant species (Tanacetum, Echium, Angelica) form the tightest clusters, while remembering the paper's conclusion that communities were only somewhat, not perfectly, segregated by plant species.

## Related tools

- **vegan** (vegdist with the 'raupcrick' method computes the pairwise Raup-Crick dissimilarity matrix from the (cube-root-transformed) community table)
- **pheatmap** (renders the clustered sample dendrogram/heatmap image for the figure)
- **R** (analysis environment in which the transformation, clustering and heatmap rendering are executed)

## Examples

```
d <- vegan::vegdist(comm^(1/3), method='raupcrick'); hc <- hclust(d, method='average'); pheatmap::pheatmap(comm^(1/3), clustering_distance_rows=d, clustering_method='average', filename='figure5_cluster_dendrogram.png')
```

## Evaluation signals

- The dissimilarity matrix returned by vegan::vegdist(..., method='raupcrick') is square, symmetric, with zero diagonal and values in [0,1].
- Samples from the same plant species (Tanacetum, Echium and Angelica) merge at the lowest linkage heights, forming the tightest clusters in the dendrogram.
- Clustering is run on cube-root (n^(1/3))-transformed abundances; verify the transformation was applied before vegdist, since raw counts over-weight dominant taxa.
- The hclust object uses method='average' (UPGMA), and the pheatmap output is saved as figure5_cluster_dendrogram.png.
- Cluster segregation should be interpreted as partial: the paper found communities were 'only somewhat, not perfectly, segregated by plant species', so perfect by-species separation is not expected.

## Limitations

- Arthropod eDNA communities were only somewhat, not perfectly, segregated by plant species — clustering supports a partial, not absolute, pattern.
- The experiment was not exhaustive and higher arthropod richness could be obtained, so cluster resolution may improve with deeper sequencing or more samples.
- Raup-Crick on cube-root-transformed data down-weights dominant taxa; conclusions depend on this transformation choice.

## Evidence

- [results] Cluster analyses (Raup–Crick, average-linkage) and RDA showed that arthropod eDNA communities were only somewhat, not perfectly, segregated by plant species.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the Raup-Crick dissimilarity matrix between samples using vegan::vegdist with the raupcrick method.: [evidence span withheld — anchor status: not_found]
- [methods] Apply a cube-root (n^(1/3)) transformation to the abundance values to down-weight dominant taxa.: [evidence span withheld — anchor status: not_found]
- [abstract] Arthropod communities clustered together according to plant species.: "Arthropod communities clustered together according to plant species."
