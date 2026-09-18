---
name: mantel-test-statistics-extraction
description: Use when a metabarcoding study reports an isolation-by-distance Mantel
  test and you must capture the correlation coefficient and p-value per sample type
  (e.g., inflow vs pond water) into a structured record.
when_to_use_negative:
- Input is raw sequencing reads or a FASTQ QC pipeline step — this skill only captures
  Mantel test statistics, not read processing
- The diversity metric of interest is eukaryotic/ITS beta-diversity — the article's
  Mantel test was computed on Bray-Curtis beta-diversity of prokaryotes only
- No geographic coordinate or distance information exists for the samples — the Mantel
  test requires an inter-farm distance matrix, and in this study exact farm locations
  were withheld
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: IBD
  role: Isolation-by-distance software used to run the Mantel test of geographic distance
    vs. beta-diversity
- name: XLSTAT
  role: Statistical add-in software (XLSTAT 2017) used to perform the Mantel test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:150241af4749a9d78fb7816b5127935610ed1ebdf1de2bb2f6a75460822dbe9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IBD
  - XLSTAT
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# mantel-test-statistics-extraction

## Summary

Extracts and encodes Mantel test results correlating Bray-Curtis beta-diversity (prokaryotes only) with inter-farm geographic distance, preserving per-water-source significance outcomes (pond-water r = 0.969, p < 0.0001 vs inflow I-source p > 0.05).

## When to use

Use when a metabarcoding study reports an isolation-by-distance Mantel test and you must capture the correlation coefficient and p-value per sample type (e.g., inflow vs pond water) into a structured record.

## When NOT to use

- Input is raw sequencing reads or a FASTQ QC pipeline step — this skill only captures Mantel test statistics, not read processing
- The diversity metric of interest is eukaryotic/ITS beta-diversity — the article's Mantel test was computed on Bray-Curtis beta-diversity of prokaryotes only
- No geographic coordinate or distance information exists for the samples — the Mantel test requires an inter-farm distance matrix, and in this study exact farm locations were withheld

## Inputs

- Article discussion text reporting the Mantel test of geographic distance vs. Bray-Curtis beta-diversity (prokaryotes only)
- Inter-farm geographic distance matrix and beta-diversity matrix (as computed with IBD v1.52 / XLSTAT 2017 in the original study)

## Outputs

- Structured JSON record with fields pond_water_r (0.969), pond_water_p (<0.0001), inflow_water_p (>0.05), and per-water-source significance flags (mantel_test_results)

## How to apply

Locate the reported Mantel test result in the discussion section, where the correlation between Bray-Curtis beta-diversity (computed for prokaryotes only) and inter-farm geographic distance was tested using IBD v1.52 and XLSTAT 2017. Extract both reported statistics: the pond-water correlation (r = 0.969, p < 0.0001, significant) and the inflow-water (I-source) outcome (not significant, p > 0.05). Encode both outcomes into a structured JSON record with fields pond_water_r, pond_water_p, inflow_water_p, and a per-water-source significance flag. Validate that the encoded values match the reported statistics exactly before saving, so downstream agents can rely on the record without re-reading the article.

## Related tools

- **IBD** (Isolation-by-distance software used to run the Mantel test of geographic distance vs. beta-diversity)
- **XLSTAT** (Statistical add-in software (XLSTAT 2017) used to perform the Mantel test)

## Examples

```
mantel_results = {"pond_water_r": 0.969, "pond_water_p": "<0.0001", "inflow_water_p": ">0.05", "significant": {"pond": True, "inflow": False}}
```

## Evaluation signals

- Encoded pond_water_r equals 0.969 exactly as reported
- Encoded pond_water_p records p < 0.0001 and is flagged significant
- Encoded inflow_water_p records p > 0.05 and is flagged not significant, matching the reported absence of correlation for I-source samples
- Record explicitly notes the beta-diversity input was Bray-Curtis on prokaryotes only, computed with IBD v1.52 and XLSTAT 2017
- A round-trip diff between encoded JSON values and the article's discussion text shows no discrepancies

## Limitations

- The Mantel test was restricted to prokaryotic Bray-Curtis beta-diversity; no equivalent distance-decay statistic is reported for eukaryotic/ITS communities
- Exact farm locations cannot be disclosed (withheld at farmers' request), limiting geographic reproducibility and independent recomputation of the distance matrix
- The study covered only three semi-intensive trout farms in Bavaria, so the correlation is based on very few pairwise comparisons

## Evidence

- [discussion] Mantel test performed with IBD and XLSTAT on distance vs beta-diversity: "tested using a Mantel test and performed using IBD v1.5259 and XLSTAT 2017 (Addinsoft, Paris, France) software"
- [discussion] No significant Mantel correlation for I-source samples: "Based on the Mantel test, no significant correlation was observed for the I-source samples"
- [discussion] Pond-water biodiversity more related to location than water source: "the biodiversity and microbial enrichment of eukaryotes and prokaryotes in pond-water are more related to location than the water source"
- [discussion] Exact farm locations withheld, limiting reproducibility: "The exact location of these farms cannot be disclosed based on the farmer demand."
