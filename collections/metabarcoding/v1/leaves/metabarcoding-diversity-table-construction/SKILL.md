---
name: metabarcoding-diversity-table-construction
description: Use when 16S rRNA/ITS metabarcoding results for multiple site-source
  combinations (e.g. inflow I and pond P at farms N, B, G) must be summarized into
  a machine-readable diversity table with alpha indices and beta-diversity ranks.
when_to_use_negative:
- Input is already a raw OTU count table or FASTQ data rather than reported diversity
  indices — diversity computation tools (e.g. SILVAngs output handling) are needed
  first
- The analysis targets beta-diversity statistics themselves (e.g. Mantel test of geographic
  distance vs. beta-diversity using IBD v1.52 and XLSTAT 2017) rather than tabulating
  reported values
- Samples are not trout-farm inflow/pond water and lack the N/B/G location code scheme
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3050
tools:
- name: SILVAngs
  role: Online automatic pipeline (SILVA rDNA database) whose output underlies the
    16S OTU data from which Simpson alpha-diversity is calculated
- name: IBD v1.52
  role: Used with XLSTAT 2017 for the Mantel test of geographic distance vs. beta-diversity,
    complementary to the tabulated beta ordering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:7cafce9ace714881f5734dbfd6c5006bfc910cffad2591a6e34cc0d61964a01d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IBD v1.52
  - SILVAngs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# metabarcoding-diversity-table-construction

## Summary

Assembles a per-site microbial diversity table from metabarcoding results for the six trout-farm water samples, capturing Simpson alpha-diversity values and the Bray-Curtis beta-diversity ordering (computed for prokaryotes only) as a validated CSV.

## When to use

Use when 16S rRNA/ITS metabarcoding results for multiple site-source combinations (e.g. inflow I and pond P at farms N, B, G) must be summarized into a machine-readable diversity table with alpha indices and beta-diversity ranks.

## When NOT to use

- Input is already a raw OTU count table or FASTQ data rather than reported diversity indices — diversity computation tools (e.g. SILVAngs output handling) are needed first
- The analysis targets beta-diversity statistics themselves (e.g. Mantel test of geographic distance vs. beta-diversity using IBD v1.52 and XLSTAT 2017) rather than tabulating reported values
- Samples are not trout-farm inflow/pond water and lack the N/B/G location code scheme

## Inputs

- Manuscript-reported Simpson D-index values for six site-source combinations (N-I, N-P, B-I, B-P, G-I, G-P)
- Reported Bray-Curtis beta-diversity ordering (prokaryotes only)
- Location codes (N, B, G) and water-source labels (inflow, pond)

## Outputs

- diversity_index_table.csv with 6 data rows and columns: location code, water source (inflow/pond), simpson_index, beta_diversity_rank

## How to apply

Extract one reported Simpson diversity index value per farm-location and water-source combination (N-I 0.86, N-P 0.49, B-I 0.64, B-P 0.54, G-I 0.79, G-P 0.66), noting that alpha-diversity was calculated according to the Simpson diversity index. Record the reported ordering of Bray-Curtis beta-diversity, which was computed for prokaryotes only, with G-I ranked highest and B-P lowest. Write one row per site-source combination into diversity_index_table.csv with columns for location code, water source (inflow/pond), simpson_index, and beta_diversity_rank. Validate that the CSV exists, contains exactly 6 data rows, includes the required fields, that every simpson_index lies between 0 and 1 and matches the reference values, and that the beta ordering places G-I first and B-P last.

## Related tools

- **SILVAngs** (Online automatic pipeline (SILVA rDNA database) whose output underlies the 16S OTU data from which Simpson alpha-diversity is calculated)
- **IBD v1.52** (Used with XLSTAT 2017 for the Mantel test of geographic distance vs. beta-diversity, complementary to the tabulated beta ordering)

## Examples

```
import pandas as pd; pd.DataFrame({'location':['N','N','B','B','G','G'],'water_source':['inflow','pond','inflow','pond','inflow','pond'],'simpson_index':[0.86,0.49,0.64,0.54,0.79,0.66]}).to_csv('diversity_index_table.csv', index=False)
```

## Evaluation signals

- diversity_index_table.csv exists and contains exactly 6 data rows (one per site-source combination)
- All required fields present: location code, water source (inflow/pond), simpson_index, beta_diversity_rank
- Every simpson_index value matches the reported reference values and lies between 0 and 1
- G-I is ranked highest and B-P lowest in the beta-diversity ordering
- Values match the reported set {0.86, 0.49, 0.64, 0.54, 0.79, 0.66} for N-I, N-P, B-I, B-P, G-I, G-P

## Limitations

- Exact farm locations were withheld at farmers' demand, limiting geographic reproducibility of the site codes
- Bray-Curtis beta-diversity was computed for prokaryotes only, so ranks do not describe the eukaryotic (ITS) community
- Reported indices are two-decimal values transcribed from the manuscript; recomputing from raw data may differ in precision

## Evidence

- [discussion] Alpha-diversity via Simpson index; beta-diversity via Bray-Curtis (prokaryotes only): "was calculated according to the Simpson diversity index56"
- [abstract] 16S/ITS metabarcoding on inflow- and pond-water samples: "The 16S rRNA and ITS metabarcoding analyses were applied on the inflow- and pond-water samples from trout farms"
- [discussion] Mantel test tooling for beta-diversity context: "tested using a Mantel test and performed using IBD v1.5259 and XLSTAT 2017 (Addinsoft, Paris, France) software"
