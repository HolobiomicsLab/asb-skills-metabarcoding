---
name: alignment-rate-qc-verification
description: Use when a VSEARCH --usearch_global OTU-picking step (97% identity, Greengenes
  97_otus.fa reference) has just run and you must confirm the reported mapping rate
  matches the EasyAmplicon expected ~81.04% before downstream analysis.
when_to_use_negative:
- Input is already an OTU/feature table rather than quality-filtered reads — the alignment
  rate check only applies to the read-to-reference step
- Reference database is not Greengenes 97% OTUs (e.g. RDP, SILVA, UNITE) — the 81.04%
  benchmark was established for the Greengenes 97_otus.fa condition and is not transferable
- Different identity threshold or pipeline stage (e.g. ASV inference, chimera removal)
  — the ~81.04% expectation is specific to --id 0.97 OTU picking in EasyAmplicon
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: VSEARCH
  role: Performs the --usearch_global alignment of filtered reads against Greengenes
    97_otus.fa at 97% identity with 12 threads and reports the mapping rate
  repo: https://github.com/torognes/vsearch
- name: USEARCH
  role: Used to validate the resulting OTU table via usearch -otutab_stats (per-sample
    counts, table validity)
- name: EasyAmplicon
  role: Provides the pipeline, positive-control data, and the expected ~81.04% alignment-rate
    benchmark for Method 2 (VSEARCH alignment)
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:aef583fb2ba9c312de3c1e48e3f75933b800fa6f0ffeb0aeb59e935e7349f482
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.14285714285714285
n_claims: 7
---

# alignment-rate-qc-verification

## Summary

Verifies the read-to-reference mapping (alignment) rate reported by VSEARCH --usearch_global when aligning quality-filtered amplicon reads against the Greengenes 97% OTUs database, using the EasyAmplicon positive-control expectation of ~81.04% as the benchmark.

## When to use

Use when a VSEARCH --usearch_global OTU-picking step (97% identity, Greengenes 97_otus.fa reference) has just run and you must confirm the reported mapping rate matches the EasyAmplicon expected ~81.04% before downstream analysis.

## When NOT to use

- Input is already an OTU/feature table rather than quality-filtered reads — the alignment rate check only applies to the read-to-reference step
- Reference database is not Greengenes 97% OTUs (e.g. RDP, SILVA, UNITE) — the 81.04% benchmark was established for the Greengenes 97_otus.fa condition and is not transferable
- Different identity threshold or pipeline stage (e.g. ASV inference, chimera removal) — the ~81.04% expectation is specific to --id 0.97 OTU picking in EasyAmplicon

## Inputs

- temp/filtered.fa — quality-filtered amplicon reads (FASTA) from EasyAmplicon preprocessing
- ${db}/gg/97_otus.fa — USEARCH-compatible Greengenes 97% OTUs reference database (FASTA)

## Outputs

- result/gg/otutab.txt — tab-delimited sample-by-OTU count table
- Reported VSEARCH alignment (mapping) rate, expected ≈ 81.04%
- usearch -otutab_stats summary of per-sample sequence counts and OTU-table validity

## How to apply

Run VSEARCH global alignment of the quality-filtered read set temp/filtered.fa against the USEARCH-compatible Greengenes reference library ${db}/gg/97_otus.fa with --id 0.97 and --threads 12, writing the sample-by-OTU matrix to result/gg/otutab.txt. Capture the mapping rate VSEARCH reports to stdout and compare it to the expected ~81.04% for this condition; a large deviation signals a problem with filtering, primer/length trimming, or the reference database. Validate the resulting table with usearch -otutab_stats to confirm it is a well-formed tab-delimited OTU table and that per-sample sequence counts are sane. VSEARCH is used here because it applies an optimal global aligner (full Needleman-Wunsch dynamic programming), which gives more accurate alignments and better sensitivity than USEARCH's default heuristic seed-and-extend aligner.

## Related tools

- **VSEARCH** (Performs the --usearch_global alignment of filtered reads against Greengenes 97_otus.fa at 97% identity with 12 threads and reports the mapping rate) — https://github.com/torognes/vsearch
- **USEARCH** (Used to validate the resulting OTU table via usearch -otutab_stats (per-sample counts, table validity))
- **EasyAmplicon** (Provides the pipeline, positive-control data, and the expected ~81.04% alignment-rate benchmark for Method 2 (VSEARCH alignment)) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
vsearch --usearch_global temp/filtered.fa --db ${db}/gg/97_otus.fa --otutabout result/gg/otutab.txt --id 0.97 --threads 12; usearch -otutab_stats result/gg/otutab.txt
```

## Evaluation signals

- VSEARCH-reported alignment (mapping) rate is approximately 81.04% for the Greengenes 97% OTUs, --id 0.97, 12-thread condition
- result/gg/otutab.txt exists and is a well-formed tab-delimited OTU table
- usearch -otutab_stats runs without errors and per-sample sequence counts are consistent with the filtered read set
- Every filtered read in temp/filtered.fa was submitted for mapping at the 97% identity threshold (no silently dropped inputs)
- Command completed using all 12 threads without terminating early

## Limitations

- The ~81.04% rate is tied to the specific EasyAmplicon positive-control dataset and Greengenes 97% OTUs database; other datasets or databases will yield different rates
- VSEARCH does not support amino acid sequences or local alignments, so the check is limited to nucleotide global-alignment searching
- The rate depends on upstream preprocessing (primer removal, length/quality filtering, chimera removal); failures in those steps change the mapping rate without VSEARCH itself being misconfigured

## Evidence

- [readme] VSEARCH uses an optimal global aligner (full dynamic programming Needleman-Wunsch), in contrast to USEARCH which by default uses a heuristic seed and extend aligner.: [evidence span withheld — anchor status: external_doc]
- [methods] Pre-packaged reference databases (e.g., RDP/SILVA/UNITE for taxonomy, Greengenes 97% OTUs for PICRUSt): "Pre-packaged reference databases (e.g., RDP/SILVA/UNITE for taxonomy, Greengenes 97% OTUs for PICRUSt)"
- [intro] VSEARCH v2.7.1+52 or later: VSEARCH is an open‐source tool for performing tasks such as sequence clustering, chimera detection, and dereplication.: "VSEARCH v2.7.1+52 or later: VSEARCH is an open‐source tool for performing tasks such as sequence clustering, chimera detection, and dereplication."
