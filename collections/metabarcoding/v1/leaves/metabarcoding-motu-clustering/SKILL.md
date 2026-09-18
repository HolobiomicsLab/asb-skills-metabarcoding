---
name: metabarcoding-motu-clustering
description: Use when dereplicated, chimera-filtered metabarcoding reads (e.g. FASTA
  output of vsearch uchime-denovo) must be clustered into MOTU representative sequences
  before taxonomic assignment.
when_to_use_negative:
- Input reads are still raw or only quality/length filtered — clustering must occur
  after obiuniq dereplication and vsearch uchime-denovo chimera removal
- The goal is taxonomic assignment itself — that is handled downstream by ecotag against
  the bespoke reference database
- A sample-by-sample OTU table already exists — re-clustering would break read-count
  consistency with the published 99% threshold
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: sumaclust
  role: Clusters the dereplicated, chimera-filtered reads into MOTUs at a constant
    99% sequence similarity threshold
- name: OBITools
  role: Metabarcoding suite within which MOTU representative sequences are extracted
    and read counts aggregated per MOTU
- name: vsearch
  role: 'Upstream producer of the chimera-filtered input: uchime-denovo chimera removal
    precedes sumaclust clustering'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:7e97127d727a3de43c62cf7157f71c2f18e89d7a4e7bb7922e96679d8c020525
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - sumaclust
  - vsearch
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metabarcoding-motu-clustering

## Summary

Delimits molecular operational taxonomic units (MOTUs) from quality-filtered, dereplicated eDNA metabarcoding reads using the sumaclust algorithm at a constant 99% sequence similarity threshold, within the OBITools metabarcoding framework.

## When to use

Use when dereplicated, chimera-filtered metabarcoding reads (e.g. FASTA output of vsearch uchime-denovo) must be clustered into MOTU representative sequences before taxonomic assignment.

## When NOT to use

- Input reads are still raw or only quality/length filtered — clustering must occur after obiuniq dereplication and vsearch uchime-denovo chimera removal
- The goal is taxonomic assignment itself — that is handled downstream by ecotag against the bespoke reference database
- A sample-by-sample OTU table already exists — re-clustering would break read-count consistency with the published 99% threshold

## Inputs

- Chimera-filtered, dereplicated sequence reads (FASTA output of vsearch uchime-denovo)
- Original filtered sequences from Mendeley dataset 489gckxbz3/1 (after quality >40, length 120-135 bp filtering)

## Outputs

- MOTU representative sequences (one per delimited MOTU)
- MOTU table with representative sequence, total read count, and MOTU identifier (e.g. motus_table.csv)
- Per-MOTU aggregated read counts within the OBITools framework

## How to apply

Take the chimera-filtered, dereplicated sequence dataset (output of the vsearch uchime-denovo step; e.g. the original filtered sequences deposited at data.mendeley.com/datasets/489gckxbz3/1) as input. Run sumaclust with a fixed, constant sequence similarity threshold of 99% (threshold 0.99) to delimit MOTUs across the entire read set — do not vary the threshold per sample or marker. Extract one representative sequence per delimited MOTU and aggregate read counts per MOTU within the OBITools framework, writing the MOTU table (representative sequence, total read count, MOTU identifier), e.g. to motus_table.csv. This table then feeds directly into downstream steps: conservative removal of all single-read MOTUs as a contamination/sequencing-error control, followed by ecotag taxonomic assignment against the bespoke elasmobranch COI reference database built from BOLD plus in silico ecoPCR (EMBL-EBI R117) sequences.

## Related tools

- **sumaclust** (Clusters the dereplicated, chimera-filtered reads into MOTUs at a constant 99% sequence similarity threshold)
- **OBITools** (Metabarcoding suite within which MOTU representative sequences are extracted and read counts aggregated per MOTU)
- **vsearch** (Upstream producer of the chimera-filtered input: uchime-denovo chimera removal precedes sumaclust clustering)

## Examples

```
sumaclust -t 0.99 chimera_filtered_dereplicated.fasta > motus_representatives.fasta
```

## Evaluation signals

- Output table contains exactly one representative sequence per delimited MOTU
- MOTU counts are consistent with a constant 99% similarity threshold (threshold -t 0.99) as reported in the paper
- Every input read is accounted for in a MOTU's aggregated read count within the OBITools framework
- Resulting motus_table.csv (representative sequence, total read count, MOTU identifier) loads cleanly into the subsequent single-read-MOTU removal step
- Clustering was run once over the full dereplicated dataset, not per sample, preserving cross-sample comparability

## Limitations

- All single-read MOTUs were conservatively removed afterwards because contamination or sequencing error cannot be excluded as their cause (potential false positives)
- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species (Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to multiple species in BOLD
- The constant 99% threshold was fixed for this 127 bp COI fragment; applying it to other markers or fragment lengths is untested here

## Evidence

- [discussion] sumaclust 99% threshold: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] OBITools framework: "The bioinformatic analysis was based on the OBITools metabarcoding software suite75."
- [discussion] upstream chimera removal: "a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77"
- [discussion] single-read MOTU removal: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [discussion] deposited filtered sequences: "Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at: https://data.mendeley.com/datasets/489gckxbz3/1."
