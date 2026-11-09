---
name: 16s-v4-read-trimming
description: Use when processing Illumina 16S V4 amplicon reads (515f/806r) with Deblur,
  where sequences must be trimmed to 90 bp so identical V4 tag sequences are comparable
  across multiple studies before merging BIOM tables.
when_to_use_negative:
- Amplicon data from a non-V4 hypervariable region — the 90-bp trim length matches
  only the V4 tag starting after primer 515f
- Input is already a merged or rarefied feature/BIOM table — trimming applies to demultiplexed
  per-sample fastq reads prior to feature generation
- Pipelines that rely on clustered OTUs rather than exact-sequence methods (e.g. closed-reference
  OTU picking against Greengenes 13.8), which do not require uniform 90-bp tag trimming
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Deblur
  role: Exact-sequence de novo OTU picking method used to derive 90-bp V4 tag sequences
    from demultiplexed reads
  repo: https://github.com/cuttlefishh/deblur
- name: QIIME1
  role: Initial processing (barcode splitting, quality filtering) and merging of per-study
    BIOM tables and mapping files
- name: QIIME2
  role: Merging per-study BIOM tables and mapping files into a single merged table
- name: redbiom
  role: Querying the Qiita/EMP catalogue to retrieve sample data into BIOM from processing
    contexts such as the Deblur 90-bp table
  repo: https://github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:53060478deef5c4a1d3d3ce532a97cbc482bf8ce6299e502dad3519f554ea2c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Python
  - QIIME1
  - QIIME2
  - redbiom
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# 16S V4 read trimming

## Summary

Trimming 16S rRNA amplicon reads to a fixed 90-bp length corresponding to the V4 tag region (starting after primer 515f) so that exact-sequence features (Deblur) can be compared across hundreds of EMP studies in a merged BIOM table.

## When to use

Use when processing Illumina 16S V4 amplicon reads (515f/806r) with Deblur, where sequences must be trimmed to 90 bp so identical V4 tag sequences are comparable across multiple studies before merging BIOM tables.

## When NOT to use

- Amplicon data from a non-V4 hypervariable region — the 90-bp trim length matches only the V4 tag starting after primer 515f
- Input is already a merged or rarefied feature/BIOM table — trimming applies to demultiplexed per-sample fastq reads prior to feature generation
- Pipelines that rely on clustered OTUs rather than exact-sequence methods (e.g. closed-reference OTU picking against Greengenes 13.8), which do not require uniform 90-bp tag trimming

## Inputs

- Per-study demultiplexed fastq files (demux tables) from EMP 16S Release 1 raw sequence deposits
- Barcode/mapping files for per-sample splitting

## Outputs

- 90-bp V4 tag-sequence features from Deblur per study
- Merged QC-filtered BIOM table (emp_deblur_90bp_qc.biom) with 96 studies, 23,828 samples, and 307,572 unique sequences after rarefaction to 5,000 sequences per sample

## How to apply

First demultiplex raw reads into per-sample fastq files by barcode with initial QIIME processing to build per-study 'demux' tables. Then run Deblur, the exact-sequence de novo OTU picking method, on the demultiplexed reads, trimming each sequence to 90 bp to match the V4 tag length starting after primer 515f. Apply the mc2 filter (minimum sequence count in an OTU to be included equals 2) and retain only samples with >=25 total reads. Rarefy samples to 5,000 observations per sample, then merge the per-study BIOM tables with QIIME1 or QIIME2. Rationale: using exact sequences instead of clustered OTUs at a uniform tag length enables bacterial and archaeal rRNA gene sequences to be followed across multiple studies.

## Related tools

- **Deblur** (Exact-sequence de novo OTU picking method used to derive 90-bp V4 tag sequences from demultiplexed reads) — https://github.com/cuttlefishh/deblur
- **QIIME1** (Initial processing (barcode splitting, quality filtering) and merging of per-study BIOM tables and mapping files)
- **QIIME2** (Merging per-study BIOM tables and mapping files into a single merged table)
- **redbiom** (Querying the Qiita/EMP catalogue to retrieve sample data into BIOM from processing contexts such as the Deblur 90-bp table) — https://github.com/biocore/redbiom

## Evaluation signals

- Trimmed tag sequences are exactly 90 bp, matching the V4 region starting after primer 515f
- Merged QC-filtered table validates against 96 studies, 23,828 samples, and 307,572 unique sequences after rarefaction to 5,000 observations per sample
- Feature table was filtered with the mc2 rule (minimum sequence count of 2 per OTU) and retains only samples with >=25 total reads
- All per-study BIOM tables merged without sample-ID or feature-ID conflicts (table saved as emp_deblur_90bp_qc.biom)

## Limitations

- The 90-bp trim is specific to the V4 tag; sequences shorter than the tag length or from other amplicon regions are not handled by this skill
- Counts in the published table depend on the release scope: EMP 16S Release 1 covers ~25,000 samples in 97 studies, while the QC-filtered rarefied table retained 96 studies and 23,828 samples
- redbiom data are organized by processing context; loading data into a shared context requires explicit --tag use to differentiate BIOM tables with potentially overlapping sample IDs

## Evidence

- [supplementary] Deblur 90-bp tag sequences: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [methods] mc2 filter definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [supplementary] Rarefaction threshold: "Samples were rarefied to 5,000 observations per sample."
- [abstract] Exact-sequence method rationale: "the use of exact sequences instead of clustered operational taxonomic units"
- [methods] Demultiplexing step: "demux is shorthand for "demultiplexed", which describes the fastq data after it is split into per-sample fastq files using barcodes"
