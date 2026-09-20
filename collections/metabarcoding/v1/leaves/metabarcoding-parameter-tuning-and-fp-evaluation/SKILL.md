---
name: metabarcoding-parameter-tuning-and-fp-evaluation
description: Use when calibrating metabarcoding pipeline parameters on ITS/16S/18S/COI
  amplicon data or validating recovery of known mock-community members via false positives
  and F1-scores.
when_to_use_negative:
- Input is already an OTU/ASV feature table with taxonomies — tuning read-level trimming
  and clustering parameters would re-process raw reads unnecessarily
- The data are shotgun metagenomic rather than amplicon metabarcoding reads for 16S/18S/12S/ITS/COI
  marker genes
- There is no expected mock community composition to compare against, so false positives
  and F1-scores cannot be computed
edam_operation: http://edamontology.org/operation_0233
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: PEMA
  role: Containerized pipeline whose parameters.tsv is varied to test trimming, clustering/ASV-inference,
    and filter settings
  repo: https://github.com/hariszaf/pema
- name: Swarm v2
  role: ASV inference algorithm used for ITS with cluster radius d = 20; chimera removal
    is applied after inference since Swarm v2 is not affected by chimeras
- name: VSEARCH
  role: OTU clustering and chimera removal when the OTU route is chosen for parameter
    testing
- name: CROP
  role: Alternative unsupervised OTU clustering algorithm to compare against VSEARCH
- name: CREST LCAClassifier
  role: Alignment-based taxonomic assignment of ASVs/OTUs against Unite (ITS) or Silva
    (16S/18S)
- name: Docker
  role: Container runtime in which PEMA and its bundled tools are executed for each
    parameter setting
  repo: https://hub.docker.com/r/hariszaf/pema
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:9b4d4db6f0f4b2fc65cc5fb17845e93798de16210294f2692ce11b9d18c7f3ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - CROP
  - Cutadapt
  - Docker
  - FastQC
  - OBITools
  - PANDAseq
  - PEMA
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0233
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# Metabarcoding parameter tuning and false-positive evaluation

## Summary

Systematically test parameter settings (quality trimming, OTU clustering/ASV inference) in a metabarcoding pipeline such as PEMA and evaluate recovery on a mock community (e.g., the Bakker 19-taxon fungal ITS dataset) using false-positive counts and genus-level F1-scores (74–86%) as acceptance criteria.

## When to use

Use when calibrating metabarcoding pipeline parameters on ITS/16S/18S/COI amplicon data or validating recovery of known mock-community members via false positives and F1-scores.

## When NOT to use

- Input is already an OTU/ASV feature table with taxonomies — tuning read-level trimming and clustering parameters would re-process raw reads unnecessarily
- The data are shotgun metagenomic rather than amplicon metabarcoding reads for 16S/18S/12S/ITS/COI marker genes
- There is no expected mock community composition to compare against, so false positives and F1-scores cannot be computed

## Inputs

- Paired-end .fastq(.gz) amplicon reads from a mock community (e.g., Bakker 19 fungal taxa ITS reads from ENA)
- PEMA parameters.tsv file matching the PEMA version in use
- Reference taxonomy database for the marker gene (Unite for ITS, Silva 128/132 for 16S/18S, MIDORI for COI)

## Outputs

- OTU- or ASV-table with taxonomies and per-sample abundances
- False-positive assignment counts against expected mock community composition
- Genus-level F1-scores per parameter setting
- FASTQC per-sample read-quality diagrams

## How to apply

Run FastQC on the .fastq read pairs (e.g., the Bakker 19-fungal-taxon ITS mock community from ENA) to obtain per-sample quality summaries, then vary the parameters.tsv settings, especially for quality trimming (Trimmomatic/Cutadapt) and OTU clustering/ASV inference (VSEARCH, CROP, or Swarm v2 with cluster radius d = 20 for ITS2). Assemble overlapping paired-end reads with PANDAseq, dereplicate with obiuniq, and optionally remove singletons and chimeras (VSEARCH) after clustering. Assign taxonomy with the alignment-based CREST LCAClassifier against Unite (ITS) or Silva (16S/18S). Judge each parameter setting on the mock community by counting false-positive assignments against the expected taxa (e.g., for the ITS setting, Chytriomyces must appear in all 3 samples, Fusarium should not resolve beyond family Nectriaceae, and no more than a single false-positive assignment is accepted); genus-level F1-scores between 74% and 86% indicate comparable quality to the original studies.

## Related tools

- **PEMA** (Containerized pipeline whose parameters.tsv is varied to test trimming, clustering/ASV-inference, and filter settings) — https://github.com/hariszaf/pema
- **Swarm v2** (ASV inference algorithm used for ITS with cluster radius d = 20; chimera removal is applied after inference since Swarm v2 is not affected by chimeras)
- **VSEARCH** (OTU clustering and chimera removal when the OTU route is chosen for parameter testing)
- **CROP** (Alternative unsupervised OTU clustering algorithm to compare against VSEARCH)
- **CREST LCAClassifier** (Alignment-based taxonomic assignment of ASVs/OTUs against Unite (ITS) or Silva (16S/18S))
- **Docker** (Container runtime in which PEMA and its bundled tools are executed for each parameter setting) — https://hub.docker.com/r/hariszaf/pema

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- The ASV/OTU table exists with assigned taxonomies and per-sample abundances for every parameter setting tested
- For the Bakker ITS mock community, Chytriomyces is detected in all 3 samples at the accepted setting
- Fusarium species are not resolved beyond their family (Nectriaceae) under the ITS2 setting
- False-positive assignments do not exceed the single false positive reported for the accepted parameter setting
- Genus-level F1-scores fall within the 74%–86% range reported for PEMA mock community recovery

## Limitations

- Many parameter settings must be tested, especially for quality trimming and OTU clustering/ASV inference, making the tuning computationally expensive
- For COI, taxonomy assignment requires a minimum of 20 GB of RAM, limiting where settings can be tested
- Whether to use ASVs instead of OTUs should be based on each study's objective(s); no single setting is universally optimal
- The choice of parameters file version must match the PEMA version used, or the run will fail

## Evidence

- [methods] Many different parameter settings were tested, especially for the steps of quality trimming of the reads and the OTU clustering/ASV inference: "Many different parameter settings were tested, especially for the steps of quality trimming of the reads and the OTU clustering/ASV inference"
- [methods] Overall the F1-scores ranged from 74% to 86%: "Overall the F1-scores ranged from 74% to 86%"
- [methods] any singletons may be removed: "any singletons, i.e., sequences with only 1 read, occurring after the (M)OTU clustering or the ASV inference may be removed"
- [abstract] PEMA evaluated on mock communities: "PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality."
- [readme] parameters file is crucial: [evidence span withheld — anchor status: not_found]
