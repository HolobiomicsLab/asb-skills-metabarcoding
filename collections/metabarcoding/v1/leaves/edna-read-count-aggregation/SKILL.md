---
name: edna-read-count-aggregation
description: Use when a dereplicated, chimera-filtered eDNA read set must be clustered
  into MOTUs and per-MOTU read abundances aggregated before taxonomic assignment.
when_to_use_negative:
- Input is already a MOTU or feature table with per-sample counts rather than dereplicated
  reads
- The study design requires species-level resolution beyond the 99% MOTU clustering
  of the short 127 bp COI fragment — closely related species (Carcharhinus, Rhizoprionodon,
  Negaprion) remain unresolved
- Reads have not yet passed the upstream quality (score >40), length (120–135 bp)
  and chimera-removal filters
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3168
tools:
- name: sumaclust
  role: Clusters dereplicated reads into MOTUs at a constant 99% similarity threshold
- name: OBITools
  role: Framework in which per-MOTU read counts are aggregated and the MOTU table
    is produced
- name: obiuniq
  role: Dereplicates identical sequences upstream, aggregating per-sample read counts
    into the count annotation consumed by clustering
- name: ecotag
  role: Consumes the aggregated MOTU representative sequences for downstream taxonomic
    assignment
merged_aliases:
- fastq-read-count-aggregation
merged_alias_records:
- alias: fastq-read-count-aggregation
  slug: fastq-read-count-aggregation
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:f0dd7b173ee2e9e8a76f51374492e9afb2cf01a4c5069de89ecd9123bf66d5fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - ecotag
  - obiuniq
  - sumaclust
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# edna-read-count-aggregation

## Summary

Aggregates eDNA metabarcoding read counts per MOTU after sumaclust clustering (99% constant similarity threshold) within the OBITools framework, producing a MOTU table of representative sequences, total read counts and MOTU identifiers used for downstream ecotag taxonomic assignment.

## When to use

Use when a dereplicated, chimera-filtered eDNA read set must be clustered into MOTUs and per-MOTU read abundances aggregated before taxonomic assignment.

## When NOT to use

- Input is already a MOTU or feature table with per-sample counts rather than dereplicated reads
- The study design requires species-level resolution beyond the 99% MOTU clustering of the short 127 bp COI fragment — closely related species (Carcharhinus, Rhizoprionodon, Negaprion) remain unresolved
- Reads have not yet passed the upstream quality (score >40), length (120–135 bp) and chimera-removal filters

## Inputs

- Chimera-filtered, dereplicated FASTQ/FASTA reads (output of obiuniq dereplication and vsearch uchime-denovo chimera removal)
- OBITools count annotations aggregated per unique sequence
- Original filtered sequences from the Mendeley dataset 489gckxbz3/1

## Outputs

- MOTU representative-sequence table (representative sequence, total read count, MOTU identifier) written to motus_table.csv
- MOTU abundance table feeding into single-read-MOTU removal and ecotag taxonomic assignment

## How to apply

Start from the dereplicated reads produced by obiuniq from reads that were aligned with illuminapairedend (quality >40), demultiplexed with ngsfilter, stripped of ambiguous bases and length-filtered with obigrep to 120–135 bp, and from which chimeras were removed using vsearch uchime-denovo (data deposited at data.mendeley.com/datasets/489gckxbz3/1). Run sumaclust with a fixed, constant sequence similarity threshold of 99% to delimit MOTUs, rather than sample-dependent or variable thresholds. Extract one representative sequence per MOTU and aggregate the read counts per MOTU within the OBITools framework (read counts were previously collapsed into the count annotation by obiuniq). Write the MOTU table (representative sequence, total read count, MOTU identifier) to motus_table.csv. Follow the article's conservative approach of removing all single-read MOTUs before ecotag taxonomic assignment to limit contamination/sequencing-error false positives.

## Related tools

- **sumaclust** (Clusters dereplicated reads into MOTUs at a constant 99% similarity threshold)
- **OBITools** (Framework in which per-MOTU read counts are aggregated and the MOTU table is produced)
- **obiuniq** (Dereplicates identical sequences upstream, aggregating per-sample read counts into the count annotation consumed by clustering)
- **ecotag** (Consumes the aggregated MOTU representative sequences for downstream taxonomic assignment)

## Examples

```
sumaclust -t 0.99 dereplicated_sequences.fastq > motus.fasta && obitools-export motus.fasta > motus_table.csv
```

## Evaluation signals

- The output table contains exactly one representative sequence per delimited MOTU
- MOTU counts are consistent with a fixed (constant) 99% similarity threshold rather than a variable threshold
- Per-MOTU read counts equal the sum of the aggregated count annotations of member reads from the obiuniq dereplicated file
- Single-read MOTUs are removed downstream as a conservative contamination/false-positive control
- The motus_table.csv schema carries the three expected columns: representative sequence, total read count, and MOTU identifier

## Limitations

- The short 127 bp COI fragment means some MOTUs are 100% identical to multiple species in BOLD, limiting taxonomic resolution after count aggregation
- Contamination or sequencing error cannot be excluded as the cause of single-read MOTUs, which motivates the conservative removal step
- Read abundance and community biomass relationships were not empirically verified for elasmobranchs, so aggregated counts must be interpreted cautiously quantitatively
- Primer bias and species-specific shedding rates can distort relative read abundances across MOTUs

## Evidence

- [discussion] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] The reads were subsequently dereplicated using obiuniq: "The reads were subsequently dereplicated using obiuniq"
- [discussion] we have opted for a more conservative approach and have removed all single read MOTUs from our samples.: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [discussion] Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75.: "Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75."
- [discussion] Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at the Mendeley repository.: "Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at: https://data.mendeley.com/datasets/489gckxbz3/1"
