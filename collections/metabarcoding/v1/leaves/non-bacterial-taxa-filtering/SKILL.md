---
name: non-bacterial-taxa-filtering
description: Use when a 16S amplicon feature table (e.g. result/raw/otutab.txt) contains
  contaminant or host-associated lineages such as chloroplasts, mitochondria, Eukarya,
  or Archaea that must be removed before diversity analysis.
when_to_use_negative:
- Input is already a cleaned feature table (result/otutab.txt) with non-bacterial
  taxa removed
- Analysis targets eukaryotic (ITS/UNITE) or archaeal amplicons where those lineages
  are the intended targets, not contaminants
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: VSEARCH
  role: Generates the raw feature table (--usearch_global --id 0.97) and SINTAX taxonomy
    annotations (--sintax --sintax_cutoff 0.1) that the filter consumes
  repo: https://github.com/torognes/vsearch
- name: EasyAmplicon
  role: Pipeline providing the workflow context and file conventions (result/raw/otutab.txt,
    result/raw/otus.sintax) for the filtering step
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Provides the R script otutab_filter_nonBac.R that discards Eukarya, Chloroplasts,
    Mitochondria and Archaea lineages
  repo: https://github.com/YongxinLiu/EasyMicrobiome
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:8a0c629fdc0d12de9f42b4d05e7f5149093a8be1b7eaea519d098dc1a12f3065
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# non-bacterial-taxa-filtering

## Summary

Filter non-bacterial sequences (Eukarya, Chloroplasts, Mitochondria, Archaea) from the raw OTU/ASV feature table and SINTAX taxonomy annotations in the EasyAmplicon pipeline, using the EasyMicrobiome R script otutab_filter_nonBac.R to produce cleaned downstream outputs.

## When to use

Use when a 16S amplicon feature table (e.g. result/raw/otutab.txt) contains contaminant or host-associated lineages such as chloroplasts, mitochondria, Eukarya, or Archaea that must be removed before diversity analysis.

## When NOT to use

- Input is already a cleaned feature table (result/otutab.txt) with non-bacterial taxa removed
- Analysis targets eukaryotic (ITS/UNITE) or archaeal amplicons where those lineages are the intended targets, not contaminants

## Inputs

- Raw per-sample feature table result/raw/otutab.txt (vsearch --usearch_global output at --id 0.97)
- Tabbed SINTAX taxonomy annotations result/raw/otus.sintax (vsearch --sintax vs RDP 16S v18, cutoff 0.1)
- Sample metadata (metadata.txt) for column-count verification

## Outputs

- Cleaned feature table result/otutab.txt with non-bacterial lineages removed
- Discard list result/raw/otus.sintax.discard identifying removed sequences

## How to apply

First construct the raw per-sample feature table by mapping quality-filtered merged sequences (temp/filtered.fa) to the OTU/ASV reference (result/raw/otus.fa) with vsearch --usearch_global at --id 0.97, and annotate the OTU sequences with vsearch --sintax against the RDP 16S v18 database (${db}/usearch/rdp_16s_v18.fa) at --sintax_cutoff 0.1, writing result/raw/otus.sintax. Then run the EasyMicrobiome R script otutab_filter_nonBac.R with the raw feature table and the sintax annotations as inputs; the script discards lineages assigned to Eukarya, Chloroplasts, Mitochondria, and Archaea. The cleaned feature table is written to result/otutab.txt and the removed sequences are listed in result/raw/otus.sintax.discard. Finally, verify the cleaned table with csvtk -t stat, confirming the column count matches the number of samples in the metadata.

## Related tools

- **VSEARCH** (Generates the raw feature table (--usearch_global --id 0.97) and SINTAX taxonomy annotations (--sintax --sintax_cutoff 0.1) that the filter consumes) — https://github.com/torognes/vsearch
- **EasyAmplicon** (Pipeline providing the workflow context and file conventions (result/raw/otutab.txt, result/raw/otus.sintax) for the filtering step) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Provides the R script otutab_filter_nonBac.R that discards Eukarya, Chloroplasts, Mitochondria and Archaea lineages) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
Rscript ${soft}/otutab_filter_nonBac.R -i result/raw/otutab.txt -d result/raw/otus.sintax -o result/otutab.txt
```

## Evaluation signals

- csvtk -t stat on result/otutab.txt reports a number of columns equal to the number of samples in the metadata
- result/raw/otus.sintax.discard exists and contains the removed sequences, confirming the filter ran and recorded its decisions
- No rows in the cleaned table have taxonomy annotations containing Eukarya, Chloroplasts, Mitochondria, or Archaea lineages
- Cleaned table retains the same sample identifiers as the raw table (only feature rows are removed, not sample columns)
- SINTAX annotations were produced with --sintax_cutoff 0.1, so bootstrap confidence of assigned lineages meets the script's expectations

## Limitations

- Filtering depends on the quality of SINTAX annotations; sequences unassigned at the relevant rank with the 0.1 bootstrap cutoff may not be classified as non-bacterial and could pass through
- The default RDP 16S v18 reference is bacteria-oriented; other marker genes or databases (SILVA, UNITE, Greengenes) may require different filtering logic
- No specific version numbers or changelog details for the filter script were documented in the discussion of the protocol

## Evidence

- [methods] Typically, an R script (e.g., `otutab_filter_nonBac.R`) is used to filter out Eukarya, Chloroplasts, Mitochondria, and Archaea: "**Remove Chloroplasts, Mitochondria, and Non-Bacteria**: Typically, an R script (e.g., `otutab_filter_nonBac.R`) is used to filter out Eukarya, Chloroplasts, Mitochondria, and Archaea"
- [other] The protocol provides an R script (otutab_filter_nonBac.R) to exclude nonbacterial/archaeal sequences, such as plastids and mitochondria: [evidence span withheld — anchor status: not_found]
- [methods] EasyMicrobiome include scripts and databases https://github.com/YongxinLiu/EasyMicrobiome: "EasyMicrobiome include scripts and databases https://github.com/YongxinLiu/EasyMicrobiome"
- [readme] Popular software, scripts and database annotation for EasyAmplicon and EasyMetagenome: [evidence span withheld — anchor status: not_found]
