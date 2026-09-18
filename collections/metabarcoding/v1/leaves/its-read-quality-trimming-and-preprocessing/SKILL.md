---
name: its-read-quality-trimming-and-preprocessing
description: Use when raw paired-end Illumina .fastq(.gz) reads of the fungal ITS
  marker need adapter/primer removal, quality trimming, error correction, paired-end
  assembly, and dereplication before ASV inference with Swarm v2 in PEMA.
when_to_use_negative:
- Input is already trimmed, merged, or dereplicated sequence data rather than raw
  paired-end .fastq reads.
- The marker gene is 16S/18S rRNA or COI rather than ITS — those PEMA branches use
  different parameter settings and reference databases (Silva, MIDORI) instead of
  the ITS-specific Cutadapt handling of length variability and the Unite database.
- The analysis goal is OTU clustering with VSEARCH/CROP rather than Swarm v2 ASV inference;
  the pre-processing output should then be routed to the (M)OTU clustering branch.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: FastQC
  role: Per-sample read-quality inspection before trimming
- name: Trimmomatic
  role: Adapter/primer removal and quality trimming of ITS read pairs
- name: Cutadapt
  role: Handles the length variability of the ITS marker after Trimmomatic
- name: BayesHammer
  role: Read error correction (SPAdes module) prior to merging
- name: PANDAseq
  role: Assembles overlapping paired-end reads
- name: OBITools
  role: obiuniq groups identical sequences per sample, keeping track of abundances
- name: PEMA
  role: Containerized pipeline (Docker/Singularity) that orchestrates the whole pre-processing
    chain
  repo: github.com/hariszaf/pema
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:3b38488b2adfa7699a0a49c49d7e70ff02c529dd648f7b26b1e1c6b58c35fdb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - CROP
  - Cutadapt
  - FastQC
  - OBITools
  - PANDAseq
  - PEMA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ITS read quality trimming and preprocessing

## Summary

Pre-process fungal ITS amplicon .fastq reads — quality inspection with FastQC, adapter/primer removal and quality trimming with Trimmomatic and Cutadapt, error correction with BayesHammer, merging of overlapping pairs with PANDAseq, and per-sample dereplication with OBITools' obiuniq — as the front end of the PEMA pipeline prior to ITS2 ASV inference.

## When to use

Use when raw paired-end Illumina .fastq(.gz) reads of the fungal ITS marker need adapter/primer removal, quality trimming, error correction, paired-end assembly, and dereplication before ASV inference with Swarm v2 in PEMA.

## When NOT to use

- Input is already trimmed, merged, or dereplicated sequence data rather than raw paired-end .fastq reads.
- The marker gene is 16S/18S rRNA or COI rather than ITS — those PEMA branches use different parameter settings and reference databases (Silva, MIDORI) instead of the ITS-specific Cutadapt handling of length variability and the Unite database.
- The analysis goal is OTU clustering with VSEARCH/CROP rather than Swarm v2 ASV inference; the pre-processing output should then be routed to the (M)OTU clustering branch.

## Inputs

- Paired-end Illumina .fastq.gz read files of fungal ITS amplicons (e.g. Bakker 19-taxa mock community, one pair per sample)
- PEMA parameters.tsv file specifying pre-processing tool settings

## Outputs

- Per-sample read-quality summaries (FastQC reports/diagrams)
- Adapter- and quality-trimmed reads (Trimmomatic, Cutadapt)
- Error-corrected reads (BayesHammer)
- Assembled overlapping paired-end reads (PANDAseq)
- Per-sample dereplicated sequence files with abundances (obiuniq)

## How to apply

Start by running FastQC on each sample to obtain per-base read-quality summaries and confirm the characteristic length variability of the ITS marker. Pre-process the read pairs with Trimmomatic for adapter/primer removal and quality trimming, then invoke Cutadapt specifically to handle ITS length variability. Error-correct bases with BayesHammer (the SPAdes read error-correction module), assemble the overlapping paired-end reads with PANDAseq, and dereplicate identical sequences per sample with the obiuniq program of OBITools, which keeps track of abundances. The output — cleaned, merged, dereplicated sequences with abundances — feeds directly into Swarm v2 ASV inference (cluster radius d = 20 on the ITS2 region) followed by taxonomy assignment with the CREST LCAClassifier against the Unite database. Note that chimera removal is performed after ASV inference, since Swarm v2 is not affected by chimeras, so no pre-clustering chimera filtering is needed at this stage.

## Related tools

- **FastQC** (Per-sample read-quality inspection before trimming)
- **Trimmomatic** (Adapter/primer removal and quality trimming of ITS read pairs)
- **Cutadapt** (Handles the length variability of the ITS marker after Trimmomatic)
- **BayesHammer** (Read error correction (SPAdes module) prior to merging)
- **PANDAseq** (Assembles overlapping paired-end reads)
- **OBITools** (obiuniq groups identical sequences per sample, keeping track of abundances)
- **PEMA** (Containerized pipeline (Docker/Singularity) that orchestrates the whole pre-processing chain) — github.com/hariszaf/pema

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema:v.2.1.4 ; ./pema_latest.bds
```

## Evaluation signals

- FastQC per-sample quality diagrams are produced for every sample before trimming proceeds.
- The dereplicated per-sample sequence files carry abundance counts (obiuniq keeps track of abundances).
- Downstream Swarm v2 ASV inference on ITS2 with d = 20 yields an ASV table with taxonomies from the CREST LCAClassifier/Unite.
- On the Bakker ITS mock community, Chytriomyces is detected in all 3 samples, Fusarium species are not resolved beyond family (Nectriaceae), and false-positive assignments do not exceed the single FP reported for this parameter setting.
- PEMA reports comparable-quality results to the original studies (F1-scores for mock community recovery at genus level ranged from 74% to 86%).

## Limitations

- The ITS marker's length variability makes trimming parameter settings delicate — many different parameter settings were tested, especially for quality trimming and OTU clustering/ASV inference.
- For COI analyses PEMA requires a minimum of 20 GB RAM for taxonomy assignment; ITS runs have different resource profiles but containerized execution still depends on available cores.
- The parameters.tsv file must match the PEMA version being used (e.g. pema:v.2.1.4 needs the v.2.1.4 parameters file), or the run will fail; files and directories must be named exactly as documented.
- Software version numbers for individual pre-processing components are not stated in the article.

## Evidence

- [other] Pre-process reads with Trimmomatic (adapter/primer removal, quality trimming) plus Cutadapt to handle the length variability of the ITS marker, error-correct bases with BayesHammer (SPAdes), assemble overlapping paired-end reads with PANDAseq: [evidence span withheld — anchor status: not_found]
- [methods] Many different parameter settings were tested, especially for the steps of quality trimming of the reads and the OTU clustering/ASV inference: "Many different parameter settings were tested, especially for the steps of quality trimming of the reads and the OTU clustering/ASV inference"
- [methods] the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances: "the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances"
- [readme] As input, PEMA accepts .fastq.gz files as returned by Illumina sequencing platforms.: [evidence span withheld — anchor status: not_found]
