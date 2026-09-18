---
name: deblur-exact-sequence-error-correction
description: Use when starting from per-study demultiplexed ('demux') FASTQ tables
  of 16S amplicon reads (e.g. V4 515f/806r) and you need error-corrected, cluster-free
  sequence variants rather than clustered OTUs.
when_to_use_negative:
- When your sequences are not Illumina 16S amplicon reads with known trim length —
  Deblur requires a fixed trim length (here 90 bp matching the V4 region after 515f)
  shared across all input studies.
- When closed- or open-reference OTU picking against a database (e.g. Greengenes 13.8
  or Silva 123) is required instead of de novo exact-sequence inference.
- When the input is already a merged, error-corrected feature/BIOM table — this skill
  operates upstream on demultiplexed reads.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Deblur
  role: Exact-sequence de novo OTU picking method used to error-correct demultiplexed
    reads into 90-bp V4 tag sequences
  repo: https://github.com/cuttlefishh/deblur
- name: QIIME1
  role: Initial processing (barcode splitting, quality filtering) and merging of per-study
    BIOM tables and mapping files
- name: QIIME2
  role: Alternative to QIIME1 for merging the BIOM tables and mapping files
- name: Python
  role: Scripting and meta-analysis of EMP data around the Deblur processing pipeline
  repo: https://github.com/biocore/emp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:b6a7c302712f8bb19749c16333b2b3b8a0c36cdabd9c018823e23d8769dc2008
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Python
  - QIIME1
  - QIIME2
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# deblur-exact-sequence-error-correction

## Summary

Run Deblur, the exact-sequence de novo OTU picking method, on demultiplexed 16S reads to correct Illumina sequencing errors into exact amplicon sequence variants, enabling sequences to be followed across multiple EMP studies without OTU clustering.

## When to use

Use when starting from per-study demultiplexed ('demux') FASTQ tables of 16S amplicon reads (e.g. V4 515f/806r) and you need error-corrected, cluster-free sequence variants rather than clustered OTUs.

## When NOT to use

- When your sequences are not Illumina 16S amplicon reads with known trim length — Deblur requires a fixed trim length (here 90 bp matching the V4 region after 515f) shared across all input studies.
- When closed- or open-reference OTU picking against a database (e.g. Greengenes 13.8 or Silva 123) is required instead of de novo exact-sequence inference.
- When the input is already a merged, error-corrected feature/BIOM table — this skill operates upstream on demultiplexed reads.

## Inputs

- Per-study demultiplexed FASTQ files ('demux' tables) from EMP 16S Release 1 (ftp://ftp.microbio.me/emp/release1)
- Raw 16S rRNA amplicon sequence files with barcode information for demultiplexing

## Outputs

- Deblur feature table (BIOM format) of exact 90-bp V4 tag sequences per study
- Merged QC-filtered BIOM table (e.g. emp_deblur_90bp_qc.biom; 307,572 unique sequences, 96 studies, 23,828 samples after rarefaction to 5,000 sequences/sample)

## How to apply

Take the per-study demultiplexed FASTQ data produced by initial QIIME processing (reads split into per-sample files by barcode and quality filtered), then run Deblur to pick exact-sequence de novo OTUs. Trim sequences to 90 bp to match the V4 tag length starting after primer 515f, so sequences from different studies are directly comparable. Filter the resulting feature table with the mc2 rule (minimum sequence count of 2 per OTU) and retain only samples with >=25 total reads. Merge the per-study BIOM tables with QIIME1 or QIIME2 into a single table (e.g. emp_deblur_90bp_qc.biom), and optionally rarefy to 5,000 observations per sample. The choice of exact sequences over clustered OTUs is the key rationale: it allows bacterial and archaeal rRNA gene sequences to be followed across multiple studies.

## Related tools

- **Deblur** (Exact-sequence de novo OTU picking method used to error-correct demultiplexed reads into 90-bp V4 tag sequences) — https://github.com/cuttlefishh/deblur
- **QIIME1** (Initial processing (barcode splitting, quality filtering) and merging of per-study BIOM tables and mapping files)
- **QIIME2** (Alternative to QIIME1 for merging the BIOM tables and mapping files)
- **Python** (Scripting and meta-analysis of EMP data around the Deblur processing pipeline) — https://github.com/biocore/emp

## Examples

```
redbiom search metadata beer | head | redbiom fetch samples --context $ctx --output beer_example.biom
```

## Evaluation signals

- Trimmed tag sequences are exactly 90 bp, corresponding to the V4 region starting after primer 515f.
- Feature table filtered by mc2 rule: no OTU/feature is retained with total sequence count < 2.
- Only samples with >= 25 total reads are retained in the filtered table.
- Merged table reproduces the reference counts: 307,572 unique sequences across 96 studies and 23,828 samples after rarefaction to 5,000 observations per sample.
- Output is a valid BIOM table mergeable via QIIME1 or QIIME2 along with merged mapping files.

## Limitations

- Deblur is de novo: features are exact amplicon sequences, not taxonomy-assigned OTU IDs; closed-reference picking against Greengenes 13.8 or Silva 123 is a separate pathway.
- The 90-bp trim length is specific to the V4 tag region — sequences from other amplicon regions or read lengths are not directly comparable.
- In redbiom, sample data loaded from different processing contexts (e.g. Deblur vs. closed-reference OTUs) are stored as separate contexts, so the processing type must be tracked when retrieving data.

## Evidence

- [methods] deblur refers to the exact-sequence de novo OTU picking method Deblur: "deblur refers to the exact-sequence de novo OTU picking method Deblur"
- [supplementary] Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm.: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [methods] mc2 (minimum sequence count in an OTU to be included equals to 2): "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [methods] You can then use QIIME1 or QIIME2 to merge the BIOM tables and mapping files.: [evidence span withheld — anchor status: not_found]
- [abstract] use of exact sequences instead of clustered OTUs enables cross-study tracking: "Coordinated protocols and new analytical methods, particularly the use of exact sequences instead of clustered operational taxonomic units, enable bacterial and archaeal ribosomal RNA gene sequences"
