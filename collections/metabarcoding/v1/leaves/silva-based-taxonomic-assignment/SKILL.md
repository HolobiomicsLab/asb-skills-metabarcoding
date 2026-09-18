---
name: silva-based-taxonomic-assignment
description: Use when assigning genus-level (or higher) taxonomy to 16S or 18S rRNA
  OTUs/ASVs from eDNA metabarcoding data, e.g. mock-community or environmental samples
  processed by PEMA.
when_to_use_negative:
- Input is COI marker gene data — PEMA uses the RDPClassifier with the MIDORI database
  (e.g. MIDORI_LONGEST_1.1) instead of Silva, so this skill is the wrong route.
- Input is ITS (Fungi) marker gene data — PEMA assigns taxonomy using the Unite database,
  not Silva.
- Taxonomy has already been assigned and you only need downstream ecological analysis
  (phyloseq/vegan alpha- and beta-diversity, PERMANOVA).
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: PEMA
  role: Containerized pipeline that invokes the taxonomic assignment step (BDS-implemented;
    parameters.tsv selects classifier and Silva version)
  repo: https://github.com/hariszaf/pema
- name: CREST LCAClassifier
  role: Alignment-based classifier used with the Silva database for 16S/18S taxonomy
    assignment
- name: Silva database
  role: Reference database (versions 128 and 132 bundled in PEMA) against which 16S/18S
    sequences are classified
- name: EPA-ng
  role: Used with RaxML-ng to build the SILVA_132_SSURef reference tree of 1000 taxa
    for phylogeny-based 16S assignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:db12fbce3149ba867f97ca2bcc1051b066e0ba72ed381bb379c57abd36c57b3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - EPA-ng
  - FastQC
  - OBITools
  - PANDAseq
  - PEMA
  - Silva
  - Silva database
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# silva-based-taxonomic-assignment

## Summary

Assign taxonomy to 16S/18S rRNA amplicon units (OTUs or ASVs) using the CREST LCAClassifier against the Silva reference database (versions 128 or 132), with an optional phylogeny-based route via a SILVA_132_SSURef reference tree. This is PEMA's alignment-based taxonomic assignment step for ribosomal marker genes.

## When to use

Use when assigning genus-level (or higher) taxonomy to 16S or 18S rRNA OTUs/ASVs from eDNA metabarcoding data, e.g. mock-community or environmental samples processed by PEMA.

## When NOT to use

- Input is COI marker gene data — PEMA uses the RDPClassifier with the MIDORI database (e.g. MIDORI_LONGEST_1.1) instead of Silva, so this skill is the wrong route.
- Input is ITS (Fungi) marker gene data — PEMA assigns taxonomy using the Unite database, not Silva.
- Taxonomy has already been assigned and you only need downstream ecological analysis (phyloseq/vegan alpha- and beta-diversity, PERMANOVA).

## Inputs

- OTU or ASV representative sequences (FASTA) from VSEARCH/CROP clustering or Swarm v2 inference
- OTU/ASV abundance table per sample
- PEMA parameters.tsv configured for the 16S or 18S marker gene with Silva version 128 or 132
- SILVA_132_SSURef-based reference tree (for the phylogeny-based 16S route)

## Outputs

- OTU- or ASV-table with taxonomic assignments (e.g. genus-level labels) and per-sample abundances
- Taxa counts identified to genus and species level per run (for benchmarking)

## How to apply

After OTU clustering (VSEARCH/CROP) or ASV inference (Swarm v2), run taxonomic assignment in PEMA's alignment-based mode: the CREST LCAClassifier classifies the representative sequences against the Silva database bundled with PEMA (Silva 128, dated 29 September 2016, or Silva 132, dated 13 December 2017). For 16S rRNA data specifically, a phylogeny-based alternative is available: PEMA ships a reference tree of 1000 taxa built from SILVA_132_SSURef using EPA-ng and RaxML-ng. Set the marker-gene and classifier choices in the parameters.tsv file of the analysis directory, and validate the result against known community composition — in the Gohl et al. 16S mock community benchmark (20 bacterial species), correct application recovers 18 of 20 taxa to genus level with precision/recall ~80% and F1-scores in the 74%–86% range. Note that for COI and ITS, Silva is the wrong reference: PEMA uses the RDPClassifier with MIDORI for COI and the Unite database for ITS.

## Related tools

- **PEMA** (Containerized pipeline that invokes the taxonomic assignment step (BDS-implemented; parameters.tsv selects classifier and Silva version)) — https://github.com/hariszaf/pema
- **CREST LCAClassifier** (Alignment-based classifier used with the Silva database for 16S/18S taxonomy assignment)
- **Silva database** (Reference database (versions 128 and 132 bundled in PEMA) against which 16S/18S sequences are classified)
- **EPA-ng** (Used with RaxML-ng to build the SILVA_132_SSURef reference tree of 1000 taxa for phylogeny-based 16S assignment)

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema
```

## Evaluation signals

- On the Gohl et al. 16S mock community (20 bacterial species), the Swarm v2 (d=3) run recovers 18 of 20 taxa to genus level.
- Per-condition precision = TP/(TP+FP) and recall = TP/(TP+FN) reach ~80% or above, with F1-scores falling in the reported 74%–86% range.
- Number of taxa assigned to genus and species level is recorded for each run and compared against known mock community composition.
- The Silva version used (128 or 132) matches the PEMA/parameters.tsv version consistency requirement.
- VSEARCH-based OTU runs show high precision (>0.79) but lower recall (0.65–0.68), serving as a cross-condition sanity check.

## Limitations

- Only Silva versions 128 (29 September 2016) and 132 (13 December 2017) are included in PEMA; no newer releases are bundled.
- Phylogeny-based assignment is available only for 16S rRNA marker gene data, not for 18S, ITS, or COI.
- Reference coverage gaps cause missed assignments — e.g. for B. leachii no entry exists in the MIDORI database, illustrating that reference databases limit assignment completeness.
- The parameters.tsv file must match the PEMA version being used, or the run will fail.

## Evidence

- [intro] CREST with Silva for 16S/18S: "Taxonomic assignment is performed in an alignment-based approach, making use of the CREST LCAClassifier [20] and the Silva database [21]"
- [methods] Silva versions bundled in PEMA: "Two versions of Silva are included in PEMA: 128 (29 September 2016) and 132 (13 December 2017)"
- [readme] Phylogeny-based 16S route: [evidence span withheld — anchor status: not_found]
- [methods] F1-score benchmark range: "Overall the F1-scores ranged from 74% to 86%"
- [methods] Marker-gene specific reference databases: "is used together with the Silva [21] and the Unite [22] database, respectively, to assign taxonomy to the OTUs"
