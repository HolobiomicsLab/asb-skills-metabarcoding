---
name: precision-recall-metric-computation
description: Use when evaluating an eDNA metabarcoding pipeline on a mock community
  with a known taxonomic ground truth and you need quantitative precision/recall/F1
  per clustering or ASV-inference condition.
when_to_use_negative:
- The input is an environmental (not mock) community sample with no known ground-truth
  taxon composition, so TP/FP/FN cannot be defined
- Taxonomy was assigned only to level above genus, since the PEMA benchmark computed
  these metrics at genus level
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3305
tools:
- name: PEMA
  role: pipeline whose mock-community outputs (OTU/ASV tables with taxonomy) are scored
    with precision/recall/F1
  repo: https://github.com/hariszaf/pema
- name: Swarm v2
  role: ASV inference condition being benchmarked (d=3 and d=30 settings)
- name: VSEARCH
  role: OTU clustering condition being benchmarked; also invoked for chimera removal
- name: CREST LCAClassifier
  role: phylogeny-based taxonomy assignment against Silva used to produce genus-level
    labels for scoring
- name: Silva 132
  role: reference database (versions 128/132) for genus-level taxonomic assignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:4be94ee87a3db4c20829f82bd44768e796aced4709037dd57775bd7c7fd0e61d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - FastQC
  - OBITools
  - PANDAseq
  - PEMA
  - Silva 132
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# precision-recall-metric-computation

## Summary

Benchmark a metabarcoding pipeline against a mock community by computing per-condition precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 = 2*(precision*recall)/(precision+recall) at genus level, as PEMA did for the Gohl et al. 16S mock community.

## When to use

Use when evaluating an eDNA metabarcoding pipeline on a mock community with a known taxonomic ground truth and you need quantitative precision/recall/F1 per clustering or ASV-inference condition.

## When NOT to use

- The input is an environmental (not mock) community sample with no known ground-truth taxon composition, so TP/FP/FN cannot be defined
- Taxonomy was assigned only to level above genus, since the PEMA benchmark computed these metrics at genus level

## Inputs

- OTU/ASV table with taxonomic assignments at genus level (from CREST LCAClassifier + Silva 128/132)
- Mock community ground-truth taxon list (e.g., 20 bacterial species from the Gohl et al. 16S mock community)
- Per-condition run outputs (Swarm v2 d=3, Swarm v2 d=30, VSEARCH OTU clustering)

## Outputs

- Per-condition precision, recall, and F1 scores
- Counts of taxa identified to genus and species level per run

## How to apply

After running each pipeline condition (e.g., PEMA with Swarm v2 d=3 for ASV inference, Swarm v2 d=30 for the KAPA samples, or VSEARCH OTU clustering), assign taxonomy with the CREST LCAClassifier against Silva 128/132 at genus level, then compare the recovered taxa list against the mock community's expected composition (20 bacterial species for the Gohl et al. dataset). Treat expected-and-recovered taxa as true positives (TP), unexpected reported taxa as false positives (FP), and expected-but-missed taxa as false negatives (FN). Compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 = 2*(precision*recall)/(precision+recall) for each condition, and record the number of taxa identified to genus and species level per run. For the PEMA paper this yielded F1-scores ranging from 74% to 86%, with the Swarm v2 (d=3) run recovering 18 of 20 taxa to genus level.

## Related tools

- **PEMA** (pipeline whose mock-community outputs (OTU/ASV tables with taxonomy) are scored with precision/recall/F1) — https://github.com/hariszaf/pema
- **Swarm v2** (ASV inference condition being benchmarked (d=3 and d=30 settings))
- **VSEARCH** (OTU clustering condition being benchmarked; also invoked for chimera removal)
- **CREST LCAClassifier** (phylogeny-based taxonomy assignment against Silva used to produce genus-level labels for scoring)
- **Silva 132** (reference database (versions 128/132) for genus-level taxonomic assignment)

## Evaluation signals

- Swarm v2 (d=3) run recovers 18 of 20 taxa to genus level on the Gohl et al. mock community
- Per-condition precision and recall are ~80% or above
- F1-scores fall within the reported 74%-86% range
- VSEARCH OTU condition shows high precision (>0.79) but recall of 0.65-0.68
- Number of taxa identified to genus and species level is recorded per run for comparison across conditions

## Limitations

- Metrics depend on the Silva database version used (128 vs 132) and on taxonomy assignment being done at genus level
- Chimera removal must be performed after ASV inference when using Swarm v2, since Swarm output clouds are not affected by chimeras; ordering affects the scored taxon list
- Whether to remove singletons after (M)OTU clustering or ASV inference affects the taxon set being scored

## Evidence

- [methods] PEMA F1-scores for mock community recovery at genus level ranged from 74% to 86%: "Overall the F1-scores ranged from 74% to 86%"
- [abstract] PEMA evaluated against mock communities with comparable quality results: "PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality."
- [intro] Taxonomic assignment approach used for scoring inputs: "Taxonomic assignment is performed in an alignment-based approach, making use of the CREST LCAClassifier [20] and the Silva database [21]"
- [methods] Two Silva versions included in PEMA: "Two versions of Silva are included in PEMA: 128 (29 September 2016) and 132 (13 December 2017)"
- [intro] Swarm v2 enables ASV inference for all marker genes: "Swarm v2 [19] allows ASV inference in all cases"
