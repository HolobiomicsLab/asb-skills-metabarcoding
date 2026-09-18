---
name: last-common-ancestor-taxonomy-assignment
description: Use when dereplicated ASV/OTU sequences from 16S, 18S, ITS, or COI amplicon
  data need reference-based taxonomic labels before downstream diversity or mock-community
  benchmarking.
when_to_use_negative:
- Input is raw .fastq.gz reads — taxonomy assignment must come after pre-processing,
  chimera removal, and clustering/ASV inference
- COI MOTUs where the target taxa may be absent from MIDORI (e.g. no entry for B.
  leachii in MIDORI_LONGEST_1.1)
- Analyses requiring a phylogeny-based placement that goes beyond the alignment-based
  LCA approach — for 16S rRNA only, prefer PEMA's phylogeny-based assignment with
  SILVA_132_SSURef, EPA-ng and RaxML-ng
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CREST LCAClassifier
  role: Performs alignment-based last-common-ancestor taxonomy assignment of ASVs/OTUs
    against Silva for 16S/18S
- name: Silva database
  role: Reference database (versions 128 and 132 included in PEMA) for 16S and 18S
    rRNA taxonomic assignment
- name: RDPClassifier
  role: Assigns taxonomy of COI MOTUs against the MIDORI reference database
- name: MIDORI
  role: Quality-controlled metazoan mitochondrial COI reference sequences from GenBank
    used for COI taxonomic assignment
- name: PEMA
  role: Containerized pipeline orchestrating the taxonomy assignment step within the
    full metabarcoding workflow
  repo: https://github.com/hariszaf/pema
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:f557962b55dde8e1a18e8687d924cf1fa03ce5171620577b5089cca40720fb82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - FastQC
  - MIDORI
  - OBITools
  - PANDAseq
  - PEMA
  - RDPClassifier
  - Silva database
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# last-common-ancestor-taxonomy-assignment

## Summary

Assign taxonomy to ASV/OTU sequences from eDNA metabarcoding using the CREST LCAClassifier against the Silva reference database for 16S/18S rRNA marker genes, with Unite for ITS and MIDORI/RDPClassifier for COI.

## When to use

Use when dereplicated ASV/OTU sequences from 16S, 18S, ITS, or COI amplicon data need reference-based taxonomic labels before downstream diversity or mock-community benchmarking.

## When NOT to use

- Input is raw .fastq.gz reads — taxonomy assignment must come after pre-processing, chimera removal, and clustering/ASV inference
- COI MOTUs where the target taxa may be absent from MIDORI (e.g. no entry for B. leachii in MIDORI_LONGEST_1.1)
- Analyses requiring a phylogeny-based placement that goes beyond the alignment-based LCA approach — for 16S rRNA only, prefer PEMA's phylogeny-based assignment with SILVA_132_SSURef, EPA-ng and RaxML-ng

## Inputs

- Dereplicated FASTA of ASV/OTU sequences with abundances (post obiuniq, chimera removal, and Swarm/VSEARCH clustering)
- Reference database: Silva 128 or Silva 132 for 16S/18S rRNA
- Unite database for ITS
- MIDORI database (e.g. MIDORI_LONGEST_1.1) for COI

## Outputs

- ASV/OTU table with taxonomic labels and per-sample abundances
- Taxonomy strings (genus/species/class resolution) used for precision, recall, and F1 benchmarking

## How to apply

After read pre-processing, chimera removal, and ASV inference (Swarm v2) or OTU clustering (VSEARCH), run taxonomic assignment in an alignment-based approach with the CREST LCAClassifier, using Silva for 16S and 18S rRNA marker gene data and the Unite database for the ITS gene. For COI, PEMA instead uses the RDPClassifier against the MIDORI reference database (e.g. MIDORI_LONGEST_1.1; note the B. leachii missing-entry caveat). Optionally remove singletons (sequences with only 1 read) after clustering/inference and before or after assignment. In the mock-community validation, assigned taxa were compared to the 12 known algal members at genus level, computing precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 = 2*(precision*recall)/(precision+recall); F1-scores ranged from 74% to 86%, with 3 of 12 members resolved to species level (Isochrysis galbana, Nannochloropsis oculata, Thalassiosira pseudonana) under Swarm v2 (d=1, strictness=0.5). For 16S rRNA, a phylogeny-based assignment alternative (SILVA_132_SSURef with EPA-ng and RaxML-ng on a 1000-taxon reference tree) is also available in PEMA.

## Related tools

- **CREST LCAClassifier** (Performs alignment-based last-common-ancestor taxonomy assignment of ASVs/OTUs against Silva for 16S/18S)
- **Silva database** (Reference database (versions 128 and 132 included in PEMA) for 16S and 18S rRNA taxonomic assignment)
- **RDPClassifier** (Assigns taxonomy of COI MOTUs against the MIDORI reference database)
- **MIDORI** (Quality-controlled metazoan mitochondrial COI reference sequences from GenBank used for COI taxonomic assignment)
- **PEMA** (Containerized pipeline orchestrating the taxonomy assignment step within the full metabarcoding workflow) — https://github.com/hariszaf/pema

## Examples

```
./pema_latest.bds  # run inside the PEMA container with the analysis directory mounted at /mnt/analysis; taxonomy assignment via CREST LCAClassifier + Silva is configured in parameters.tsv
```

## Evaluation signals

- Mock-community benchmark at genus level: precision = TP/(TP+FP), recall = TP/(TP+FN), F1 = 2*(precision*recall)/(precision+recall); reported F1-scores ranged from 74% to 86%
- Species-level recovery check: 3 of 12 mock community members identified to species level (Isochrysis galbana, Nannochloropsis oculata, Thalassiosira pseudonana) under Swarm v2 (d=1, strictness=0.5)
- False-positive check: best configuration (d=1, strictness=0.3) yields a single false positive at F1 = 0.82
- PEMA output table includes taxonomies of the taxa found and their abundances in each sample

## Limitations

- COI taxonomy assignment requires a minimum of 20 GB of RAM
- Reference databases have gaps — no entry exists for B. leachii in MIDORI database version MIDORI_LONGEST_1.1
- Alignment-based LCA assignment applies to 16S/18S and ITS; COI uses RDPClassifier instead, so the LCA path is not universal across marker genes
- Version consistency matters: the parameters file must match the PEMA version used (e.g. pema:v.2.1.4 requires the v.2.1.4 parameters.tsv)

## Evidence

- [intro] Alignment-based LCA classification approach: "Taxonomic assignment is performed in an alignment-based approach, making use of the CREST LCAClassifier [20] and the Silva database [21]"
- [intro] Marker-gene-specific reference databases: "the Unite database [22] is used for the ITS gene"
- [methods] COI uses RDPClassifier with MIDORI: "PEMA uses the RDPClassifier [25] and the MIDORI reference database [26] to assign taxonomy of the MOTUs"
- [methods] Database coverage gap caveat: "For B. leachii no entry exists in the MIDORI database, version MIDORI_LONGEST_1.1"
- [methods] Benchmark F1-score range on mock community: "Overall the F1-scores ranged from 74% to 86%"
- [readme] Resource requirement for COI assignment: [evidence span withheld — anchor status: not_found]
