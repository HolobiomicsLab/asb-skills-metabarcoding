---
name: swarm-clustering-parameter-tuning
description: Use when inferring ASVs/MOTUs from paired-end COI (or 16S/18S/ITS) amplicon
  .fastq.gz reads with PEMA and you must pick Swarm v2 clustering parameters such
  as the d radius.
when_to_use_negative:
- You need OTU clustering via VSEARCH for COI — PEMA's COI workflow supports ASVs/Swarm
  only, not VSEARCH OTUs
- Input is raw, un-merged reads — clustering must be preceded by quality trimming,
  merging (PANDAseq) and dereplication (obiuniq)
- Your taxonomy assignment step requires a non-COI reference workflow (e.g. CREST
  LCAClassifier + SILVA for 16S/18S) — Swarm d-radius tuning here is grounded in the
  COI mock community context
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PEMA
  role: containerized pipeline that orchestrates the ASV inference step and subsequent
    chimera removal and taxonomy assignment
  repo: https://github.com/hariszaf/pema
- name: Swarm v2
  role: ASV inference algorithm; clustering parameter (d radius) is the key tuning
    knob
- name: VSEARCH
  role: invoked for chimera removal after Swarm clustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:706667882d2f8581762fedb208f615c0ee846ba68a8e6181a87227f77f45a700
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - FastQC
  - OBITools
  - PANDAseq
  - PEMA
  - RDPClassifier
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# swarm-clustering-parameter-tuning

## Summary

Tuning and running Swarm v2 ASV inference for COI metabarcoding reads within PEMA, including choice of cluster radius (d=10 for the Bista COI mock community) and deferring chimera removal to the post-clustering VSEARCH step since Swarm v2 is not affected by chimeras.

## When to use

Use when inferring ASVs/MOTUs from paired-end COI (or 16S/18S/ITS) amplicon .fastq.gz reads with PEMA and you must pick Swarm v2 clustering parameters such as the d radius.

## When NOT to use

- You need OTU clustering via VSEARCH for COI — PEMA's COI workflow supports ASVs/Swarm only, not VSEARCH OTUs
- Input is raw, un-merged reads — clustering must be preceded by quality trimming, merging (PANDAseq) and dereplication (obiuniq)
- Your taxonomy assignment step requires a non-COI reference workflow (e.g. CREST LCAClassifier + SILVA for 16S/18S) — Swarm d-radius tuning here is grounded in the COI mock community context

## Inputs

- Pre-processed paired-end COI .fastq.gz reads (quality-trimmed, merged with PANDAseq, dereplicated with obiuniq)
- PEMA parameters.tsv configured for ASV inference (Swarm) on the COI marker gene

## Outputs

- ASV/MOTU table with per-sample abundances
- Sequence cluster representatives ready for taxonomy assignment (RDPClassifier + MIDORI for COI)

## How to apply

Set the ASV inference step in PEMA's parameters.tsv to use Swarm v2, which allows ASV inference for all supported marker genes (COI in PEMA is supported only via Swarm, not VSEARCH OTUs). Run the pre-processing steps first (FastQC, Trimmomatic, BayesHammer, PANDAseq, obiuniq) so that clustered units are abundance-annotated unique sequences. For the Bista et al. COI mock community evaluation, use cluster radius d=10, a value arrived at after testing many different parameter settings, especially for quality trimming and the OTU clustering/ASV inference steps. Let PEMA handle chimera removal after clustering (via VSEARCH) rather than pre-clustering, because Swarm v2 is not affected by chimeras. Optionally remove singletons occurring after clustering, then pass the ASV table to RDPClassifier with the MIDORI reference database for taxonomy.

## Related tools

- **PEMA** (containerized pipeline that orchestrates the ASV inference step and subsequent chimera removal and taxonomy assignment) — https://github.com/hariszaf/pema
- **Swarm v2** (ASV inference algorithm; clustering parameter (d radius) is the key tuning knob)
- **VSEARCH** (invoked for chimera removal after Swarm clustering)

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; then inside the container: ./pema_latest.bds (with parameters.tsv set to Swarm ASV inference, d=10, for COI)
```

## Evaluation signals

- On the Bista et al. COI mock community, verify that 12 of the 14 mock community species are identified in the taxonomy-assigned ASV table
- F1-scores for mock community recovery at genus level should fall in the reported 74%–86% range
- Confirm chimera filtering ran post-clustering (VSEARCH step) and that no chimera step was applied before Swarm
- Check the ASV table contains per-sample abundance counts and that singletons were optionally removed as configured
- Confirm the parameters.tsv version matches the PEMA container version used (e.g. pema:v.2.1.4)

## Limitations

- Swarm-based ASV inference for COI requires substantial memory downstream: a minimum of 20 GB of RAM is required for the COI taxonomy assignment step
- Whether to use ASVs instead of OTUs should be based on each study's objective(s), so d=10 tuned on a mock community may not generalize to every study design
- No MIDORI entry existed for B. leachii in MIDORI_LONGEST_1.1, so some mock community taxa may lack reference entries and fail taxonomy assignment
- PEMA will fail unless parameters.tsv and directory naming conventions are followed exactly, and the parameters file version must match the PEMA version

## Evidence

- [intro] Swarm v2 allows ASV inference in all cases: "Two clustering algorithms, VSEARCH [17] and CROP [18], are used for the clustering of reads in (M)OTUs"
- [methods] Parameter testing scope: "Many different parameter settings were tested, especially for the steps of quality trimming of the reads and the OTU clustering/ASV inference"
- [readme] COI ASVs via Swarm only: [evidence span withheld — anchor status: not_found]
- [methods] Post-clustering chimera removal: "The VSEARCH package [17] is then invoked for chimera removal"
- [methods] Taxonomy assignment for MOTUs: "PEMA uses the RDPClassifier [25] and the MIDORI reference database [26] to assign taxonomy of the MOTUs"
