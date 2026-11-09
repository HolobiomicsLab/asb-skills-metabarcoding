---
name: location-wise-shared-species-accounting
description: Use when a paper reports species/OTU occurrence only as Venn-diagram
  segment counts across sampling sites or sources, and a machine-readable occurrence
  table must be reconstructed and validated.
when_to_use_negative:
- The raw OTU/species abundance table or Venn source data is already available digitally
  — then extract counts directly instead of reconstructing them
- Relative-abundance or read-count data is needed rather than integer species-presence
  counts, since this skill reproduces occurrence (species) numbers only
- The paper reports continuous diversity metrics (e.g., Simpson index, Bray-Curtis
  distances) rather than discrete species segment counts
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0621
tools:
- name: SILVAngs
  role: Online SILVA rDNA-based pipeline whose 16S analysis delimited the bacterial
    species counts being reconstructed
- name: cd-hit-est v3.1.2
  role: Clustered reads into OTUs at identity criteria 1.00 and 0.98, defining the
    species units counted per location and source
- name: manual expert review
  role: 'The only workflow tool listed for this task: reading Venn segments from the
    article and transcribing/validating the counts'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:ef4838e8ebf4dd97f44ff0444dea19fea140a25167384715104901dec569f691
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SILVAngs
  - cd-hit-est v3.1.2
  - manual expert review
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# location-wise-shared-species-accounting

## Summary

Reconstructs a tidy species-occurrence table from published Venn-diagram counts, splitting bacterial species into inflow (I-source) and pond (P-source) per location (B, N, G) plus unique and shared segments. This lets downstream agents reuse species-level occurrence numbers without access to raw OTU tables.

## When to use

Use when a paper reports species/OTU occurrence only as Venn-diagram segment counts across sampling sites or sources, and a machine-readable occurrence table must be reconstructed and validated.

## When NOT to use

- The raw OTU/species abundance table or Venn source data is already available digitally — then extract counts directly instead of reconstructing them
- Relative-abundance or read-count data is needed rather than integer species-presence counts, since this skill reproduces occurrence (species) numbers only
- The paper reports continuous diversity metrics (e.g., Simpson index, Bray-Curtis distances) rather than discrete species segment counts

## Inputs

- Published Venn-diagram species counts per source type and location (e.g., B-I 774, N-I 669, G-I 548, B-P 882, N-P 553, G-P 415)
- Reported source-level totals (I-sources = 1074, P-sources = 1006 species)
- Species delimited from 16S rRNA metabarcoding of inflow- and pond-water samples from the three trout farms (B, N, G)

## Outputs

- venn_species_occurrence_counts.csv — tidy table with columns: source type (inflow/pond), location (B/N/G), integer species count
- Per-location totals and unique vs. shared (pairwise) species counts derived from Venn segments

## How to apply

Retrieve the reported Venn-diagram segment counts for each source type and location (here: I-sources total 1074 species with B-I 774, N-I 669, G-I 548; P-sources total 1006 with B-P 882, N-P 553, G-P 415). Compute per-location totals by combining the corresponding I- and P-source counts, correcting for the shared/overlap segments shown in the diagrams. Derive unique (single-location) species counts and reported shared-species counts for each location pair from the Venn segments. Assemble everything into a tidy table with columns for source type (inflow/pond), location, and integer species count, then validate that reconstructed source totals reconcile to the published totals (1074 for I-sources, 1006 for P-sources) and that per-location sums match the reported segments.

## Related tools

- **SILVAngs** (Online SILVA rDNA-based pipeline whose 16S analysis delimited the bacterial species counts being reconstructed)
- **cd-hit-est v3.1.2** (Clustered reads into OTUs at identity criteria 1.00 and 0.98, defining the species units counted per location and source)
- **manual expert review** (The only workflow tool listed for this task: reading Venn segments from the article and transcribing/validating the counts)

## Examples

```
import pandas as pd; pd.DataFrame({'source':['inflow','pond'],'location':['B','B'],'species_count':[774,882]}).to_csv('venn_species_occurrence_counts.csv', index=False)
```

## Evaluation signals

- Reconstructed I-source species total equals the reported 1074; P-source total equals the reported 1006
- Per-location I+P combinations (B, N, G) sum consistently with the reported Venn-diagram segments
- Every species count in the output table is a non-negative integer
- Output CSV contains exactly the columns source type (inflow/pond), location, and integer species count
- Unique (single-location) plus pairwise-shared counts per source reconcile with the per-location marginal totals

## Limitations

- Exact farm locations cannot be disclosed at the farmers' request, so reconstructed counts cannot be tied to verifiable geographic coordinates
- Only segment totals reported in the figures can be recovered; the underlying per-species list is not reconstructible from the article
- Counts pertain to species delimited from 16S metabarcoding; eukaryotic (ITS) OTUs are outside this accounting

## Evidence

- [results] I-source highest counts: [evidence span withheld — anchor status: not_found]
- [discussion] I vs P related to location: "the biodiversity and microbial enrichment of eukaryotes and prokaryotes in pond-water are more related to location than the water source"
- [discussion] sample provenance: "Samples were collected at five cm below the water surface using individual 500 ml sterile disposable plastic bottles"
- [discussion] OTU clustering criteria: "ignoring overhangs and applying identity criteria of 1.00 and 0.98, respectively"
