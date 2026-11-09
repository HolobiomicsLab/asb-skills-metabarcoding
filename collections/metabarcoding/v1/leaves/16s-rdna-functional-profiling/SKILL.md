---
name: 16s-rdna-functional-profiling
description: Use when the input is a 16S rDNA amplicon OTU/ASV abundance table (QIIME
  2 feature table, BIOM/TSV) with representative sequences, and the analysis goal
  is predicted KEGG pathway abundances rather than taxonomy alone.
when_to_use_negative:
- You already have shotgun metagenomic data — the review recommends a separate metagenomic
  analysis pipeline; functional profiling from reads does not require 16S-based prediction
- Input is raw amplicon reads rather than a processed feature table — first produce
  an OTU/ASV abundance table (e.g., in QIIME 2) before applying this skill
- The question concerns taxonomic composition, alpha/beta diversity, or visualization
  of amplicon results rather than predicted functional pathways
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt2
  role: 'Core prediction engine: places ASVs against reference genomes, predicts KO/EC
    gene-family abundances with copy-number normalization, and infers KEGG pathway
    abundances'
  repo: https://github.com/picrust/picrust2
- name: QIIME 2
  role: Upstream amplicon pipeline that produces the feature table and representative
    16S rDNA sequences consumed by PICRUSt2
- name: table2itol
  role: Optional downstream visualization companion from the same review for generating
    iTOL annotations from result tables
  repo: https://github.com/mgoeker/table2itol
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:f918df3b2ef1c9428e8e777c49c3238070dc85f8843a5e200c94f22bb69bc982
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/picrust/picrust2
  tools:
  - PICRUSt2
  - QIIME 2
  - table2itol
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# 16s-rdna-functional-profiling

## Summary

Predicts per-sample metagenomic functional composition (KEGG pathways) from 16S rDNA amplicon OTU/ASV feature tables using PICRUSt2, bridging the gap where amplicon sequencing alone only yields taxonomic composition. This is the recommended approach when a reviewer's study design asks for functional profiles from marker-gene data.

## When to use

Use when the input is a 16S rDNA amplicon OTU/ASV abundance table (QIIME 2 feature table, BIOM/TSV) with representative sequences, and the analysis goal is predicted KEGG pathway abundances rather than taxonomy alone.

## When NOT to use

- You already have shotgun metagenomic data — the review recommends a separate metagenomic analysis pipeline; functional profiling from reads does not require 16S-based prediction
- Input is raw amplicon reads rather than a processed feature table — first produce an OTU/ASV abundance table (e.g., in QIIME 2) before applying this skill
- The question concerns taxonomic composition, alpha/beta diversity, or visualization of amplicon results rather than predicted functional pathways

## Inputs

- OTU/ASV abundance table (QIIME 2 feature table, BIOM or TSV format)
- Representative 16S rDNA sequences (e.g., QIIME 2 rep-seqs FASTA)

## Outputs

- pathway_abundance.tsv — per-sample predicted KEGG pathway abundance table
- Predicted gene-family (KO/EC) abundance table retained as an intermediate

## How to apply

Load the OTU/ASV abundance table (QIIME 2 feature table, BIOM/TSV) together with its representative 16S rDNA sequences. Run PICRUSt2's pipeline.py on the table and sequences to place ASVs against reference genomes and predict gene-family (KO/EC) abundances with marker-gene copy-number normalization. Convert the predicted gene-family abundances into KEGG pathway abundance estimates using PICRUSt2's pathway inference step (MinPath source code is distributed inside the picrust2 repository). Retain the predicted KO/EC abundance table as an intermediate and write the final per-sample KEGG pathway abundance table to pathway_abundance.tsv. Consult the PICRUSt2 wiki for installation and run documentation before executing, since the GitHub README directs all documentation there.

## Related tools

- **PICRUSt2** (Core prediction engine: places ASVs against reference genomes, predicts KO/EC gene-family abundances with copy-number normalization, and infers KEGG pathway abundances) — https://github.com/picrust/picrust2
- **QIIME 2** (Upstream amplicon pipeline that produces the feature table and representative 16S rDNA sequences consumed by PICRUSt2)
- **table2itol** (Optional downstream visualization companion from the same review for generating iTOL annotations from result tables) — https://github.com/mgoeker/table2itol

## Examples

```
picrust2_pipeline.py -s rep_seqs.fna -i otu_table.biom -o picrust2_out -p 8
```

## Evaluation signals

- pathway_abundance.tsv exists as a named output file after running the pipeline
- pathway_abundance.tsv contains a non-negative predicted pathway abundance value for every sample present in the input OTU/ASV table
- The intermediate predicted KO/EC gene-family abundance table is retained and covers the same sample set as the pathway table
- Predicted gene-family abundances reflect marker-gene copy-number normalization (16S copy number correction applied during ASV placement)
- Every pathway row maps to a KEGG pathway identifier produced by PICRUSt2's pathway inference step

## Limitations

- Functional composition is predicted, not measured — amplicon sequencing alone only yields taxonomic composition, so results are inference-based estimates dependent on reference genome placement
- PICRUSt2's documentation lives in the GitHub wiki rather than the README; the README directs users to consult the wiki and FAQ before running and to use the PICRUSt users Google group rather than GitHub issues for questions
- Metagenomic functional profiles obtained from actual shotgun data are a more direct measurement; this skill substitutes prediction when only 16S data exist

## Evidence

- [results] The review describes PICRUSt2 as a functional-prediction mechanism that takes an arbitrary OTU/ASV table as input and directly predicts metagenomic functional composition from 16S rDNA amplicon data.: [evidence span withheld — anchor status: not_found]
- [abstract] Review scope statement supporting the recommended amplicon pipeline context.: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
- [readme] PICRUSt2 README pointing to the wiki for installation and run documentation.: [evidence span withheld — anchor status: external_doc]
- [intro] QIIME 2 introduced as the next-generation microbiome analysis pipeline producing feature tables.: "The next-generation microbiome analysis pipeline QIIME 2, a reproducible, interactive, efficient, community-supported platform was recently published"
