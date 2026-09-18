---
name: greengenes-otu-table-generation
description: Use when quality-filtered 16S amplicon reads are available and an OTU-level
  feature table against Greengenes 97% OTUs is needed for downstream EasyAmplicon
  steps (e.g., PICRUSt functional prediction).
when_to_use_negative:
- The input is already a feature/OTU table — this skill aligns raw filtered reads
  to a reference, not an existing table
- The amplicon target is fungal/eukaryotic ITS — the Greengenes database is for 16S
  bacterial OTUs; UNITE is the relevant reference in the EasyAmplicon database set
- An ASV (amplicon sequence variant) table is required instead of 97%-identity OTU
  clustering
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0610
tools:
- name: VSEARCH
  role: Runs --usearch_global global alignment of filtered reads against Greengenes
    97% OTUs at 97% identity to emit the OTU table (Method 2, 12 threads, ~81.04%
    alignment rate)
  repo: https://github.com/torognes/vsearch
- name: USEARCH
  role: Alternative alignment engine (Method 1, 4 threads, 80.0% rate) and provider
    of -otutab_stats for validating the resulting OTU table
- name: EasyAmplicon
  role: Pipeline that packages the filtered.fa generation, Greengenes database paths,
    and the otutab generation/validation steps
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Distributes the scripts and reference databases (including Greengenes 97%
    OTUs) used by the EasyAmplicon alignment step
  repo: https://github.com/YongxinLiu/EasyMicrobiome
merged_aliases:
- greengenes-reference-otu-mapping
merged_alias_records:
- alias: greengenes-reference-otu-mapping
  slug: greengenes-reference-otu-mapping
  cosine_score: 0.8121
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:6b1b557dc79ad6134ce393ee7bb385037bd38f17d42d59d33f8531d554b756ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# greengenes-otu-table-generation

## Summary

Generates a sample-by-OTU feature table by globally aligning quality-filtered amplicon reads (temp/filtered.fa) against the Greengenes 97% OTUs reference database using USEARCH/VSEARCH --usearch_global at 97% identity, as done in the EasyAmplicon pipeline.

## When to use

Use when quality-filtered 16S amplicon reads are available and an OTU-level feature table against Greengenes 97% OTUs is needed for downstream EasyAmplicon steps (e.g., PICRUSt functional prediction).

## When NOT to use

- The input is already a feature/OTU table — this skill aligns raw filtered reads to a reference, not an existing table
- The amplicon target is fungal/eukaryotic ITS — the Greengenes database is for 16S bacterial OTUs; UNITE is the relevant reference in the EasyAmplicon database set
- An ASV (amplicon sequence variant) table is required instead of 97%-identity OTU clustering

## Inputs

- Quality-filtered amplicon reads (FASTA, e.g. temp/filtered.fa)
- Greengenes 97% OTUs reference database in USEARCH-compatible format (${db}/gg/97_otus.fa)
- Sample metadata (to verify OTU table sample columns match)

## Outputs

- Sample-by-OTU tab-delimited OTU table (result/gg/otutab.txt)
- Alignment/mapping rate summary (e.g., 80.0% or 81.04% of reads assigned to Greengenes OTUs)
- otutab_stats summary of per-sample sequence counts and table validity

## How to apply

Load the quality-filtered sequence set temp/filtered.fa produced by upstream quality control, and the USEARCH-compatible Greengenes reference library ${db}/gg/97_otus.fa (downloadable from ftp.microbio.me in USEARCH-compatible format). Run a global alignment with vsearch --usearch_global at --id 0.97, multithreaded (EasyAmplicon reports Method 1 with USEARCH-style parameters at 4 threads and Method 2 with VSEARCH at 12 threads), writing the OTU table to result/gg/otutab.txt via --otutabout. Inspect the table with usearch -otutab_stats to summarize per-sample sequence counts, and confirm the sample column count matches the metadata. Compute the overall alignment rate from the mapping output: EasyAmplicon reports 80.0% for the USEARCH 4-thread condition and 81.04% for the VSEARCH 12-thread condition against the Greengenes 97% OTUs database.

## Related tools

- **VSEARCH** (Runs --usearch_global global alignment of filtered reads against Greengenes 97% OTUs at 97% identity to emit the OTU table (Method 2, 12 threads, ~81.04% alignment rate)) — https://github.com/torognes/vsearch
- **USEARCH** (Alternative alignment engine (Method 1, 4 threads, 80.0% rate) and provider of -otutab_stats for validating the resulting OTU table)
- **EasyAmplicon** (Pipeline that packages the filtered.fa generation, Greengenes database paths, and the otutab generation/validation steps) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Distributes the scripts and reference databases (including Greengenes 97% OTUs) used by the EasyAmplicon alignment step) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
vsearch --usearch_global temp/filtered.fa --db ${db}/gg/97_otus.fa --otutabout result/gg/otutab.txt --id 0.97 --threads 12; usearch -otutab_stats result/gg/otutab.txt
```

## Evaluation signals

- result/gg/otutab.txt exists and is a well-formed tab-delimited OTU table
- The OTU table's sample columns match the sample count and names in the metadata
- usearch -otutab_stats runs successfully and reports sensible per-sample sequence counts
- The computed read-mapping rate is approximately 80.0% for the USEARCH 4-thread condition or ~81.04% for the VSEARCH 12-thread condition (within reasonable tolerance)

## Limitations

- The reported alignment rates (80.0% and 81.04%) are for the EasyAmplicon positive-control dataset; other datasets may yield different rates, and only 'reasonable tolerance' is specified
- VSEARCH uses an optimal global Needleman-Wunsch aligner versus USEARCH's heuristic seed-and-extend, so the two methods may differ in sensitivity, especially for alignments with gaps
- The article provides no DOI/journal or changelog details in some sections, and pre-packaged databases must be downloaded separately (e.g., Greengenes from ftp.microbio.me)

## Evidence

- [methods] VSEARCH global alignment command at 97% identity against Greengenes: [evidence span withheld — anchor status: not_found]
- [intro] Greengenes available in USEARCH-compatible format: "users can download Greengenes database (https://ftp.microbio.me/greengenes_release/current/) in USEARCH‐compatible format"
- [readme] VSEARCH global alignment capability documented in README: [evidence span withheld — anchor status: external_doc]
- [methods] Pre-packaged databases in EasyMicrobiome include Greengenes 97% OTUs: "Pre-packaged reference databases (e.g., RDP/SILVA/UNITE for taxonomy, Greengenes 97% OTUs for PICRUSt)"
