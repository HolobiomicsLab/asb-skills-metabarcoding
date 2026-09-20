---
name: aquaculture-microbial-biogeography
description: Use when you have pairwise geographic distances between spatially separated
  aquaculture farms (e.g., three Bavarian trout farms with inflow- and pond-water
  samples) and must test whether microbial beta-diversity or characteristic-species
  counts correlate with distance.
when_to_use_negative:
- Input is raw FASTQ or an OTU abundance table — the Mantel test consumes square pairwise
  distance matrices (beta-diversity and geographic), not reads or count tables.
- All samples come from a single farm or a single water source, so no inter-farm distance-vs-distance
  correlation is testable.
- The target is eukaryotic (ITS) isolation-by-distance — the published Bray-Curtis
  Mantel analysis was computed on prokaryotes only.
tools:
- name: IBD v1.52
  role: Runs the isolation-by-distance Mantel test correlating the inter-farm geographic
    distance matrix with the beta-diversity matrix
- name: XLSTAT 2017
  role: Addinsoft statistical software used together with IBD v1.52 to perform the
    Mantel test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:fa45cfaa26b5e24483abfc0b5270f863aaa1101150444b1400ece417026b498a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IBD
  - IBD v1.52
  - XLSTAT
  - XLSTAT 2017
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# aquaculture-microbial-biogeography

## Summary

Tests isolation-by-distance in trout-farm microbiomes by Mantel-correlating inter-farm geographic distances with Bray-Curtis beta-diversity (prokaryotes only) or per-farm characteristic-species counts, run separately for inflow- and pond-water samples with IBD v1.52 and XLSTAT 2017.

## When to use

Use when you have pairwise geographic distances between spatially separated aquaculture farms (e.g., three Bavarian trout farms with inflow- and pond-water samples) and must test whether microbial beta-diversity or characteristic-species counts correlate with distance.

## When NOT to use

- Input is raw FASTQ or an OTU abundance table — the Mantel test consumes square pairwise distance matrices (beta-diversity and geographic), not reads or count tables.
- All samples come from a single farm or a single water source, so no inter-farm distance-vs-distance correlation is testable.
- The target is eukaryotic (ITS) isolation-by-distance — the published Bray-Curtis Mantel analysis was computed on prokaryotes only.

## Inputs

- Bray-Curtis beta-diversity distance matrix computed from prokaryotic (16S) OTU abundances (SILVAngs; cd-hit-est v3.1.2 clustering at identity criteria 1.00 and 0.98)
- Inter-farm pairwise geographic distance matrix among the three semi-intensive trout farms
- Per-sample water-source labels separating inflow (I-source) from pond-water samples
- Per-farm characteristic-species counts (for the species-count variant of the Mantel correlation)

## Outputs

- Mantel correlation statistics per water source: pond-water r = 0.969, p < 0.0001 (significant); inflow I-source p > 0.05 (not significant)
- Structured JSON record (mantel_test_results) with fields pond_water_r, pond_water_p, inflow_water_p and per-water-source significance flags
- Biogeographic verdict: pond-water biodiversity is more related to location than to the water source

## How to apply

Build two square distance matrices over the same ordered samples: (a) Bray-Curtis beta-diversity computed from prokaryotic (16S) OTU abundances (SILVAngs pipeline, cd-hit-est v3.1.2 clustering at identity criteria 1.00 and 0.98) and (b) pairwise inter-farm geographic distances among the three Bavarian trout farms. Run the Mantel test with IBD v1.52 and XLSTAT 2017 (Addinsoft, Paris, France), once per water source — inflow (I-source) samples and pond-water samples — never pooled. Decision point: a pond-water Mantel r = 0.969 with p < 0.0001 alongside a non-significant inflow result (p > 0.05) reproduces the article's conclusion that pond-water biodiversity is more related to location than to the water source. Encode both outcomes into a JSON record with fields pond_water_r, pond_water_p, inflow_water_p plus a per-water-source significance flag, and validate each value against the published statistic exactly before saving as mantel_test_results. The species-count variant of the correlation replaces the Bray-Curtis matrix with per-farm characteristic-species counts while keeping the same geographic distance matrix.

## Related tools

- **IBD v1.52** (Runs the isolation-by-distance Mantel test correlating the inter-farm geographic distance matrix with the beta-diversity matrix)
- **XLSTAT 2017** (Addinsoft statistical software used together with IBD v1.52 to perform the Mantel test)

## Examples

```
python -c "import ; .dump({'pond_water_r':0.969,'pond_water_p':'<0.0001','inflow_water_p':'>0.05','pond_water_sig':True,'inflow_sig':False},open('mantel_test_results.','w'))"
```

## Evaluation signals

- Encoded record matches the reported statistics exactly: pond_water_r = 0.969, pond_water_p '< 0.0001' flagged significant, inflow_water_p '> 0.05' flagged non-significant.
- Both matrices passed to the Mantel test are square, symmetric, zero-diagonal, and share identical sample order; with three farms each holds only 3 unique pairwise distances.
- The per-water-source contrast is preserved in the output: pond-water significant, I-source not — pooling inflow and pond samples would erase the reported pattern.
- mantel_test_results. parses as JSON and contains all required fields (pond_water_r, pond_water_p, inflow_water_p, per-source significance flags) with correct types.
- Re-running the Mantel correlation on the same distance matrices in IBD v1.52 or XLSTAT 2017 reproduces the recorded r and p within numerical tolerance.

## Limitations

- Exact farm locations were withheld at the farmers' demand, so inter-farm geographic distances cannot be independently recomputed from the publication — a geographic-reproducibility gap.
- Only three semi-intensive farms were sampled, so each Mantel test rests on very few pairwise distances and the near-perfect r = 0.969 should be read as a small-n, upper-bound result.
- The Bray-Curtis beta-diversity used in the Mantel test covers prokaryotes only (16S); the isolation-by-distance conclusion is not demonstrated for the eukaryotic (ITS) community.

## Evidence

- [discussion] Mantel test performed with IBD and XLSTAT: "tested using a Mantel test and performed using IBD v1.5259 and XLSTAT 2017 (Addinsoft, Paris, France) software"
- [discussion] No significant Mantel correlation for inflow (I-source) samples: "Based on the Mantel test, no significant correlation was observed for the I-source samples"
- [discussion] Pond-water biodiversity tracks location over water source: "the biodiversity and microbial enrichment of eukaryotes and prokaryotes in pond-water are more related to location than the water source"
- [discussion] Farm locations withheld, limiting geographic reproducibility: "The exact location of these farms cannot be disclosed based on the farmer demand."
- [intro] Three-farm, two-locus sampling design: "profile both prokaryotic (16S) and eukaryotic (ITS) microbial community of three different semi-intensive freshwater trout farms"
- [abstract] Inflow- and pond-water sample strata: "The 16S rRNA and ITS metabarcoding analyses were applied on the inflow- and pond-water samples from trout farms"
