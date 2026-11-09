---
name: functional-prediction-input-preparation
description: Use when a filtered amplicon sequence set (e.g., temp/filtered.fa) must
  be converted into a reference-based Greengenes OTU table as input for functional
  prediction, and the alignment rate needs verification.
when_to_use_negative:
- Input is already a feature/OTU table — no sequence alignment is needed
- Amplicons target ITS/fungi, where the UNITE database (not Greengenes 97% 16S OTUs)
  is the appropriate reference
- ASV-based (de novo, non-reference) analysis is required rather than Greengenes-reference
  OTU assignment for functional prediction
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: VSEARCH
  role: Performs the vsearch --usearch_global global alignment of filtered reads against
    the Greengenes 97% OTUs database at --id 0.97 to build the OTU table
  repo: https://github.com/torognes/vsearch
- name: USEARCH
  role: Used for usearch -otutab_stats to summarize the OTU table and compute/verify
    the alignment rate
- name: EasyAmplicon
  role: Pipeline that orchestrates the functional-prediction alignment step and packages
    the expected inputs/outputs (temp/filtered.fa, result/gg/otutab.txt)
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Supplies the pre-packaged reference databases, including the Greengenes 97%
    OTUs file used for PICRUSt-style functional prediction
  repo: https://github.com/YongxinLiu/EasyMicrobiome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:cfb281ed3b8a3b501f6c2739457cd173e153ebc784bc32b26e6e436c0c974702
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# functional-prediction-input-preparation

## Summary

Prepares the OTU table that downstream functional-prediction tools (e.g., PICRUSt/BugBase) consume by globally aligning quality-filtered amplicon reads against the Greengenes 97% OTUs reference with vsearch --usearch_global, and validates the resulting alignment rate against the reported 80.0% benchmark.

## When to use

Use when a filtered amplicon sequence set (e.g., temp/filtered.fa) must be converted into a reference-based Greengenes OTU table as input for functional prediction, and the alignment rate needs verification.

## When NOT to use

- Input is already a feature/OTU table — no sequence alignment is needed
- Amplicons target ITS/fungi, where the UNITE database (not Greengenes 97% 16S OTUs) is the appropriate reference
- ASV-based (de novo, non-reference) analysis is required rather than Greengenes-reference OTU assignment for functional prediction

## Inputs

- Quality-filtered amplicon FASTA (temp/filtered.fa) from the upstream QC step
- Greengenes 97% OTUs reference database (${db}/gg/97_otus.fa) from the EasyMicrobiome database directory
- Sample metadata file for column-count cross-check

## Outputs

- Reference-based Greengenes OTU table (result/gg/otutab.txt)
- OTU table summary statistics from usearch -otutab_stats (per-sample sequence counts)
- Overall USEARCH/VSEARCH alignment rate (reported value: 80.0%)

## How to apply

Load the quality-filtered sequence set temp/filtered.fa produced by the upstream QC step and run a global alignment of all reads against the Greengenes 97% OTUs database (${db}/gg/97_otus.fa) at 97% identity (--id 0.97) using vsearch --usearch_global with multithreading (4 threads), writing the reference OTU table to result/gg/otutab.txt. VSEARCH is used here because it applies an optimal global aligner (full dynamic programming Needleman-Wunsch) rather than a heuristic seed-and-extend search. Inspect the table with usearch -otutab_stats to summarize per-sample sequence counts, then compute the overall alignment rate (fraction of reads assigned to Greengenes OTUs) and confirm it matches the reported 80.0% (metric_usearch_align_rate) within reasonable tolerance. Verify the OTU table's sample columns match the metadata file, since Greengenes 97% OTUs is the pre-packaged reference intended for functional prediction (e.g., PICRUSt) in EasyAmplicon.

## Related tools

- **VSEARCH** (Performs the vsearch --usearch_global global alignment of filtered reads against the Greengenes 97% OTUs database at --id 0.97 to build the OTU table) — https://github.com/torognes/vsearch
- **USEARCH** (Used for usearch -otutab_stats to summarize the OTU table and compute/verify the alignment rate)
- **EasyAmplicon** (Pipeline that orchestrates the functional-prediction alignment step and packages the expected inputs/outputs (temp/filtered.fa, result/gg/otutab.txt)) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Supplies the pre-packaged reference databases, including the Greengenes 97% OTUs file used for PICRUSt-style functional prediction) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
vsearch --usearch_global temp/filtered.fa --db ${db}/gg/97_otus.fa --id 0.97 --threads 4 --otutabout result/gg/otutab.txt
```

## Evaluation signals

- Computed USEARCH-alignment rate matches the reported 80.0% (metric_usearch_align_rate) within reasonable tolerance
- result/gg/otutab.txt exists and parses as a valid OTU table
- Column count of result/gg/otutab.txt matches the number of samples in the metadata file
- usearch -otutab_stats reports sensible per-sample sequence counts (non-zero, consistent with upstream filtered reads)
- Alignment was run at the specified --id 0.97 identity threshold against ${db}/gg/97_otus.fa with 4 threads

## Limitations

- The 80.0% alignment rate is a positive-control benchmark; different datasets or reference database versions will yield different rates
- VSEARCH uses an optimal global aligner versus USEARCH's heuristic seed-and-extend aligner, so rates may differ slightly between the two tools
- Greengenes 97% OTUs is specifically pre-packaged for PICRUSt-style functional prediction; other taxonomic steps use RDP/SILVA/UNITE instead

## Evidence

- [methods] Run a global alignment of all filtered reads against the Greengenes 97% OTUs database (${db}/gg/97_otus.fa) at 97% identity (--id 0.97) using vsearch --usearch_global: [evidence span withheld — anchor status: not_found]
- [methods] Pre-packaged reference databases (e.g., RDP/SILVA/UNITE for taxonomy, Greengenes 97% OTUs for PICRUSt): "Pre-packaged reference databases (e.g., RDP/SILVA/UNITE for taxonomy, Greengenes 97% OTUs for PICRUSt)"
- [readme] VSEARCH uses an optimal global aligner (full dynamic programming Needleman-Wunsch): [evidence span withheld — anchor status: external_doc]
- [results] The USEARCH-based alignment with 4 threads results in an 80.0% alignment rate.: [evidence span withheld — anchor status: not_found]
