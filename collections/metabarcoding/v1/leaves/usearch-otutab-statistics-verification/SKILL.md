---
name: usearch-otutab-statistics-verification
description: Use when a quality-filtered FASTA from amplicon QC must be aligned against
  Greengenes 97% OTUs and the reported alignment rate (e.g. 80.0% with 4 threads)
  or OTU-table validity needs verification.
when_to_use_negative:
- Input is already a feature/OTU table — alignment against Greengenes is only for
  raw filtered reads
- The analysis targets ITS/fungal data, where the UNITE database (not Greengenes 97%
  OTUs) is the appropriate reference
- The goal is ASV (denoised variant) inference rather than reference-based OTU assignment
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: EasyAmplicon
  role: Pipeline hosting the functional-prediction alignment step and defining file
    paths (temp/filtered.fa, result/gg/otutab.txt) and the 4-thread Method 1 variant
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: VSEARCH
  role: Open-source engine for vsearch --usearch_global alignment of filtered reads
    against Greengenes 97% OTUs at --id 0.97
  repo: https://github.com/torognes/vsearch
- name: USEARCH
  role: Runs -otutab_stats to summarize the OTU table and provides the USEARCH-compatible
    workflow whose reported alignment rate (80.0%) is the validation target
- name: EasyMicrobiome
  role: Supplies the scripts and reference databases (including Greengenes 97% OTUs)
    used by the EasyAmplicon alignment step
  repo: https://github.com/YongxinLiu/EasyMicrobiome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:23f7fea16951fd5ca3b78a9c50455a3d2494b9f623e4ecaeb623ba74f12fe2cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# usearch-otutab-statistics-verification

## Summary

Verifies the USEARCH/VSEARCH global-alignment of quality-filtered amplicon reads (temp/filtered.fa) against the Greengenes 97% OTUs database by inspecting the resulting OTU table with -otutab_stats and computing the overall alignment rate, which EasyAmplicon reports as 80.0% with 4 threads.

## When to use

Use when a quality-filtered FASTA from amplicon QC must be aligned against Greengenes 97% OTUs and the reported alignment rate (e.g. 80.0% with 4 threads) or OTU-table validity needs verification.

## When NOT to use

- Input is already a feature/OTU table — alignment against Greengenes is only for raw filtered reads
- The analysis targets ITS/fungal data, where the UNITE database (not Greengenes 97% OTUs) is the appropriate reference
- The goal is ASV (denoised variant) inference rather than reference-based OTU assignment

## Inputs

- Quality-filtered FASTA of merged amplicon reads (temp/filtered.fa) from the upstream QC step
- Greengenes 97% OTUs reference database (${db}/gg/97_otus.fa, USEARCH-compatible format)
- Sample metadata sheet for cross-checking OTU-table columns

## Outputs

- Reference OTU table result/gg/otutab.txt (OTUs x samples)
- OTU-table statistics summary from usearch -otutab_stats (per-sample sequence counts, column/sample count)
- Overall USEARCH alignment rate (reported value: 80.0% with 4 threads)

## How to apply

Load the quality-filtered sequence set temp/filtered.fa produced by the upstream QC step and run vsearch --usearch_global against the Greengenes 97% OTUs database (${db}/gg/97_otus.fa) at --id 0.97 with multithreading (Method 1 uses 4 threads), writing the reference OTU table result/gg/otutab.txt. Then summarize the table with usearch -otutab_stats, checking per-sample sequence counts and that the column count matches the number of samples in the metadata. Compute the overall alignment rate as the fraction of reads assigned to Greengenes OTUs from the alignment output. VSEARCH is the open-source drop-in replacement for USEARCH here; it uses a global (Needleman-Wunsch) aligner, so results are full-length global alignments rather than heuristic seed-and-extend matches. Validation succeeds when the computed rate matches the reported 80.0% (metric_usearch_align_rate) within reasonable tolerance and result/gg/otutab.txt is a valid OTU table whose sample columns match the metadata.

## Related tools

- **EasyAmplicon** (Pipeline hosting the functional-prediction alignment step and defining file paths (temp/filtered.fa, result/gg/otutab.txt) and the 4-thread Method 1 variant) — https://github.com/YongxinLiu/EasyAmplicon
- **VSEARCH** (Open-source engine for vsearch --usearch_global alignment of filtered reads against Greengenes 97% OTUs at --id 0.97) — https://github.com/torognes/vsearch
- **USEARCH** (Runs -otutab_stats to summarize the OTU table and provides the USEARCH-compatible workflow whose reported alignment rate (80.0%) is the validation target)
- **EasyMicrobiome** (Supplies the scripts and reference databases (including Greengenes 97% OTUs) used by the EasyAmplicon alignment step) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
vsearch --usearch_global temp/filtered.fa --db ${db}/gg/97_otus.fa --id 0.97 --otutab result/gg/otutab.txt --threads 4; usearch -otutab_stats result/gg/otutab.txt
```

## Evaluation signals

- Computed USEARCH-alignment rate matches the reported 80.0% (metric_usearch_align_rate) within reasonable tolerance
- result/gg/otutab.txt exists and parses as a valid OTU table
- Column count of result/gg/otutab.txt equals the number of samples in the metadata
- usearch -otutab_stats reports per-sample sequence counts consistent with the input read counts in temp/filtered.fa
- Alignment was run at --id 0.97 against ${db}/gg/97_otus.fa with the expected thread count (4 in Method 1)

## Limitations

- The 80.0% alignment rate is dataset- and pipeline-version-specific; other datasets or database versions may deviate, so only 'reasonable tolerance' matching is expected
- VSEARCH uses an optimal global (Needleman-Wunsch) aligner while USEARCH defaults to a heuristic seed-and-extend aligner, so exact rates can differ between the two tools
- The reported finding is a benchmark/positive-control value from the EasyAmplicon pipeline rather than a universal constant

## Evidence

- [methods] Reported 80.0% alignment rate with 4 threads: [evidence span withheld — anchor status: not_found]
- [methods] Workflow defines inputs, database, identity threshold, and output table: [evidence span withheld — anchor status: not_found]
- [results] Validation criterion for the skill: [evidence span withheld — anchor status: not_found]
- [readme] VSEARCH global-aligner rationale (readme): [evidence span withheld — anchor status: external_doc]
- [intro] Greengenes availability in USEARCH-compatible format (intro): "users can download Greengenes database (https://ftp.microbio.me/greengenes_release/current/) in USEARCH‐compatible format"
