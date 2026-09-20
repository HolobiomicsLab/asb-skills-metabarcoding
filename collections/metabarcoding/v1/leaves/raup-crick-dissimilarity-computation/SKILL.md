---
name: raup-crick-dissimilarity-computation
description: Use when you have a sample × taxon count table (e.g. plant-species ×
  arthropod-OTU from eDNA metabarcoding) and need a presence/absence-weighted dissimilarity
  matrix and clustered dendrogram/heatmap to test whether communities group by a categorical
  factor such as host plant species.
when_to_use_negative:
- The question requires ordination rather than clustering — this study complemented
  Raup–Crick/average-linkage clustering with RDA, which should be used when a constrained
  gradient analysis is needed.
- The abundance table has not been transformed and dominant taxa would overwhelm presence/absence-driven
  Raup–Crick dissimilarity — apply the cube-root transformation first.
- Input is raw sequencing reads rather than an OTU count table; metabarcoding must
  be completed first.
edam_operation: http://edamontology.org/operation_2929
tools:
- name: R
  role: Statistical environment in which the dissimilarity computation, hierarchical
    clustering and heatmap rendering are scripted
- name: vegan
  role: vegan::vegdist with the raupcrick method computes the Raup–Crick dissimilarity
    matrix from the transformed OTU table
- name: pheatmap
  role: Renders the clustered sample dendrogram/heatmap (figure5_cluster_dendrogram.png)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:a87bb786a9ef7440017450a626012408f348396f78ff16799bf4879b1bda4883
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pheatmap
  - vegan
  edam_operation: http://edamontology.org/operation_2929
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# raup-crick-dissimilarity-computation

## Summary

Computes Raup–Crick dissimilarity between community samples and builds an average-linkage (UPGMA) hierarchical clustering to test whether arthropod eDNA communities recovered from wild flowers segregate by plant species. In this workflow the plant-species × arthropod-OTU count table is cube-root transformed before clustering and rendered as a pheatmap dendrogram.

## When to use

Use when you have a sample × taxon count table (e.g. plant-species × arthropod-OTU from eDNA metabarcoding) and need a presence/absence-weighted dissimilarity matrix and clustered dendrogram/heatmap to test whether communities group by a categorical factor such as host plant species.

## When NOT to use

- The question requires ordination rather than clustering — this study complemented Raup–Crick/average-linkage clustering with RDA, which should be used when a constrained gradient analysis is needed.
- The abundance table has not been transformed and dominant taxa would overwhelm presence/absence-driven Raup–Crick dissimilarity — apply the cube-root transformation first.
- Input is raw sequencing reads rather than an OTU count table; metabarcoding must be completed first.

## Inputs

- Plant-species × arthropod-OTU community count table (abundance matrix from eDNA metabarcoding of flowers, e.g. 16S/COI metabarcoding results)

## Outputs

- Raup–Crick dissimilarity matrix between samples
- Average-linkage (UPGMA) sample dendrogram / heatmap (saved as figure5_cluster_dendrogram.png)

## How to apply

Load the plant-species × arthropod-OTU community count table into R and apply a cube-root (n^(1/3)) transformation to the abundance values to down-weight dominant taxa. Compute the Raup–Crick dissimilarity matrix between samples using vegan::vegdist with the raupcrick method. Build an average-linkage (UPGMA) hierarchical clustering of the dissimilarity matrix with R's hclust(method='average'), then render the clustered sample dendrogram/heatmap with the pheatmap package. Judge success by inspecting whether samples from the same plant species (in this study Tanacetum, Echium and Angelica) form the tightest clusters; the article's expectation is that communities cluster by plant species, though the corresponding finding notes segregation was only partial.

## Related tools

- **R** (Statistical environment in which the dissimilarity computation, hierarchical clustering and heatmap rendering are scripted)
- **vegan** (vegan::vegdist with the raupcrick method computes the Raup–Crick dissimilarity matrix from the transformed OTU table)
- **pheatmap** (Renders the clustered sample dendrogram/heatmap (figure5_cluster_dendrogram.png))

## Examples

```
d <- vegan::vegdist(root(otu_tab), method='raupcrick'); hc <- hclust(d, method='average'); pheatmap::pheatmap(otu_tab, clustering_distance_rows=d, clustering_method='average', filename='figure5_cluster_dendrogram.png')
```

## Evaluation signals

- The dissimilarity matrix is square, symmetric, with zero diagonal and values bounded by the Raup–Crick range.
- Tanacetum, Echium and Angelica samples form the tightest clusters in the pheatmap dendrogram.
- The abundance values fed to vegdist are cube-root transformed (n^(1/3)), not raw counts.
- hclust is called with method='average' (UPGMA) on the Raup–Crick matrix, and the resulting tree can be cross-checked against an RDA ordination for consistent, though imperfect, grouping by plant species.
- The output figure file figure5_cluster_dendrogram.png is produced and shows samples ordered by the clustering rather than raw table order.

## Limitations

- The article reports that arthropod eDNA communities were only somewhat, not perfectly, segregated by plant species, so the dendrogram should not be interpreted as perfect host-plant discrimination.
- The experiment was not exhaustive and higher arthropod richness could be obtained, meaning clustering is based on incomplete community recovery.
- eDNA traces on flowers can represent any ecological interaction (pollinators, parasitoids, gall inducers, predators, phytophagous species), not only pollination, so community grouping reflects mixed interaction types.

## Evidence

- [other] Cluster analyses (Raup–Crick, average-linkage) and RDA showed that arthropod eDNA communities were only somewhat, not perfectly, segregated by plant species.: [evidence span withheld — anchor status: not_found]
- [abstract] Arthropod communities clustered together according to plant species.: "Arthropod communities clustered together according to plant species."
- [abstract] We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI.: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [other] Apply a cube-root (n^(1/3)) transformation to the abundance values to down-weight dominant taxa.: [evidence span withheld — anchor status: not_found]
