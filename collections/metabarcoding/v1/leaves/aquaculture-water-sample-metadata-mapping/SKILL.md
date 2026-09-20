---
name: aquaculture-water-sample-metadata-mapping
description: Use when annotating 16S/ITS metabarcoding water samples from trout farms
  with location (N, B, G) and water-source (I/P) metadata in order to tabulate or
  verify per-sample Simpson alpha- and Bray-Curtis beta-diversity values from a publication.
when_to_use_negative:
- Input is already a per-sample OTU/ASV feature table from which Simpson and Bray-Curtis
  values can be computed directly, rather than transcribed from a publication
- Samples are not from the three semi-intensive Bavarian trout farms with inflow/pond
  pairing described in the article (other aquaculture designs)
- Exact geographic coordinates are required - the article withholds farm locations
  at the farmers' request, limiting geographic reproducibility
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:b43b17e652c52402bb309eb40d0afe7f09e847a3ce1aceedacbc264d8e2bf312
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# aquaculture-water-sample-metadata-mapping

## Summary

Maps aquaculture metabarcoding sample metadata (farm location codes N, B, G crossed with water source inflow/pond) to reported Simpson alpha-diversity values and Bray-Curtis beta-diversity ranks, producing a structured per-sample diversity table from manuscript text.

## When to use

Use when annotating 16S/ITS metabarcoding water samples from trout farms with location (N, B, G) and water-source (I/P) metadata in order to tabulate or verify per-sample Simpson alpha- and Bray-Curtis beta-diversity values from a publication.

## When NOT to use

- Input is already a per-sample OTU/ASV feature table from which Simpson and Bray-Curtis values can be computed directly, rather than transcribed from a publication
- Samples are not from the three semi-intensive Bavarian trout farms with inflow/pond pairing described in the article (other aquaculture designs)
- Exact geographic coordinates are required - the article withholds farm locations at the farmers' request, limiting geographic reproducibility

## Inputs

- Manuscript/report text stating the six Simpson diversity index values per farm-location and water-source combination (N-I 0.86, N-P 0.49, B-I 0.64, B-P 0.54, G-I 0.79, G-P 0.66)
- Reported Bray-Curtis beta-diversity ordering (prokaryotes only; G-I highest, B-P lowest)
- Sample metadata: farm location codes (N, B, G) and water source labels (inflow I / pond P)

## Outputs

- diversity_index_table.csv: 6 data rows with columns location_code, water_source (inflow/pond), simpson_index (0-1), beta_diversity_rank

## How to apply

Extract from the manuscript the six Simpson D-index values, one per farm-location and water-source combination: N-I 0.86, N-P 0.49, B-I 0.64, B-P 0.54, G-I 0.79, G-P 0.66 (alpha-diversity calculated according to the Simpson diversity index). Record the reported Bray-Curtis beta-diversity ordering (computed for prokaryotes only), with G-I ranked highest and B-P lowest. Assemble a CSV table (diversity_index_table.csv) with columns for location code, water source (inflow/pond), simpson_index, and beta_diversity_rank, one row per site-source combination (3 locations x 2 sources = 6 rows). Validate that each simpson_index is between 0 and 1 and matches the reference values, and that the rank ordering preserves G-I as highest and B-P as lowest. Interpret the table using the article's finding that pond-water biodiversity is more related to location than to water source.

## Examples

```
python -c "import pandas as pd; pd.DataFrame({'site':['N-I','N-P','B-I','B-P','G-I','G-P'],'simpson_index':[0.86,0.49,0.64,0.54,0.79,0.66]}).to_csv('diversity_index_table.csv',index=False)"
```

## Evaluation signals

- diversity_index_table.csv exists and contains exactly 6 data rows (3 locations x 2 water sources)
- Every simpson_index value matches the reported reference values (N-I 0.86, N-P 0.49, B-I 0.64, B-P 0.54, G-I 0.79, G-P 0.66), each within the range 0-1
- beta_diversity_rank ordering preserves G-I as the highest and B-P as the lowest, as reported for prokaryotes only
- All required fields (location_code, water_source, simpson_index, beta_diversity_rank) are present and populated in every row
- Cross-check the interpretation: pond-water biodiversity should track location more than water source, consistent with the article's discussion

## Limitations

- Exact farm locations cannot be disclosed based on farmer demand, so the mapping is limited to anonymized location codes (N, B, G)
- Bray-Curtis beta-diversity was computed for prokaryotes only; the ranking does not describe eukaryotic (ITS) communities
- Supplementary table 1 with sequencing output and quality stats is referenced, but the supplementary information was found to be empty, preventing per-sample read-depth validation

## Evidence

- [discussion] Simpson index used for alpha-diversity: "was calculated according to the Simpson diversity index56"
- [discussion] Pond-water biodiversity more related to location than water source: "the biodiversity and microbial enrichment of eukaryotes and prokaryotes in pond-water are more related to location than the water source"
- [discussion] Mantel test result for inflow samples: "Based on the Mantel test, no significant correlation was observed for the I-source samples"
- [abstract] Sample design: inflow and pond water at trout farms: "The 16S rRNA and ITS metabarcoding analyses were applied on the inflow- and pond-water samples from trout farms"
- [discussion] Farm locations withheld: "The exact location of these farms cannot be disclosed based on the farmer demand."
