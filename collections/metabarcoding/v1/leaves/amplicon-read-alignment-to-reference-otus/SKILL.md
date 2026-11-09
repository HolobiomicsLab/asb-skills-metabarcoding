---
name: amplicon-read-alignment-to-reference-otus
description: Use when you have a quality-filtered amplicon FASTA (e.g., temp/filtered.fa
  from vsearch --fastx_filter) and need an OTU table by aligning reads against a USEARCH-compatible
  reference such as Greengenes 97% OTUs for downstream taxonomy or PICRUSt functional
  prediction.
when_to_use_negative:
- Input is already an OTU/feature table or ASV sequences rather than raw quality-filtered
  reads — alignment to reference OTUs is redundant
- Reference database is not in USEARCH-compatible format or targets ITS/eukaryotic
  amplicons better served by UNITE than the Greengenes 97% OTUs database
- Local (not global) alignment is required — VSEARCH does not support local alignments,
  as documented in its README
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: VSEARCH
  role: Runs the --usearch_global alignment of filtered reads against Greengenes 97%
    OTUs at --id 0.97 with multithreading, writing the OTU table
  repo: https://github.com/torognes/vsearch
- name: USEARCH
  role: Alternative aligner (Method 1, 4 threads) and provider of -otutab_stats for
    validating the resulting OTU table
- name: EasyAmplicon
  role: Pipeline that integrates the alignment step and specifies file paths (temp/filtered.fa,
    result/gg/otutab.txt) and the Greengenes database location
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Supplies the scripts and pre-packaged reference databases (e.g., Greengenes
    97% OTUs) used by the alignment step
  repo: https://github.com/YongxinLiu/EasyMicrobiome
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:e6fd525451e321a4994ffdded6631c083b8e5bba8de1f44178a86e9ebc7af4d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Amplicon read alignment to reference OTUs

## Summary

Maps quality-filtered amplicon reads to a reference OTU database (e.g., Greengenes 97% OTUs) using VSEARCH's --usearch_global global alignment at a 97% identity threshold, producing a sample-by-OTU count table whose read-mapping rate serves as a key quality metric in the EasyAmplicon pipeline.

## When to use

Use when you have a quality-filtered amplicon FASTA (e.g., temp/filtered.fa from vsearch --fastx_filter) and need an OTU table by aligning reads against a USEARCH-compatible reference such as Greengenes 97% OTUs for downstream taxonomy or PICRUSt functional prediction.

## When NOT to use

- Input is already an OTU/feature table or ASV sequences rather than raw quality-filtered reads — alignment to reference OTUs is redundant
- Reference database is not in USEARCH-compatible format or targets ITS/eukaryotic amplicons better served by UNITE than the Greengenes 97% OTUs database
- Local (not global) alignment is required — VSEARCH does not support local alignments, as documented in its README

## Inputs

- Quality-filtered amplicon FASTA file (temp/filtered.fa) from upstream QC steps
- USEARCH-compatible Greengenes 97% OTUs reference database (${db}/gg/97_otus.fa)
- Sample metadata for validating sample columns

## Outputs

- Sample-by-OTU count table (result/gg/otutab.txt, tab-delimited OTU table)
- Overall alignment/mapping rate (e.g., 80.0% for USEARCH 4 threads, 81.04% for VSEARCH 12 threads)
- OTU table statistics summary from usearch -otutab_stats

## How to apply

Load the quality-filtered sequence set (temp/filtered.fa) produced by upstream QC (primer removal, length/quality filtering with --fastq_minlen 1200 --fastq_maxlen 1800, chimera removal) and the reference database ${db}/gg/97_otus.fa in USEARCH-compatible format. Run a global alignment with vsearch --usearch_global at --id 0.97, using --otutabout to write result/gg/otutab.txt and --threads for multithreading (EasyAmplicon exercises both a 4-thread USEARCH condition and a 12-thread VSEARCH condition). Compute the overall alignment rate (fraction of filtered reads assigned to Greengenes OTUs) and inspect the OTU table with usearch -otutab_stats to confirm per-sample counts and that the sample columns match the metadata. VSEARCH is preferred over USEARCH because it uses an optimal global aligner (full Needleman-Wunsch dynamic programming) rather than a heuristic seed-and-extend aligner, improving sensitivity.

## Related tools

- **VSEARCH** (Runs the --usearch_global alignment of filtered reads against Greengenes 97% OTUs at --id 0.97 with multithreading, writing the OTU table) — https://github.com/torognes/vsearch
- **USEARCH** (Alternative aligner (Method 1, 4 threads) and provider of -otutab_stats for validating the resulting OTU table)
- **EasyAmplicon** (Pipeline that integrates the alignment step and specifies file paths (temp/filtered.fa, result/gg/otutab.txt) and the Greengenes database location) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Supplies the scripts and pre-packaged reference databases (e.g., Greengenes 97% OTUs) used by the alignment step) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
vsearch --usearch_global temp/filtered.fa --db ${db}/gg/97_otus.fa --otutabout result/gg/otutab.txt --id 0.97 --threads 12
```

## Evaluation signals

- Computed alignment rate matches the reported values: ~80.0% for the USEARCH-based 4-thread condition and ~81.04% for the VSEARCH 12-thread condition, within reasonable tolerance
- result/gg/otutab.txt exists and is a well-formed tab-delimited OTU table
- Number of sample columns in the OTU table matches the number of samples in the metadata
- usearch -otutab_stats runs without error and reports sensible per-sample sequence counts
- All filtered reads were searched at the --id 0.97 identity threshold against the Greengenes 97% OTUs database

## Limitations

- The alignment rate depends on the reference database choice; Greengenes 97% OTUs is used specifically for the PICRUSt functional-prediction route, and other databases (RDP/SILVA/UNITE) serve taxonomy purposes
- VSEARCH does not support amino acid sequences or local alignments, so this global-alignment approach cannot be extended to protein-level searching
- Reads below the 97% identity threshold remain unassigned, and the reported rates (80.0%/81.04%) are specific to the EasyAmplicon positive-control dataset (GSA CRA002352) and may vary with sample type and read quality

## Evidence

- [methods] Greengenes 97% OTUs for PICRUSt: "Pre-packaged reference databases (e.g., RDP/SILVA/UNITE for taxonomy, Greengenes 97% OTUs for PICRUSt)"
- [intro] USEARCH-compatible Greengenes: "users can download Greengenes database (https://ftp.microbio.me/greengenes_release/current/) in USEARCH‐compatible format"
- [readme] VSEARCH global aligner rationale: [evidence span withheld — anchor status: external_doc]
- [readme] VSEARCH usearch_global example invocation: [evidence span withheld — anchor status: external_doc]
