---
name: count-data-cube-root-transformation
description: Use when a raw read-count community matrix (samples x OTUs) contains
  highly dominant taxa that would bias dissimilarity-based ordination or hierarchical
  clustering, and a milder alternative to presence/absence conversion is needed.
when_to_use_negative:
- The abundance table has already been normalized or transformed (e.g. rarefied, Hellinger-
  or clr-transformed), where a second transformation would distort distances.
- The analysis goal requires strict presence/absence community comparisons, in which
  case the transformation is unnecessary.
- Count data are not community abundance data (e.g. continuous measurements or compositional
  percentages), for which a cube root of counts is not meaningful.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3058
tools:
- name: R
  role: Environment in which the cube-root transformation is applied element-wise
    to the count matrix before dissimilarity computation
- name: vegan
  role: Consumes the transformed matrix via vegan::vegdist(method='raupcrick') to
    produce the Raup–Crick dissimilarity matrix
- name: pheatmap
  role: Renders the clustered sample dendrogram/heatmap built from the transformed-data-derived
    clustering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:973ce71d54b54f3950f16f99f46fa8d2cb06e34d9efd66b52ac77336b4bbe368
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pheatmap
  - vegan
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3058
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# Count-data cube-root transformation

## Summary

Applies a cube-root (n^(1/3)) transformation to plant-species x arthropod-OTU abundance count tables before computing Raup–Crick dissimilarity, down-weighting dominant taxa so that average-linkage clustering of flower eDNA communities is not driven by a few highly abundant OTUs.

## When to use

Use when a raw read-count community matrix (samples x OTUs) contains highly dominant taxa that would bias dissimilarity-based ordination or hierarchical clustering, and a milder alternative to presence/absence conversion is needed.

## When NOT to use

- The abundance table has already been normalized or transformed (e.g. rarefied, Hellinger- or clr-transformed), where a second transformation would distort distances.
- The analysis goal requires strict presence/absence community comparisons, in which case the transformation is unnecessary.
- Count data are not community abundance data (e.g. continuous measurements or compositional percentages), for which a cube root of counts is not meaningful.

## Inputs

- Plant-species x arthropod-OTU community count table (raw eDNA read abundances per flower sample)
- Sample metadata mapping each row to the plant species sampled (e.g. Tanacetum, Echium, Angelica)

## Outputs

- Cube-root-transformed abundance matrix of the same dimensions as the input count table
- Downstream Raup–Crick dissimilarity matrix and average-linkage (UPGMA) sample dendrogram/heatmap (figure5_cluster_dendrogram.png)

## How to apply

Load the plant-species x arthropod-OTU community count table into R and apply the cube-root transformation element-wise to every abundance value, i.e. value^(1/3), to reduce the leverage of dominant taxa. The transformed matrix is then used as input to vegan::vegdist with method='raupcrick' to build the dissimilarity matrix, so the transformation directly shapes downstream UPGMA clustering via hclust(method='average'). The cube root was chosen as an intermediate between retaining relative abundances and discarding them entirely (presence/absence), preserving some quantitative structure while tempering dominance. After clustering, render the dendrogram/heatmap with pheatmap and inspect whether communities from Tanacetum, Echium and Angelica form the tightest clusters, keeping in mind the paper's finding that segregation by plant species was only partial.

## Related tools

- **R** (Environment in which the cube-root transformation is applied element-wise to the count matrix before dissimilarity computation)
- **vegan** (Consumes the transformed matrix via vegan::vegdist(method='raupcrick') to produce the Raup–Crick dissimilarity matrix)
- **pheatmap** (Renders the clustered sample dendrogram/heatmap built from the transformed-data-derived clustering)

## Examples

```
Rscript -e "x <- read.csv('otu_counts.csv', row.names=1); xt <- x^(1/3); d <- vegan::vegdist(xt, method='raupcrick'); hc <- hclust(d, method='average'); pheatmap::pheatmap(xt, cluster_cols=hc, filename='figure5_cluster_dendrogram.png')"
```

## Evaluation signals

- Every cell of the transformed matrix equals the cube root of the corresponding raw count, and dimensions (samples x OTUs) are unchanged.
- No negative or missing values are introduced; zero counts remain zero after transformation.
- The relative influence of previously dominant OTUs on the Raup–Crick dissimilarity matrix decreases compared with clustering on raw counts.
- The resulting average-linkage dendrogram groups flower samples by plant species only partially, matching the paper's finding that communities were 'only somewhat, not perfectly, segregated by plant species'.
- Tanacetum, Echium and Angelica samples form the tightest clusters in the pheatmap output.

## Limitations

- The transformation is a choice that trades off quantitative information against dominance artifacts; the article reports only partial, not perfect, segregation of communities by plant species, so clustering results should not be over-interpreted.
- The experiment was not exhaustive — higher arthropod richness could be obtained — so the count table underlying the transformation may under-sample the true community.

## Evidence

- [methods] Cube-root transformation rationale: [evidence span withheld — anchor status: not_found]
- [methods] Clustering outcome: [evidence span withheld — anchor status: not_found]
- [methods] Partial segregation finding: [evidence span withheld — anchor status: not_found]
- [abstract] Community clustering by plant species: "Arthropod communities clustered together according to plant species."
