---
name: beta-diversity-geographic-correlation
description: Use when sample sets span multiple geographic sites and you need to test
  whether community composition turnover (beta-diversity) tracks geographic distance,
  e.g. isolation-by-distance in aquaculture or similar spatial metabarcoding designs.
when_to_use_negative:
- When only inflow-water samples are available — the I-source samples showed no significant
  Mantel correlation (p > 0.05), so the isolation-by-distance signal is not expected
  there
- When beta-diversity is computed from eukaryotic (ITS) OTUs — the reported Mantel
  correlation was based on Bray-Curtis beta-diversity of prokaryotes only
tools:
- name: IBD
  role: Runs the isolation-by-distance Mantel test correlating geographic distance
    with beta-diversity
- name: XLSTAT 2017
  role: Statistical add-in (Addinsoft, Paris, France) used together with IBD to perform
    the Mantel test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:6e05042d5820754bced4f1558efc7e4e8ebd2319271664219a1e35b6f88e3e5d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IBD
  - XLSTAT
  - XLSTAT 2017
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# beta-diversity-geographic-correlation

## Summary

Tests whether Bray-Curtis beta-diversity (prokaryotic OTUs only) correlates with inter-farm geographic distance using a Mantel test, distinguishing water-source-specific patterns (inflow vs pond water).

## When to use

Use when sample sets span multiple geographic sites and you need to test whether community composition turnover (beta-diversity) tracks geographic distance, e.g. isolation-by-distance in aquaculture or similar spatial metabarcoding designs.

## When NOT to use

- When only inflow-water samples are available — the I-source samples showed no significant Mantel correlation (p > 0.05), so the isolation-by-distance signal is not expected there
- When beta-diversity is computed from eukaryotic (ITS) OTUs — the reported Mantel correlation was based on Bray-Curtis beta-diversity of prokaryotes only

## Inputs

- Pairwise inter-farm geographic distance matrix (3 semi-intensive freshwater trout farms in Bavaria)
- Bray-Curtis beta-diversity matrix computed from prokaryotic OTU abundances (prokaryotes only)
- Sample source labels distinguishing inflow-water (I-source) vs pond-water samples

## Outputs

- Mantel test r statistic and p-value per water source (pond_water_r = 0.969, pond_water_p < 0.0001, inflow_water_p > 0.05)
- Per-water-source significance flag encoded in a structured JSON record (mantel_test_results)

## How to apply

Compute pairwise inter-farm geographic distances and a Bray-Curtis dissimilarity matrix from prokaryotic OTU abundance data (16S reads clustered with cd-hit-est v3.1.2 and classified via the SILVAngs pipeline against SILVA SSU release 132). Then run a Mantel test correlating the two distance matrices with IBD v1.52 and XLSTAT 2017 (Addinsoft, Paris, France), once per water source: inflow (I-source) samples and pond-water samples. Record r and p for each stratum separately; in the source study pond-water characteristic species showed r = 0.969, p < 0.0001 (significant), while the I-source samples showed no significant correlation (p > 0.05), supporting the conclusion that pond-water biodiversity is more related to location than to water source. Encode both outcomes with per-water-source significance flags before saving.

## Related tools

- **IBD** (Runs the isolation-by-distance Mantel test correlating geographic distance with beta-diversity)
- **XLSTAT 2017** (Statistical add-in (Addinsoft, Paris, France) used together with IBD to perform the Mantel test)

## Evaluation signals

- Encoded pond-water values match the reported statistics exactly: r = 0.969 and p < 0.0001 (significant)
- Inflow-water (I-source) result is encoded as non-significant with p > 0.05
- Both distance matrices (geographic and Bray-Curtis) have identical sample sets and dimensions (3 farms) before testing
- Bray-Curtis matrix is derived from prokaryotic OTU data only, not ITS/eukaryotic data
- A per-water-source significance flag is present in the saved mantel_test_results record

## Limitations

- Exact farm locations cannot be disclosed at the farmers' request, limiting geographic reproducibility of the distance matrix
- Only three farms were sampled, so the Mantel correlation rests on very few pairwise distances
- No significant Mantel correlation was observed for the I-source samples, so inflow-water isolation-by-distance inference is not supported

## Evidence

- [discussion] Mantel test performed with IBD and XLSTAT: "tested using a Mantel test and performed using IBD v1.5259 and XLSTAT 2017 (Addinsoft, Paris, France) software"
- [discussion] No significant Mantel correlation for I-source samples: "Based on the Mantel test, no significant correlation was observed for the I-source samples"
- [discussion] Pond-water biodiversity more related to location: "the biodiversity and microbial enrichment of eukaryotes and prokaryotes in pond-water are more related to location than the water source"
- [other] Pond-water Mantel r = 0.969, p < 0.0001 (source card finding): [evidence span withheld — anchor status: not_found]
- [discussion] Exact farm locations withheld: "The exact location of these farms cannot be disclosed based on the farmer demand."
