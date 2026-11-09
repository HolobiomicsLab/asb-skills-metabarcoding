---
name: feature-table-dimension-validation
description: Use when a feature table (otutab.txt, rarefied or trimmed variants) has
  just been generated or filtered (e.g., non-bacterial removal, rarefaction, otutab_trim)
  and must be checked for structural integrity before diversity analysis or visualization.
when_to_use_negative:
- The input is not a feature table but raw reads or reference sequences (temp/filtered.fa,
  otus.fa) — structural checks there use FASTQ/FASTA tools, not csvtk
- The table has not yet been filtered (result/raw/otutab.txt still contains non-bacterial
  lineages) — validate after otutab_filter_nonBac.R, not before, since the raw table
  may legitimately differ in feature rows
- Only a sample-count mismatch needs checking while no transformation has occurred
  — trivially compare metadata row count to table header instead of invoking the full
  validation workflow
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: csvtk
  role: Used to compute statistics on the tab-separated feature table (csvtk -t stat)
    to verify the number of columns matches the samples in the metadata
  repo: https://github.com/YongxinLiu/EasyMicrobiome
- name: VSEARCH
  role: Generates the raw feature table whose dimensions are validated, via vsearch
    --usearch_global mapping of quality-filtered reads to the OTU reference
  repo: https://github.com/torognes/vsearch
- name: EasyAmplicon
  role: Hosts the pipeline script and otutab_filter_nonBac.R step that produces the
    cleaned table subject to dimension validation
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:919121bd0e5468aa1178d4e8f9f42c9fc60635d95278a96c74640b81a66da254
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyMicrobiome
  tools:
  - EasyAmplicon
  - VSEARCH
  - csvtk
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# feature-table-dimension-validation

## Summary

A quality-control step in the EasyAmplicon pipeline that verifies the cleaned per-sample OTU/ASV feature table (e.g., result/otutab.txt) has the expected tabular shape — column count matching the samples declared in the metadata — before downstream diversity analysis.

## When to use

Use when a feature table (otutab.txt, rarefied or trimmed variants) has just been generated or filtered (e.g., non-bacterial removal, rarefaction, otutab_trim) and must be checked for structural integrity before diversity analysis or visualization.

## When NOT to use

- The input is not a feature table but raw reads or reference sequences (temp/filtered.fa, otus.fa) — structural checks there use FASTQ/FASTA tools, not csvtk
- The table has not yet been filtered (result/raw/otutab.txt still contains non-bacterial lineages) — validate after otutab_filter_nonBac.R, not before, since the raw table may legitimately differ in feature rows
- Only a sample-count mismatch needs checking while no transformation has occurred — trivially compare metadata row count to table header instead of invoking the full validation workflow

## Inputs

- Cleaned tab-separated feature table (result/otutab.txt; rows = OTUs/ASVs, columns = samples)
- Sample metadata file defining the expected sample set

## Outputs

- Dimension report (row/column counts) from csvtk -t stat
- Pass/fail determination that table columns match metadata sample count

## How to apply

After constructing the raw per-sample table with vsearch --usearch_global (temp/filtered.fa mapped to result/raw/otus.fa at --id 0.97) and cleaning it with EasyMicrobiome's otutab_filter_nonBac.R (dropping Eukarya, Chloroplasts, Mitochondria, and Archaea from result/raw/otutab.txt and result/raw/otus.sintax), run csvtk -t stat on the cleaned table to report dimensions. Confirm that the number of columns equals the number of samples listed in the metadata file; a mismatch signals lost samples or mis-split tabbed fields. Re-check dimensions after any later transformation (e.g., rarefaction with vegan, or usearch -otutab_trim -min_otu_freq 0.005), since trimming can drop OTUs/features and change the table shape. Only proceed to diversity analysis, STAMP, or PICRUSt2 once sample-column count is verified against metadata.

## Related tools

- **csvtk** (Used to compute statistics on the tab-separated feature table (csvtk -t stat) to verify the number of columns matches the samples in the metadata) — https://github.com/YongxinLiu/EasyMicrobiome
- **VSEARCH** (Generates the raw feature table whose dimensions are validated, via vsearch --usearch_global mapping of quality-filtered reads to the OTU reference) — https://github.com/torognes/vsearch
- **EasyAmplicon** (Hosts the pipeline script and otutab_filter_nonBac.R step that produces the cleaned table subject to dimension validation) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
csvtk -t stat result/otutab.txt
```

## Evaluation signals

- csvtk -t stat column count on result/otutab.txt equals the number of samples in the metadata file
- Every sample ID present in metadata appears as a column header in the cleaned table (no sample loss through filtering)
- Row count decreases or stays equal relative to result/raw/otutab.txt only through discarded non-bacterial features (cross-check with result/raw/otus.sintax.discard)
- Table remains valid tab-separated format after rarefaction (vegan) and trimming (usearch -otutab_trim -min_otu_freq 0.005), with dimensions re-verified at each stage
- Downstream diversity-analysis scripts consume the table without parsing errors, confirming schema integrity

## Limitations

- Validation only checks structural dimensions (column/sample counts); it does not confirm taxonomic correctness of removed lineages, which relies on SINTAX annotations at --sintax_cutoff 0.1 against rdp_16s_v18
- The check is specific to tab-delimited EasyAmplicon outputs (otutab.txt family); BIOM-format tables or QIIME 2 artifacts are out of scope
- Fungal/ITS analyses using UNITE instead of 16S databases may intentionally retain Eukarya, making the non-bacterial filtering assumption behind this validation invalid

## Evidence

- [methods] Remove Chloroplasts, Mitochondria, and Non-Bacteria: "**Remove Chloroplasts, Mitochondria, and Non-Bacteria**: Typically, an R script (e.g., `otutab_filter_nonBac.R`) is used to filter out Eukarya, Chloroplasts, Mitochondria, and Archaea"
- [methods] OTU trimming changes table shape: "usearch -otutab_trim result/otutab_rare.txt -min_otu_freq 0.005 -output tree/otutab.txt"
- [intro] Pipeline context for table construction: "These tools work together to assess sequence quality, split barcoded sequences, remove sequencing background, and identify amplicon sequence variants (ASVs) and operational taxonomic units (OTUs)."
