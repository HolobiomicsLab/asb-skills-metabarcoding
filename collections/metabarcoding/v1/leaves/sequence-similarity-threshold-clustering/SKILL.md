---
name: sequence-similarity-threshold-clustering
description: Use when processed (dereplicated, chimera-removed) amplicon reads must
  be grouped into MOTUs at a fixed similarity threshold before taxonomic assignment,
  as in OBITools-based eDNA metabarcoding pipelines.
when_to_use_negative:
- The input is raw, unaligned, or not yet demultiplexed/length-filtered/dereplicated
  reads — clustering must come after obigrep length filtering (120–135 bp), ambiguous-base
  removal, obiuniq dereplication, and vsearch uchime-denovo chimera removal.
- You need assignment to a reference database rather than de novo MOTU delimitation
  — taxonomic assignment is a separate later step performed with ecotag.
- The goal is species-level distinction of MOTUs that are 100% identical to multiple
  BOLD reference species (e.g. some Carcharhinus, Rhizoprionodon, Negaprion MOTUs)
  — the 127 bp COI fragment's resolution limits cannot be fixed by re-clustering.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: sumaclust
  role: Clustering algorithm that delimits MOTUs from the processed reads at a constant
    99% similarity threshold and selects representative sequences
- name: OBITools
  role: Metabarcoding software suite within which per-MOTU read counts are aggregated
    and the MOTU table is written (motus_table.csv)
- name: vsearch
  role: Provides the uchime-denovo chimera-removal step whose output is the clustering
    input
- name: ecotag
  role: Consumes the MOTU representative sequences for downstream taxonomic assignment
    against the bespoke elasmobranch reference database
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:5b20727949f1108fdc040f337e6b4c387f4191c5faf30b32400e04bce4b33243
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - ecotag
  - sumaclust
  - vsearch
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sequence-similarity-threshold-clustering

## Summary

Delimits Molecular Operational Taxonomic Units (MOTUs) from chimera-filtered, dereplicated eDNA metabarcoding reads using the sumaclust clustering algorithm at a constant 99% sequence similarity threshold, producing representative sequences and per-MOTU read counts for downstream taxonomic assignment with ecotag within the OBITools framework.

## When to use

Use when processed (dereplicated, chimera-removed) amplicon reads must be grouped into MOTUs at a fixed similarity threshold before taxonomic assignment, as in OBITools-based eDNA metabarcoding pipelines.

## When NOT to use

- The input is raw, unaligned, or not yet demultiplexed/length-filtered/dereplicated reads — clustering must come after obigrep length filtering (120–135 bp), ambiguous-base removal, obiuniq dereplication, and vsearch uchime-denovo chimera removal.
- You need assignment to a reference database rather than de novo MOTU delimitation — taxonomic assignment is a separate later step performed with ecotag.
- The goal is species-level distinction of MOTUs that are 100% identical to multiple BOLD reference species (e.g. some Carcharhinus, Rhizoprionodon, Negaprion MOTUs) — the 127 bp COI fragment's resolution limits cannot be fixed by re-clustering.

## Inputs

- Chimera-filtered, dereplicated FASTA sequence dataset (output of vsearch uchime-denovo on obiuniq-dereplicated reads)
- Original filtered sequences from the deposited Mendeley dataset (data.mendeley.com/datasets/489gckxbz3/1) if rebuilding the pipeline

## Outputs

- MOTU representative-sequence table (motus_table.csv) with columns: representative sequence, total read count, MOTU identifier
- One representative sequence per delimited MOTU, ready for single-read-MOTU removal and ecotag taxonomic assignment

## How to apply

Take the output of the vsearch uchime-denovo chimera-removal step applied to the dereplicated reads (produced earlier with illuminapairedend alignment, ngsfilter demultiplexing, obigrep length filtering to 120–135 bp, and obiuniq dereplication) and run sumaclust with a fixed similarity threshold of 99%. Sumaclust groups the dereplicated reads into MOTUs based on sequence similarity and designates a representative sequence per MOTU. Within the OBITools framework, aggregate read counts per MOTU and write the MOTU table (representative sequence, total read count, MOTU identifier) to motus_table.csv. The rationale for the 99% constant threshold is that it preserves fine-scale discrimination of shark COI MOTUs from the 127 bp elasmobranch COI amplicon while merging sequencing-error variants; the resulting MOTU table feeds directly into single-read-MOTU removal and ecotag taxonomic assignment.

## Related tools

- **sumaclust** (Clustering algorithm that delimits MOTUs from the processed reads at a constant 99% similarity threshold and selects representative sequences)
- **OBITools** (Metabarcoding software suite within which per-MOTU read counts are aggregated and the MOTU table is written (motus_table.csv))
- **vsearch** (Provides the uchime-denovo chimera-removal step whose output is the clustering input)
- **ecotag** (Consumes the MOTU representative sequences for downstream taxonomic assignment against the bespoke elasmobranch reference database)

## Examples

```
sumaclust -t 0.99 chimera_filtered_dereplicated.fasta > motus_representatives.fasta
```

## Evaluation signals

- The output motus_table.csv contains exactly one representative sequence per delimited MOTU.
- MOTU counts are consistent with clustering at a 99% constant similarity threshold as reported in the paper (a stricter threshold would yield more, finer-grained MOTUs; a looser threshold fewer).
- The total number of reads summed over MOTUs matches the input chimera-filtered, dereplicated read count.
- The table columns match the expected schema: representative sequence, total read count, MOTU identifier.
- The MOTU table feeds without modification into the subsequent single-read-MOTU removal step and the ecotag taxonomic assignment step.

## Limitations

- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species: some MOTUs are 100% identical to multiple species in BOLD, including species from different ocean regions (e.g. Carcharhinus, Rhizoprionodon, Negaprion), and clustering cannot resolve this.
- Single-read MOTUs may arise from contamination or sequencing errors (potential false positives), which is why all single-read MOTUs are removed conservatively after clustering.
- The constant 99% threshold is a fixed choice from the paper; it is not tuned per marker length, so applying it to other amplicons or gene regions may require re-justification.

## Evidence

- [discussion] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] The reads were subsequently dereplicated using obiuniq: "The reads were subsequently dereplicated using obiuniq"
- [discussion] a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77: "a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77"
- [discussion] Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75.: "Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75."
- [discussion] we have opted for a more conservative approach and have removed all single read MOTUs from our samples.: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
