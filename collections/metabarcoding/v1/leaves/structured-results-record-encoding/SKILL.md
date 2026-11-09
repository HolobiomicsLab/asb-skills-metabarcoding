---
name: structured-results-record-encoding
description: Use when a published statistical result (r, p thresholds per sample group)
  must be captured as a machine-readable record, e.g., Mantel test outcomes for pond-water
  vs inflow-water trout farm samples.
when_to_use_negative:
- Input is already a machine-readable results file (e.g., raw IBD/XLSTAT output tables)
  — parse it directly instead of hand-encoding
- The article reports exact p-values (not inequality bounds) and a defined numeric
  schema exists — use the native numeric type rather than string thresholds
- The claim of interest is a qualitative finding without any reported statistics (e.g.,
  'biodiversity more related to location than water source') — encode as a finding
  statement, not a statistical record
tools:
- name: IBD
  role: Source of the reported isolation-by-distance Mantel test statistics that are
    encoded into the structured record
- name: XLSTAT
  role: Software (XLSTAT 2017, Addinsoft, Paris, France) that produced the reported
    Mantel test result being encoded
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:c470018e5b7197a697be8604c70c9a2622b229ae57a0b3eeb9a50b0d17efcce5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IBD
  - XLSTAT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# structured-results-record-encoding

## Summary

Encodes reported statistical results (e.g., Mantel test correlations between Bray-Curtis beta-diversity of prokaryotes and inter-farm geographic distance) into a validated structured JSON record with explicit fields and per-condition significance flags.

## When to use

Use when a published statistical result (r, p thresholds per sample group) must be captured as a machine-readable record, e.g., Mantel test outcomes for pond-water vs inflow-water trout farm samples.

## When NOT to use

- Input is already a machine-readable results file (e.g., raw IBD/XLSTAT output tables) — parse it directly instead of hand-encoding
- The article reports exact p-values (not inequality bounds) and a defined numeric schema exists — use the native numeric type rather than string thresholds
- The claim of interest is a qualitative finding without any reported statistics (e.g., 'biodiversity more related to location than water source') — encode as a finding statement, not a statistical record

## Inputs

- Published article text (discussion section) reporting statistical test results
- Reported test statistics: correlation coefficient (r) and significance threshold (p-value bounds) per sample group
- Named software versions used for the test (e.g., IBD v1.52, XLSTAT 2017)

## Outputs

- Structured JSON record with fields: pond_water_r (0.969), pond_water_p ('<0.0001'), inflow_water_p ('>0.05'), and per-water-source significance flags
- Saved artifact, e.g., mantel_test_results.

## How to apply

Locate the reported statistic in the article discussion section — here the Mantel test of geographic distance vs Bray-Curtis beta-diversity (prokaryotes only), computed with IBD v1.52 and XLSTAT 2017. Extract each reported value exactly: pond-water r = 0.969 with p < 0.0001 (significant), and inflow-water (I-source) not significant at p > 0.05. Encode these into a JSON object with explicit fields such as pond_water_r, pond_water_p, inflow_water_p, plus a per-water-source significance boolean. As a decision point, encode threshold-style p-values as strings ('<0.0001', '>0.05') rather than floats when only inequality bounds are reported, to avoid fabricating precision. Finally, validate by re-checking the encoded values against the source text field-by-field before saving (e.g., to mantel_test_results.).

## Related tools

- **IBD** (Source of the reported isolation-by-distance Mantel test statistics that are encoded into the structured record)
- **XLSTAT** (Software (XLSTAT 2017, Addinsoft, Paris, France) that produced the reported Mantel test result being encoded)

## Examples

```
import ; rec = {"pond_water_r": 0.969, "pond_water_p": "<0.0001", "inflow_water_p": ">0.05", "pond_water_significant": True, "inflow_water_significant": False}; .dump(rec, open("mantel_test_results.", "w"))
```

## Evaluation signals

- Encoded pond_water_r equals the reported value exactly (0.969) with no rounding drift
- pond_water_p is stored as the inequality bound '<0.0001' (not a fabricated float), matching the article
- inflow_water_p is stored as '>0.05' and its significance flag is false, while pond-water's flag is true
- Field-by-field diff of the JSON against the source text passes before saving to mantel_test_results
- The record notes the test provenance: Mantel test of Bray-Curtis beta-diversity (prokaryotes only) vs inter-farm geographic distance

## Limitations

- Only inequality bounds on p-values are reported in the article, so exact p-values cannot be encoded without over-claiming precision
- Exact farm locations were withheld at farmers' request, limiting geographic reproducibility of the underlying correlation
- The correlation is based on prokaryotic (16S) Bray-Curtis beta-diversity only; eukaryotic ITS results are not covered by this record

## Evidence

- [discussion] mantel inflow non-significant: "Based on the Mantel test, no significant correlation was observed for the I-source samples"
- [discussion] mantel test software: "tested using a Mantel test and performed using IBD v1.5259 and XLSTAT 2017 (Addinsoft, Paris, France) software"
- [discussion] location vs water source: "the biodiversity and microbial enrichment of eukaryotes and prokaryotes in pond-water are more related to location than the water source"
