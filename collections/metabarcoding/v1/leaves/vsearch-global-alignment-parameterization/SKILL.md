---
name: vsearch-global-alignment-parameterization
description: Use when quality-filtered amplicon reads (e.g. temp/filtered.fa) must
  be assigned to reference OTUs to build a sample-by-OTU table, and the expected alignment
  (mapping) rate must be quantified against a reference database like Greengenes 97%
  OTUs.
when_to_use_negative:
- Input is already a feature/OTU table or dereplicated representative sequences rather
  than quality-filtered per-sample reads
- Working with ITS/fungal data where Greengenes (16S bacterial) is the wrong reference
  — use UNITE instead
- Local (seed-and-extend) alignment semantics are required — VSEARCH does not support
  local alignments, only global alignment search
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: VSEARCH
  role: Performs the --usearch_global alignment of filtered reads against Greengenes
    97% OTUs at --id 0.97 with 12 threads and writes the OTU table
  repo: https://github.com/torognes/vsearch
- name: USEARCH
  role: Validates the resulting OTU table via 'usearch -otutab_stats' (per-sample
    counts, matrix validity); VSEARCH mirrors USEARCH v7 option names as a drop-in
    replacement
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyAmplicon
  role: 'Wraps the workflow: supplies the filtered reads, database paths, and expected
    81.04% mapping-rate benchmark for the positive control'
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:dcde30522b320904ab582aefb5989daa27761b4047bdb0d712126c36ce8cf04a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# vsearch-global-alignment-parameterization

## Summary

Parameterize VSEARCH's global alignment search (--usearch_global with a 97% identity threshold, multi-threaded) to map quality-filtered amplicon reads against a reference OTU database such as Greengenes 97% OTUs, producing an OTU count table and a measurable alignment rate.

## When to use

Use when quality-filtered amplicon reads (e.g. temp/filtered.fa) must be assigned to reference OTUs to build a sample-by-OTU table, and the expected alignment (mapping) rate must be quantified against a reference database like Greengenes 97% OTUs.

## When NOT to use

- Input is already a feature/OTU table or dereplicated representative sequences rather than quality-filtered per-sample reads
- Working with ITS/fungal data where Greengenes (16S bacterial) is the wrong reference — use UNITE instead
- Local (seed-and-extend) alignment semantics are required — VSEARCH does not support local alignments, only global alignment search

## Inputs

- Quality-filtered amplicon reads in FASTA format (temp/filtered.fa)
- USEARCH-compatible Greengenes 97% OTUs reference database (${db}/gg/97_otus.fa)

## Outputs

- Tab-delimited sample-by-OTU count table (result/gg/otutab.txt)
- VSEARCH console report including the alignment (mapping) rate (expected ~81.04%)

## How to apply

Load the quality-filtered read set (temp/filtered.fa) and the USEARCH-compatible Greengenes reference library (${db}/gg/97_otus.fa), then run VSEARCH global alignment with --usearch_global, --id 0.97 and --threads 12, writing the OTU table with --otutabout (e.g. result/gg/otutab.txt). Record the alignment (mapping) rate printed in the VSEARCH output; the EasyAmplicon positive-control dataset reaches approximately 81.04% under this configuration, so a substantially lower rate signals a problem upstream (over-aggressive filtering, wrong database format, or mismatched primers/region). Use the USEARCH-compatible Greengenes format because EasyAmplicon packages it for reference-based OTU picking and downstream PICRUSt use. Validate the resulting table with 'usearch -otutab_stats' to confirm per-sample sequence counts and a well-formed tab-delimited sample-by-OTU matrix. Rationale: VSEARCH uses an optimal global aligner (full Needleman-Wunsch) rather than USEARCH's heuristic seed-and-extend, giving improved sensitivity, and it reuses USEARCH v7 option names so the same command line works as a drop-in replacement.

## Related tools

- **VSEARCH** (Performs the --usearch_global alignment of filtered reads against Greengenes 97% OTUs at --id 0.97 with 12 threads and writes the OTU table) — https://github.com/torognes/vsearch
- **USEARCH** (Validates the resulting OTU table via 'usearch -otutab_stats' (per-sample counts, matrix validity); VSEARCH mirrors USEARCH v7 option names as a drop-in replacement) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyAmplicon** (Wraps the workflow: supplies the filtered reads, database paths, and expected 81.04% mapping-rate benchmark for the positive control) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
vsearch --usearch_global temp/filtered.fa --db ${db}/gg/97_otus.fa --otutabout result/gg/otutab.txt --id 0.97 --threads 12; usearch -otutab_stats result/gg/otutab.txt
```

## Evaluation signals

- VSEARCH reports an alignment/mapping rate of approximately 81.04% for the EasyAmplicon positive control under --id 0.97 with 12 threads
- result/gg/otutab.txt exists and is a well-formed tab-delimited sample-by-OTU matrix
- 'usearch -otutab_stats' succeeds and reports sane per-sample sequence counts
- Reference database is in USEARCH-compatible format (gg/97_otus.fa) and loads without format errors
- OTU identifiers in the table correspond to Greengenes 97% OTU references

## Limitations

- The ~81.04% mapping rate is specific to the EasyAmplicon positive-control dataset (GSA CRA002352 reads, Greengenes 97% OTUs); other datasets or regions will map differently
- VSEARCH does not support amino acid sequences or local alignments, unlike USEARCH
- The mapping rate is sensitive to upstream filtering choices (length 1200–1800 nt window, --fastq_maxee, primer removal via cutadapt), so a low rate may reflect filtering rather than the alignment step

## Evidence

- [other] VSEARCH global alignment with 97% identity threshold: "vsearch --usearch_global temp/filtered.fa --db ${db}/gg/97_otus.fa --otutabout result/gg/otutab.txt --id 0.97 --threads 12"
- [other] Expected mapping rate on the positive control: [evidence span withheld — anchor status: not_found]
- [other] Table validation step: [evidence span withheld — anchor status: not_found]
- [readme] VSEARCH global aligner rationale: [evidence span withheld — anchor status: external_doc]
- [intro] USEARCH-compatible Greengenes availability: "users can download Greengenes database (https://ftp.microbio.me/greengenes_release/current/) in USEARCH‐compatible format"
