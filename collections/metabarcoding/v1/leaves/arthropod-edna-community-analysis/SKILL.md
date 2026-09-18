---
name: arthropod-edna-community-analysis
description: Use when a plant-species x arthropod-OTU count table from flower eDNA
  metabarcoding needs hierarchical clustering (e.g. pheatmap/UPGMA) or RDA to test
  whether community composition tracks the sampled plant species.
when_to_use_negative:
- Input is raw sequencing data (reads/FASTQ) rather than an already-built OTU count
  table — metabarcoding and OTU calling must happen first
- The analysis goal is taxonomic richness estimation rather than community-composition
  clustering/segregation by plant species
edam_operation: http://edamontology.org/operation_0291
tools:
- name: vegan
  role: computes the Raup–Crick dissimilarity matrix (vegdist, raupcrick method) between
    flower-eDNA samples
- name: pheatmap
  role: renders the average-linkage clustered sample dendrogram/heatmap as the final
    figure
- name: R
  role: host environment for the cube-root transformation, clustering, and RDA of
    the OTU table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:9fd330d798b5c11bbdb2b511ea3c8decc9e49ec38b2229907da8bb6808b2ac0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pheatmap
  - vegan
  edam_operation: http://edamontology.org/operation_0291
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# arthropod-edna-community-analysis

## Summary

Clustering and ordination analysis of arthropod eDNA communities recovered from wild-flower samples (16S rRNA and COI metabarcoding), testing whether communities segregate by plant species using Raup–Crick dissimilarity, average-linkage UPGMA clustering, and RDA.

## When to use

Use when a plant-species x arthropod-OTU count table from flower eDNA metabarcoding needs hierarchical clustering (e.g. pheatmap/UPGMA) or RDA to test whether community composition tracks the sampled plant species.

## When NOT to use

- Input is raw sequencing data (reads/FASTQ) rather than an already-built OTU count table — metabarcoding and OTU calling must happen first
- The analysis goal is taxonomic richness estimation rather than community-composition clustering/segregation by plant species

## Inputs

- plant-species x arthropod-OTU community abundance count table (from flower eDNA metabarcoding with 16S rRNA and COI primer sets)

## Outputs

- Raup–Crick dissimilarity matrix between samples
- average-linkage (UPGMA) sample dendrogram / clustered heatmap (e.g. figure5_cluster_dendrogram.png)
- assessment of whether communities cluster by plant species (Tanacetum, Echium, Angelica as tightest clusters)

## How to apply

Load the plant-species x arthropod-OTU community count table into R and apply a cube-root (n^(1/3)) transformation to abundance values to down-weight dominant taxa. Compute the Raup–Crick dissimilarity matrix between samples with vegan::vegdist(method='raupcrick'). Build an average-linkage (UPGMA) hierarchical clustering of that matrix with hclust(method='average') and render the clustered dendrogram/heatmap with the pheatmap package. Optionally cross-check with RDA to confirm that arthropod eDNA communities are only somewhat, not perfectly, segregated by plant species, checking that Tanacetum, Echium and Angelica form the tightest clusters. Save the rendered clustering as a figure such as figure5_cluster_dendrogram.png.

## Related tools

- **vegan** (computes the Raup–Crick dissimilarity matrix (vegdist, raupcrick method) between flower-eDNA samples)
- **pheatmap** (renders the average-linkage clustered sample dendrogram/heatmap as the final figure)
- **R** (host environment for the cube-root transformation, clustering, and RDA of the OTU table)

## Examples

```
d <- vegan::vegdist(otu_table^(1/3), method='raupcrick'); hc <- hclust(d, method='average'); pheatmap(hc, filename='figure5_cluster_dendrogram.png')
```

## Evaluation signals

- The dissimilarity matrix is computed after cube-root transformation, not on raw counts — verify the transformation was applied to abundances before vegdist
- The dendrogram is built with average-linkage (UPGMA) on a Raup–Crick (raupcrick) distance, not on Euclidean or Bray–Curtis distances
- Samples from the same plant species group together, with Tanacetum, Echium and Angelica forming the tightest clusters
- Segregation by plant species should be only partial ('somewhat, not perfectly' segregated), consistent with the article's finding; perfect separation suggests a pipeline error or overfitting
- Output heatmap/dendrogram file (e.g. figure5_cluster_dendrogram.png) renders successfully from the hclust object via pheatmap

## Limitations

- Arthropod eDNA communities were only somewhat, not perfectly, segregated by plant species — do not expect clean species-level separation of samples
- The experiment was not exhaustive; an even higher arthropod richness could be obtained with this eDNA approach, so the OTU table may under-represent true community diversity
- Cube-root transformation down-weights dominant taxa, so results will differ from analyses on raw abundance counts

## Evidence

- [abstract] primer targets: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] clustering finding: "Arthropod communities clustered together according to plant species."
- [other] dissimilarity computation: [evidence span withheld — anchor status: not_found]
- [other] transformation rationale: [evidence span withheld — anchor status: not_found]
- [other] degree of segregation: [evidence span withheld — anchor status: not_found]
