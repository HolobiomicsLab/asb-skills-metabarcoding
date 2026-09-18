---
name: bray-curtis-beta-diversity-ordering
description: Use when reproducing, validating, or tabulating reported Simpson alpha-diversity
  values and the Bray-Curtis beta-diversity ranking for paired inflow/pond-water samples
  across farm locations N, B, and G.
when_to_use_negative:
- Input is already an OTU/feature table from which you need to recompute dissimilarities
  de novo rather than validate reported values
- The analysis targets eukaryotic (ITS) communities alone — the reported Bray-Curtis
  beta-diversity ordering was computed for prokaryotes only
- Sample set is not the six N/B/G inflow-pond combinations from this trout farming
  study; the reference Simpson values would not apply
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: IBD v1.52
  role: Used with XLSTAT 2017 to test geographic distance vs. beta-diversity via Mantel
    test, contextualizing the diversity ordering
- name: XLSTAT 2017
  role: Statistical software used to run the Mantel test correlating geographic distance
    with Bray-Curtis beta-diversity
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:426a0902fbfab017870b81f00c07eba9729b2e94f1af9d698bd2cbdd7e35bcc5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IBD v1.52
  - XLSTAT 2017
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# bray-curtis-beta-diversity-ordering

## Summary

Reproduce and validate the reported alpha-diversity (Simpson index) values and the Bray-Curtis beta-diversity ordering of inflow- (I) and pond-water (P) samples from three semi-intensive Bavarian trout farms (N, B, G), as described in the 16S/ITS metabarcoding study of trout aquaculture microbiota.

## When to use

Use when reproducing, validating, or tabulating reported Simpson alpha-diversity values and the Bray-Curtis beta-diversity ranking for paired inflow/pond-water samples across farm locations N, B, and G.

## When NOT to use

- Input is already an OTU/feature table from which you need to recompute dissimilarities de novo rather than validate reported values
- The analysis targets eukaryotic (ITS) communities alone — the reported Bray-Curtis beta-diversity ordering was computed for prokaryotes only
- Sample set is not the six N/B/G inflow-pond combinations from this trout farming study; the reference Simpson values would not apply

## Inputs

- Manuscript text (Scientific Reports DOI 10.1038/s41598-020-80236-x) reporting Simpson and Bray-Curtis diversity per site-source combination
- Sample metadata: six combinations of farm location (N, B, G) x water source (inflow I, pond P)
- Prokaryotic (16S) OTU community data underlying the Bray-Curtis computation

## Outputs

- diversity_index_table.csv with 6 data rows and columns: location, water_source, simpson_index, beta_diversity_rank
- Beta-diversity ordering statement: G-I highest, B-P lowest
- Optional Mantel test outcome (no significant correlation for I-source samples)

## How to apply

Extract the six reported Simpson diversity index values from the manuscript, one per farm-location and water-source combination: N-I 0.86, N-P 0.49, B-I 0.64, B-P 0.54, G-I 0.79, G-P 0.66. Record the reported ordering of the Bray-Curtis beta-diversity (computed for prokaryotes only), with G-I ranked highest and B-P ranked lowest. Assemble a CSV (diversity_index_table.csv) with columns location code, water source (inflow/pond), simpson_index, and beta_diversity_rank, one row per site-source combination. Optionally connect the ordering to geography via the Mantel test, noting the article found no significant correlation for the I-source samples using IBD v1.52 and XLSTAT 2017. Validate that every simpson_index lies between 0 and 1 and matches the reference values, and that the ranking respects the G-I highest / B-P lowest ordering.

## Related tools

- **IBD v1.52** (Used with XLSTAT 2017 to test geographic distance vs. beta-diversity via Mantel test, contextualizing the diversity ordering)
- **XLSTAT 2017** (Statistical software used to run the Mantel test correlating geographic distance with Bray-Curtis beta-diversity)

## Examples

```
python -c "import pandas as pd; d=pd.read_csv('diversity_index_table.csv'); print(d.sort_values('simpson_index', ascending=False))"
```

## Evaluation signals

- diversity_index_table.csv exists and contains exactly 6 data rows (one per site-source combination)
- Every simpson_index value is between 0 and 1 and matches the reported reference values (N-I 0.86, N-P 0.49, B-I 0.64, B-P 0.54, G-I 0.79, G-P 0.66)
- Beta-diversity ranking has G-I as the highest and B-P as the lowest entry
- Required fields (location, water_source, simpson_index, beta_diversity_rank) are present and non-empty in every row
- Any Mantel test recapitulates the reported non-significant correlation for I-source samples

## Limitations

- The exact farm locations cannot be disclosed based on farmer demand, limiting geographic reproducibility of the diversity ordering
- Bray-Curtis beta-diversity was computed for prokaryotes only, so eukaryotic (ITS) communities cannot be validated against this ordering
- No significant Mantel correlation was observed for the I-source samples, so the beta-diversity ordering should not be over-interpreted as purely distance-driven; the article notes pond-water biodiversity is more related to location than water source

## Evidence

- [discussion] alpha/beta diversity calculation method: "was calculated according to the Simpson diversity index56"
- [discussion] Mantel test result for I-source samples: "Based on the Mantel test, no significant correlation was observed for the I-source samples"
- [discussion] location vs water source drivers of pond biodiversity: "the biodiversity and microbial enrichment of eukaryotes and prokaryotes in pond-water are more related to location than the water source"
- [abstract] scope of the metabarcoding sampling design: "The 16S rRNA and ITS metabarcoding analyses were applied on the inflow- and pond-water samples from trout farms"
