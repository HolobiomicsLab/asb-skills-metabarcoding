---
name: otu-abundance-table-construction
description: Use when quality-filtered amplicon reads (e.g., temp/filtered.fa) need
  to be mapped to a reference OTU database at 97% identity to produce a valid tab-delimited
  OTU count table.
when_to_use_negative:
- Input is already a feature/OTU table — this skill maps raw filtered reads to a reference,
  it does not transform existing count matrices.
- The goal is de novo OTU clustering rather than reference-based mapping of reads
  to a pre-built database such as Greengenes 97% OTUs.
- Sequences are amino acid or require local alignment — VSEARCH does not support amino
  acid sequences or local alignments.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: VSEARCH
  role: Performs the global alignment (--usearch_global) of filtered reads against
    the Greengenes 97% OTUs database at --id 0.97 with 12 threads, producing the OTU
    table via --otutabout.
  repo: https://github.com/torognes/vsearch
- name: USEARCH
  role: Used with -otutab_stats to verify per-sample sequence counts and the validity
    of the sample-by-OTU matrix.
- name: EasyAmplicon
  role: Pipeline providing the quality-filtered reads (temp/filtered.fa), database
    paths, and the positive-control workflow this skill reproduces.
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Supplies the scripts and reference databases (including USEARCH-compatible
    Greengenes) used by EasyAmplicon.
  repo: https://github.com/YongxinLiu/EasyMicrobiome
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:6fba905564c67f9e1205a08a67365d47c6b7adeb9f4c2e5392fb9b189af6d86d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# otu-abundance-table-construction

## Summary

Construct a sample-by-OTU abundance table by globally aligning quality-filtered amplicon reads against a reference OTU database (e.g., Greengenes 97% OTUs) with VSEARCH --usearch_global at a 97% identity threshold, then validate the resulting matrix with usearch -otutab_stats.

## When to use

Use when quality-filtered amplicon reads (e.g., temp/filtered.fa) need to be mapped to a reference OTU database at 97% identity to produce a valid tab-delimited OTU count table.

## When NOT to use

- Input is already a feature/OTU table — this skill maps raw filtered reads to a reference, it does not transform existing count matrices.
- The goal is de novo OTU clustering rather than reference-based mapping of reads to a pre-built database such as Greengenes 97% OTUs.
- Sequences are amino acid or require local alignment — VSEARCH does not support amino acid sequences or local alignments.

## Inputs

- Quality-filtered amplicon reads in FASTA format (temp/filtered.fa)
- USEARCH-compatible reference OTU database (e.g., Greengenes 97% OTUs, ${db}/gg/97_otus.fa)
- Thread count / identity threshold parameters (--id 0.97, --threads 12)

## Outputs

- Tab-delimited sample-by-OTU abundance table (result/gg/otutab.txt)
- Reported VSEARCH alignment (mapping) rate (~81.04% in the reference condition)

## How to apply

Load the quality-filtered read set (temp/filtered.fa) and the USEARCH-compatible reference library (e.g., ${db}/gg/97_otus.fa from Greengenes). Run VSEARCH global alignment with --usearch_global, --id 0.97, and multiple threads (12 in the reported condition), writing the OTU table with --otutabout to result/gg/otutab.txt; every filtered read is mapped to a reference Greengenes OTU at the 97% identity threshold. Record the alignment (mapping) rate reported in the VSEARCH output — the EasyAmplicon positive control achieves approximately 81.04% for this VSEARCH-alignment condition. Check the resulting table with usearch -otutab_stats to verify per-sample sequence counts and the validity of the sample-by-OTU matrix before downstream diversity analysis.

## Related tools

- **VSEARCH** (Performs the global alignment (--usearch_global) of filtered reads against the Greengenes 97% OTUs database at --id 0.97 with 12 threads, producing the OTU table via --otutabout.) — https://github.com/torognes/vsearch
- **USEARCH** (Used with -otutab_stats to verify per-sample sequence counts and the validity of the sample-by-OTU matrix.)
- **EasyAmplicon** (Pipeline providing the quality-filtered reads (temp/filtered.fa), database paths, and the positive-control workflow this skill reproduces.) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Supplies the scripts and reference databases (including USEARCH-compatible Greengenes) used by EasyAmplicon.) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
vsearch --usearch_global temp/filtered.fa --db ${db}/gg/97_otus.fa --otutabout result/gg/otutab.txt --id 0.97 --threads 12
```

## Evaluation signals

- result/gg/otutab.txt exists and is a well-formed tab-delimited OTU table.
- The computed read-mapping rate is approximately 81.04% for the VSEARCH --usearch_global --id 0.97 --threads 12 condition against Greengenes 97% OTUs.
- usearch -otutab_stats reports valid per-sample sequence counts with no malformed sample or OTU entries.
- The OTU table has samples as columns and OTUs as rows with integer abundances suitable for downstream rarefaction/diversity analysis.

## Limitations

- The ~81.04% alignment rate is specific to the EasyAmplicon positive-control dataset and Greengenes 97% OTUs reference; other datasets or databases will yield different rates.
- VSEARCH uses an optimal global aligner (full dynamic programming Needleman-Wunsch) versus USEARCH's heuristic seed-and-extend aligner, so results may differ between the two tools.
- VSEARCH does not support amino acid sequences or local alignments, restricting the skill to nucleotide global-alignment searches.

## Evidence

- [methods] VSEARCH global alignment of filtered reads against Greengenes at 97% identity: "vsearch --usearch_global temp/filtered.fa --db ${db}/gg/97_otus.fa --otutabout result/gg/otutab.txt --id 0.97 --threads 12"
- [methods] Expected alignment rate for the VSEARCH condition: [evidence span withheld — anchor status: not_found]
- [methods] Table validation step: [evidence span withheld — anchor status: not_found]
- [readme] VSEARCH aligner characteristics from README: [evidence span withheld — anchor status: external_doc]
- [readme] VSEARCH scope limitation: [evidence span withheld — anchor status: external_doc]
