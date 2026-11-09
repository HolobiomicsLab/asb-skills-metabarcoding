---
name: coi-taxonomy-assignment-against-midori
description: Use when a PEMA COI metabarcoding run has produced an ASV/MOTU table
  (e.g., from Swarm v2 with d=10) and species/genus-level taxonomic labels must be
  attached to the sequences.
when_to_use_negative:
- Marker gene is 16S/18S rRNA or ITS — PEMA uses CREST LCAClassifier with SILVA, or
  RDPClassifier/UNITE for ITS, not the COI MIDORI workflow
- You need VSEARCH-based OTU clustering for COI — PEMA's COI path supports ASV inference
  only (Swarm), not OTUs/VSEARCH
- The input is already a taxonomy-assigned ASV table (e.g., from Barque with BOLD)
  — re-running RDPClassifier/MIDORI would duplicate the step
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3174
tools:
- name: PEMA
  role: Pipeline that orchestrates the RDPClassifier + MIDORI taxonomy assignment
    step for COI MOTUs/ASVs
  repo: https://github.com/hariszaf/pema
- name: RDPClassifier
  role: Classifier invoked by PEMA to assign taxonomy to COI MOTUs/ASVs
- name: MIDORI
  role: Reference database of quality-controlled metazoan mitochondrial gene sequences
    from GenBank used for COI taxonomic assignment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:b35972d08d36e6e9a08345746c92ce21bf6c02b51a93723bfb866919ca8beef0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - FastQC
  - MIDORI
  - OBITools
  - PANDAseq
  - PEMA
  - RDPClassifier
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# coi-taxonomy-assignment-against-midori

## Summary

Assign taxonomy to COI MOTUs/ASVs produced by PEMA using the RDPClassifier against the MIDORI reference database of quality-controlled metazoan mitochondrial COI sequences from GenBank, yielding a taxonomy-annotated ASV table.

## When to use

Use when a PEMA COI metabarcoding run has produced an ASV/MOTU table (e.g., from Swarm v2 with d=10) and species/genus-level taxonomic labels must be attached to the sequences.

## When NOT to use

- Marker gene is 16S/18S rRNA or ITS — PEMA uses CREST LCAClassifier with SILVA, or RDPClassifier/UNITE for ITS, not the COI MIDORI workflow
- You need VSEARCH-based OTU clustering for COI — PEMA's COI path supports ASV inference only (Swarm), not OTUs/VSEARCH
- The input is already a taxonomy-assigned ASV table (e.g., from Barque with BOLD) — re-running RDPClassifier/MIDORI would duplicate the step

## Inputs

- COI ASV/MOTU table with abundances per sample (output of PEMA's Swarm v2 ASV inference step)
- MIDORI reference database (e.g., MIDORI_LONGEST_1.1), bundled with PEMA
- PEMA parameters.tsv configured for the COI marker gene

## Outputs

- ASV/MOTU table with RDPClassifier-assigned taxonomies and abundances per sample
- Statistics reported by PEMA for the run

## How to apply

Run PEMA's taxonomy assignment step on the COI ASV table produced after Swarm v2 ASV inference: PEMA invokes the RDPClassifier with the MIDORI reference database (e.g., version MIDORI_LONGEST_1.1) to label each MOTU/ASV. Note that for COI, PEMA supports only ASV inference (Swarm), not VSEARCH OTU clustering, so the input table will always be ASV-based. Because MIDORI is a large database, allocate at least 20 GB of RAM for the COI taxonomy assignment step. Check assignments against known mock community composition — in the Bista et al. mock community evaluation, 12 of 14 species were recovered, but taxa absent from MIDORI (e.g., B. leachii in MIDORI_LONGEST_1.1) cannot be assigned.

## Related tools

- **PEMA** (Pipeline that orchestrates the RDPClassifier + MIDORI taxonomy assignment step for COI MOTUs/ASVs) — https://github.com/hariszaf/pema
- **RDPClassifier** (Classifier invoked by PEMA to assign taxonomy to COI MOTUs/ASVs)
- **MIDORI** (Reference database of quality-controlled metazoan mitochondrial gene sequences from GenBank used for COI taxonomic assignment)

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- Mock community validation: expected recovery of 12 of the 14 Bista et al. mock community species in the taxonomy-assigned table
- Reported F1-scores for mock community recovery at genus level fall in the 74–86% range
- Every ASV row in the output table carries a MIDORI-derived taxonomy string and per-sample abundance counts
- Taxonomy assignment completes only with sufficient memory — verify the run did not fail for lack of the ~20 GB RAM required for COI
- Unassigned taxa should correspond to species genuinely absent from the MIDORI version used (e.g., B. leachii in MIDORI_LONGEST_1.1), not pipeline failures

## Limitations

- Taxa without an entry in MIDORI (e.g., B. leachii in MIDORI_LONGEST_1.1) cannot be assigned taxonomy
- MIDORI contains only metazoan mitochondrial gene sequences, so non-metazoan COI amplicons fall outside its reference scope
- For COI, PEMA requires a minimum of 20 GB of RAM for the taxonomy assignment step, which limits small-PC runs
- PEMA supports only ASV inference (Swarm), not OTU/VSEARCH clustering, for the COI marker gene

## Evidence

- [methods] PEMA uses the RDPClassifier [25] and the MIDORI reference database [26] to assign taxonomy of the MOTUs: "PEMA uses the RDPClassifier [25] and the MIDORI reference database [26] to assign taxonomy of the MOTUs"
- [intro] For the COI marker gene, the RDPClassifier and MIDORI database are used for taxonomic assignment: "For the COI marker gene, the RDPClassifier [25] and the MIDORI database [26] are used for the taxonomic assignment"
- [readme] For COI, a minimum of 20 GB of RAM for the taxonomy assignment step is required.: [evidence span withheld — anchor status: not_found]
- [methods] The MIDORI database contains quality-controlled metazoan mitochondrial gene sequences from GenBank: "The MIDORI database contains quality-controlled metazoan mitochondrial gene sequences from GenBank [37]"
- [methods] For B. leachii no entry exists in the MIDORI database, version MIDORI_LONGEST_1.1: "For B. leachii no entry exists in the MIDORI database, version MIDORI_LONGEST_1.1"
