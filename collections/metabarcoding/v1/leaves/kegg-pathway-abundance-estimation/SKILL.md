---
name: kegg-pathway-abundance-estimation
description: Use when the only input is 16S rDNA amplicon data (QIIME 2 feature table,
  BIOM/TSV plus representative sequences) and the research goal is to estimate metagenomic
  functional composition such as KEGG pathway abundances.
when_to_use_negative:
- The input is shotgun metagenomic data, where KO/EC and pathway abundances can be
  computed directly rather than predicted from 16S rDNA taxonomic composition.
- The input is not a 16S rDNA amplicon feature table (e.g., an already-derived KO/EC
  table or a non-amplicon dataset), so ASV placement and gene-family prediction are
  inapplicable.
- The goal is visualization of existing results (e.g., generating iTOL annotations
  with table2itol) rather than functional inference from amplicons.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt2
  role: 'Runs the pipeline.py workflow: places ASVs against reference genomes, predicts
    KO/EC gene-family abundances with marker-gene copy-number normalization, and infers
    KEGG pathway abundances.'
  repo: https://github.com/picrust/picrust2
- name: QIIME 2
  role: Produces the OTU/ASV feature table and representative 16S rDNA sequences that
    serve as input to PICRUSt2.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:e8d39056f20bb5df78fd5d1efb84708ece9322d1feaedfac293bcfa6eb480d44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/picrust/picrust2
  tools:
  - PICRUSt2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# kegg-pathway-abundance-estimation

## Summary

Predicts per-sample KEGG pathway abundance from 16S rDNA amplicon feature tables (OTU/ASV) using PICRUSt2, which places ASVs against reference genomes and infers gene-family (KO/EC) abundances before pathway-level inference.

## When to use

Use when the only input is 16S rDNA amplicon data (QIIME 2 feature table, BIOM/TSV plus representative sequences) and the research goal is to estimate metagenomic functional composition such as KEGG pathway abundances.

## When NOT to use

- The input is shotgun metagenomic data, where KO/EC and pathway abundances can be computed directly rather than predicted from 16S rDNA taxonomic composition.
- The input is not a 16S rDNA amplicon feature table (e.g., an already-derived KO/EC table or a non-amplicon dataset), so ASV placement and gene-family prediction are inapplicable.
- The goal is visualization of existing results (e.g., generating iTOL annotations with table2itol) rather than functional inference from amplicons.

## Inputs

- OTU/ASV abundance table (QIIME 2 feature table, BIOM, or TSV)
- Representative 16S rDNA sequences (FASTA) for the feature table

## Outputs

- pathway_abundance.tsv — per-sample KEGG pathway abundance table
- Predicted KO/EC gene-family abundance table (intermediate)

## How to apply

Load the OTU/ASV abundance table (QIIME 2 feature table, BIOM or TSV) together with its representative 16S rDNA sequences. Run PICRUSt2's pipeline.py on the table and sequences to place ASVs against reference genomes and predict gene-family (KO/EC) abundances, with marker-gene copy-number normalization applied during the prediction step. Convert the predicted gene-family abundances into KEGG pathway abundance estimates using PICRUSt2's pathway inference step. Write the per-sample KEGG pathway abundance table to pathway_abundance.tsv, retaining the predicted KO/EC abundance table as an intermediate output. The rationale, per the Liu et al. review, is that amplicon sequencing alone only yields taxonomic composition, so functional composition must be predicted rather than measured.

## Related tools

- **PICRUSt2** (Runs the pipeline.py workflow: places ASVs against reference genomes, predicts KO/EC gene-family abundances with marker-gene copy-number normalization, and infers KEGG pathway abundances.) — https://github.com/picrust/picrust2
- **QIIME 2** (Produces the OTU/ASV feature table and representative 16S rDNA sequences that serve as input to PICRUSt2.)

## Examples

```
picrust2_pipeline.py -s rep_seqs.fna -i feature_table.biom -o picrust2_out -p 4
```

## Evaluation signals

- pathway_abundance.tsv exists as a named output file after the run.
- pathway_abundance.tsv contains a non-negative predicted pathway abundance value for every sample present in the input OTU/ASV table.
- The predicted KO/EC gene-family abundance table is retained as an intermediate output.
- Marker-gene copy-number normalization was applied during gene-family abundance prediction.
- Each ASV was placed against the reference genomes before KO/EC prediction.

## Limitations

- 16S rDNA amplicon sequencing alone only yields taxonomic composition; KEGG pathway abundances here are predictions, not measurements, so they are estimates of metagenomic functional composition.
- Prediction quality depends on placing ASVs against the reference genomes used by PICRUSt2; ASVs absent from references may yield poor functional estimates.
- The review notes metatranscriptome, metaproteome, and metabolome methods are not widely accepted due to high cost and complex experimental/analysis methods, so predicted functional profiles are not a substitute for these measured modalities.

## Evidence

- [other] PICRUSt2 prediction mechanism from source card: [evidence span withheld — anchor status: not_found]
- [abstract] Review pipeline recommendation: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
- [readme] PICRUSt2 README documentation pointer: [evidence span withheld — anchor status: external_doc]
