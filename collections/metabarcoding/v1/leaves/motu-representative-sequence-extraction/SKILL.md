---
name: motu-representative-sequence-extraction
description: Use when processed (dereplicated, chimera-removed) amplicon reads must
  be grouped into MOTUs at a fixed similarity threshold to obtain representative sequences
  for taxonomic assignment.
when_to_use_negative:
- Input reads still contain chimeras or have not been dereplicated — the article applies
  this clustering only after uchime-denovo chimera removal and obiuniq dereplication.
- The goal is OTU clustering at coarser resolution for general eukaryotic metabarcoding
  — this skill fixes a constant 99% threshold tuned to a 127 bp elasmobranch COI fragment.
- Input is already a MOTU/feature table — representative sequences have already been
  extracted and clustering should not be re-run.
edam_operation: http://edamontology.org/operation_0436
tools:
- name: sumaclust
  role: clusters the dereplicated reads into MOTUs at a constant 99% similarity threshold
- name: OBITools
  role: framework used to aggregate read counts per MOTU and build the MOTU representative-sequence
    table
- name: ecotag
  role: performs taxonomic assignment of the representative sequences for each MOTU
    produced by this step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:77f6ce12534708132ad0f3c8b2a35e1c76d77f7f393c357011003ad281710a58
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - ecotag
  - sumaclust
  edam_operation: http://edamontology.org/operation_0436
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# motu-representative-sequence-extraction

## Summary

Delimits Molecular Operational Taxonomic Units (MOTUs) from chimera-filtered, dereplicated eDNA reads using sumaclust at a constant 99% sequence similarity threshold, and extracts the representative sequence and read count for each MOTU as input to downstream taxonomic assignment.

## When to use

Use when processed (dereplicated, chimera-removed) amplicon reads must be grouped into MOTUs at a fixed similarity threshold to obtain representative sequences for taxonomic assignment.

## When NOT to use

- Input reads still contain chimeras or have not been dereplicated — the article applies this clustering only after uchime-denovo chimera removal and obiuniq dereplication.
- The goal is OTU clustering at coarser resolution for general eukaryotic metabarcoding — this skill fixes a constant 99% threshold tuned to a 127 bp elasmobranch COI fragment.
- Input is already a MOTU/feature table — representative sequences have already been extracted and clustering should not be re-run.

## Inputs

- chimera-filtered, dereplicated sequence reads (FASTA output of vsearch uchime-denovo; original filtered sequences deposited at data.mendeley.com/datasets/489gckxbz3/1)
- read-count metadata carried through the OBITools framework (e.g. from obiuniq dereplication)

## Outputs

- MOTU table (e.g. motus_table.csv) with one representative sequence, total read count, and MOTU identifier per delimited MOTU
- MOTU representative sequences formatted for downstream single-read-MOTU removal and ecotag taxonomic assignment

## How to apply

Load the chimera-filtered, dereplicated sequence dataset (in this workflow, the output of vsearch's uchime-denovo step applied after obiuniq dereplication of OBITools-filtered reads). Run sumaclust with a fixed (constant) similarity threshold of 99% to cluster the dereplicated reads into MOTUs. For each delimited MOTU, extract the representative sequence and aggregate the total read count per MOTU within the OBITools framework, writing the MOTU table (representative sequence, total read count, MOTU identifier). This MOTU table then feeds the conservative single-read-MOTU removal step and ecotag taxonomic assignment. The rationale for a constant 99% threshold is that the 127 bp elasmobranch COI amplicon requires fine-grained clustering to preserve species-level discrimination while collapsing sequencing-error variants.

## Related tools

- **sumaclust** (clusters the dereplicated reads into MOTUs at a constant 99% similarity threshold)
- **OBITools** (framework used to aggregate read counts per MOTU and build the MOTU representative-sequence table)
- **ecotag** (performs taxonomic assignment of the representative sequences for each MOTU produced by this step)

## Examples

```
sumaclust -t 0.99 chimera_filtered_dereplicated.fasta > motus.fasta
```

## Evaluation signals

- The output MOTU table contains exactly one representative sequence per delimited MOTU.
- MOTU counts are consistent with clustering at a constant 99% similarity threshold, as reported in the paper.
- Every representative sequence carries an associated total read count and MOTU identifier, and the table feeds directly into the subsequent single-read-MOTU removal and ecotag assignment steps.
- The input dataset matches the state expected by the workflow: reads have passed the >40 quality filter, the 120–135 bp length filter, ambiguous-base removal, and uchime-denovo chimera removal.
- Taxonomic assignment (ecotag) runs successfully on the representative sequences against the bespoke elasmobranch COI reference database built from BOLD.

## Limitations

- The 127 bp COI fragment gives insufficient taxonomic resolution to separate some closely related species (Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to multiple species in BOLD.
- Single-read MOTUs cannot be fully distinguished from contamination or sequencing error, motivating the conservative removal of all single-read MOTUs after clustering.
- Species whose sequences are 100% identical across ocean regions cannot be assigned unambiguously at this threshold/fragment length.

## Evidence

- [discussion] MOTU clustering at a constant 99% similarity threshold: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] Representative sequences feed ecotag assignment: "Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75."
- [discussion] Downstream single-read-MOTU removal after clustering: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
