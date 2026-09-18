---
name: community-composition-heatmap-visualization
description: Use when a sample-by-taxon abundance/count table (e.g. flower samples
  x arthropod OTUs from eDNA metabarcoding) must be compared across sample groups
  to visualize whether communities cluster by the source species.
when_to_use_negative:
- Input is a raw sequencing read file (FASTQ) rather than a processed OTU/count table
  — metabarcoding bioinformatics must be done first
- The analysis goal requires formal hypothesis testing of group differences (e.g.
  RDA, PERMANOVA) rather than a visual dendrogram/heatmap
- Count data should be compared with abundance-sensitive indices such as Bray-Curtis
  when dominant-taxa weighting is desired rather than down-weighted by cube-root transformation
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: environment in which the count table is transformed, the dissimilarity matrix
    is computed, and clustering is performed
  repo: tobiasgf/Bioinformatic-tools
- name: vegan
  role: provides vegan::vegdist with the raupcrick method to compute the Raup-Crick
    dissimilarity matrix
  repo: tobiasgf/Bioinformatic-tools
- name: pheatmap
  role: renders the clustered sample dendrogram/heatmap and saves it as figure5_cluster_dendrogram.png
  repo: tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:3236c69ccdc610e7ae091e5c8c9df34b77dba5642d71a8ab7928e98ec60ae755
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - R
  - pheatmap
  - vegan
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# community-composition-heatmap-visualization

## Summary

A visualization skill that turns a plant-species x arthropod-OTU count table into a Raup-Crick dissimilarity matrix, clusters samples by average linkage (UPGMA), and renders the clustered dendrogram as a pheatmap heatmap to assess whether arthropod eDNA communities segregate by plant species.

## When to use

Use when a sample-by-taxon abundance/count table (e.g. flower samples x arthropod OTUs from eDNA metabarcoding) must be compared across sample groups to visualize whether communities cluster by the source species.

## When NOT to use

- Input is a raw sequencing read file (FASTQ) rather than a processed OTU/count table — metabarcoding bioinformatics must be done first
- The analysis goal requires formal hypothesis testing of group differences (e.g. RDA, PERMANOVA) rather than a visual dendrogram/heatmap
- Count data should be compared with abundance-sensitive indices such as Bray-Curtis when dominant-taxa weighting is desired rather than down-weighted by cube-root transformation

## Inputs

- plant-species x arthropod-OTU community count table from eDNA metabarcoding of flower samples (mitochondrial 16S rRNA and COI primer sets)
- sample metadata mapping samples to source plant species (e.g. Tanacetum, Echium, Angelica)

## Outputs

- clustered sample dendrogram/heatmap image (figure5_cluster_dendrogram.png)
- Raup-Crick dissimilarity matrix between samples
- hclust average-linkage (UPGMA) clustering object

## How to apply

Load the plant-species x arthropod-OTU community count table into R and apply a cube-root (n^(1/3)) transformation to the abundance values so dominant taxa do not swamp the dissimilarity calculation. Compute pairwise Raup-Crick dissimilarities between flower samples using vegan::vegdist(method='raupcrick'), which is appropriate for presence/absence-weighted incidence structure in the OTU table. Cluster the dissimilarity matrix with hclust(method='average') — i.e. average-linkage/UPGMA — so that cluster heights reflect mean pairwise dissimilarity. Render the clustered dendrogram/heatmap with pheatmap and save it (here as figure5_cluster_dendrogram.png), checking visually that samples from the same plant species (Tanacetum, Echium, Angelica) form the tightest clusters, consistent with the paper's finding that communities were only somewhat, not perfectly, segregated by plant species.

## Related tools

- **R** (environment in which the count table is transformed, the dissimilarity matrix is computed, and clustering is performed) — tobiasgf/Bioinformatic-tools
- **vegan** (provides vegan::vegdist with the raupcrick method to compute the Raup-Crick dissimilarity matrix) — tobiasgf/Bioinformatic-tools
- **pheatmap** (renders the clustered sample dendrogram/heatmap and saves it as figure5_cluster_dendrogram.png) — tobiasgf/Bioinformatic-tools

## Examples

```
hcl <- hclust(vegan::vegdist(counts^(1/3), method='raupcrick'), method='average'); pheatmap(as.matrix(hcl$height), filename='figure5_cluster_dendrogram.png')
```

## Evaluation signals

- The dissimilarity matrix is square, symmetric, with zero diagonal, one entry per flower sample
- Dendrogram is built with hclust(method='average') on the Raup-Crick matrix, not on raw counts
- Samples from the same plant species (Tanacetum, Echium, Angelica) appear as the tightest clusters, matching the paper's reported segregation pattern
- Output heatmap/dendrogram file figure5_cluster_dendrogram.png is produced and renders correctly
- Cube-root transformation is applied to abundances before dissimilarity computation, not after

## Limitations

- The paper found arthropod eDNA communities were only somewhat, not perfectly, segregated by plant species, so clustering by plant species is expected to be partial
- The experiment was not exhaustive and higher arthropod richness could be obtained with this eDNA approach, so community completeness may affect cluster resolution

## Evidence

- [methods] workflow cube-root step: [evidence span withheld — anchor status: not_found]
- [methods] workflow raup-crick step: [evidence span withheld — anchor status: not_found]
- [methods] workflow clustering step: [evidence span withheld — anchor status: not_found]
- [methods] workflow pheatmap step: [evidence span withheld — anchor status: not_found]
- [abstract] abstract clustering finding: "Arthropod communities clustered together according to plant species."
- [abstract] abstract metabarcoding input: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
