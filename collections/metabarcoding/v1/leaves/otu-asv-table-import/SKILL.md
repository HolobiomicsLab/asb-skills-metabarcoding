---
name: otu-asv-table-import
description: Use when the input is an amplicon (16S rDNA) OTU/ASV abundance table
  from QIIME 2 or BIOM/TSV format and the analysis goal requires importing it into
  a functional-prediction or amplicon analysis pipeline.
when_to_use_negative:
- The data are shotgun metagenomic reads rather than 16S rDNA amplicon-derived OTU/ASV
  tables — use a metagenomic analysis pipeline instead of amplicon-based import.
- The table is already loaded in the target tool's native format and sample/feature
  identifiers are consistent — re-importing adds no information.
- The goal is only tree/annotation visualization of existing results (e.g., generating
  iTOL annotations with table2itol), which consumes spreadsheet tables rather than
  QIIME 2/BIOM feature tables.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: QIIME 2
  role: Source platform producing the feature table (and BIOM/TSV representations)
    that this skill imports
- name: PICRUSt2
  role: Consumes the imported OTU/ASV table and representative 16S sequences as input
    to predict gene-family and KEGG pathway abundances
  repo: https://github.com/picrust/picrust2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:a8ca1a0cbfcaa61d11a7bdac9112dcfd0337a08ddf0f2d55cab97435ed0659dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/picrust/picrust2
  tools:
  - PICRUSt2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# otu-asv-table-import

## Summary

Import an OTU/ASV abundance table (QIIME 2 feature table, BIOM/TSV) together with its representative 16S rDNA sequences so downstream functional prediction (e.g., PICRUSt2 KEGG pathway inference) and other amplicon analyses can operate on it.

## When to use

Use when the input is an amplicon (16S rDNA) OTU/ASV abundance table from QIIME 2 or BIOM/TSV format and the analysis goal requires importing it into a functional-prediction or amplicon analysis pipeline.

## When NOT to use

- The data are shotgun metagenomic reads rather than 16S rDNA amplicon-derived OTU/ASV tables — use a metagenomic analysis pipeline instead of amplicon-based import.
- The table is already loaded in the target tool's native format and sample/feature identifiers are consistent — re-importing adds no information.
- The goal is only tree/annotation visualization of existing results (e.g., generating iTOL annotations with table2itol), which consumes spreadsheet tables rather than QIIME 2/BIOM feature tables.

## Inputs

- QIIME 2 feature table (qza)
- OTU/ASV abundance table in BIOM format
- OTU/ASV abundance table in TSV format
- Representative 16S rDNA sequences for the OTUs/ASVs

## Outputs

- Imported feature/abundance table indexed by sample and OTU/ASV, ready for functional prediction (e.g., PICRUSt2 pipeline.py)

## How to apply

Load the OTU/ASV abundance table — as a QIIME 2 feature table, BIOM, or TSV file — alongside the corresponding representative 16S rDNA sequences. This step is the entry point of the recommended amplicon analysis pipeline in the review, and in the PICRUSt2 workflow it directly precedes placing ASVs against reference genomes and predicting gene-family (KO/EC) abundances with marker-gene copy-number normalization. Verify that every sample present in the abundance table is retained after import, since the downstream per-sample KEGG pathway abundance table (pathway_abundance.tsv) must cover the same samples. Rationale: amplicon sequencing alone yields only taxonomic composition, so importing the feature table correctly is what enables taxonomic compositions to be translated into predicted metagenomic functional composition.

## Related tools

- **QIIME 2** (Source platform producing the feature table (and BIOM/TSV representations) that this skill imports)
- **PICRUSt2** (Consumes the imported OTU/ASV table and representative 16S sequences as input to predict gene-family and KEGG pathway abundances) — https://github.com/picrust/picrust2

## Examples

```
qiime tools import --input-path otu_table.biom --type 'FeatureTable[Frequency]' --input-format BIOMV210Format --output-path feature-table.qza
```

## Evaluation signals

- The imported table retains every sample present in the original OTU/ASV table (sample set is unchanged after import).
- Feature/sample identifiers are consistent between the abundance table and the representative 16S sequence FASTA, so downstream ASV placement succeeds.
- Abundance values remain non-negative and in original scale after import (no unintended normalization or rounding).
- The downstream PICRUSt2 run accepts the imported table and produces pathway_abundance.tsv with a non-negative predicted pathway abundance for every input sample.

## Limitations

- Amplicon sequencing alone yields only taxonomic composition; imported tables support functional prediction (e.g., PICRUSt2) but not direct functional measurement.
- Review notes that many tools/pipelines make selection challenging for non-bioinformaticians; import must match the pipeline's expected format (QIIME 2 vs BIOM vs TSV).
- Representative 16S rDNA sequences must accompany the table for ASV-placement workflows; a table alone is insufficient for PICRUSt2.

## Evidence

- [other] Load the OTU/ASV abundance table (QIIME 2 feature table, BIOM/TSV) and its representative 16S rDNA sequences.: [evidence span withheld — anchor status: not_found]
- [intro] The next-generation microbiome analysis pipeline QIIME 2, a reproducible, interactive, efficient, community-supported platform was recently published: "The next-generation microbiome analysis pipeline QIIME 2, a reproducible, interactive, efficient, community-supported platform was recently published"
- [abstract] we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases"
- [intro] The appropriate method(s) should be selected based on sample types and research goals.: "The appropriate method(s) should be selected based on sample types and research goals."
- [readme] picrust/picrust2: [evidence span withheld — anchor status: external_doc]
