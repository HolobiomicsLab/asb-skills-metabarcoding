---
name: picrust2-metagenome-prediction
description: Use when the input is a 16S rDNA amplicon OTU/ASV abundance table (QIIME
  2 feature table, BIOM/TSV) plus representative sequences, and the goal is predicted
  KEGG pathway or KO/EC functional composition without shotgun metagenomics.
when_to_use_negative:
- You already have shotgun metagenomic data — directly profile functions instead of
  predicting them from 16S taxonomic composition
- The input is not an amplicon OTU/ASV feature table (e.g., it is already a gene-family
  or pathway abundance table)
- Your research question requires true functional profiling rather than predictions
  inferred from marker-gene copy-number-normalized taxonomy
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt2
  role: 'Core tool: places ASVs against reference genomes, predicts KO/EC gene-family
    abundances with copy-number normalization, and infers KEGG pathway abundances
    (pathway_abundance.tsv)'
  repo: https://github.com/picrust/picrust2
- name: QIIME 2
  role: Upstream amplicon analysis platform that produces the feature table and representative
    sequences fed into PICRUSt2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:28bd803977e680721d9bbe07c4b1c3ff1a43bdc9b10f36a4777b7ecfed8d4f9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/picrust/picrust2
  tools:
  - PICRUSt2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# picrust2-metagenome-prediction

## Summary

Predicts metagenomic functional composition (KO/EC gene families and KEGG pathway abundances) from a 16S rDNA amplicon OTU/ASV feature table using PICRUSt2, bridging the gap between taxonomic-only amplicon data and functional profiles.

## When to use

Use when the input is a 16S rDNA amplicon OTU/ASV abundance table (QIIME 2 feature table, BIOM/TSV) plus representative sequences, and the goal is predicted KEGG pathway or KO/EC functional composition without shotgun metagenomics.

## When NOT to use

- You already have shotgun metagenomic data — directly profile functions instead of predicting them from 16S taxonomic composition
- The input is not an amplicon OTU/ASV feature table (e.g., it is already a gene-family or pathway abundance table)
- Your research question requires true functional profiling rather than predictions inferred from marker-gene copy-number-normalized taxonomy

## Inputs

- OTU/ASV abundance table (QIIME 2 feature table, BIOM or TSV)
- Representative 16S rDNA sequences (e.g., rep_seqs.fna)

## Outputs

- pathway_abundance.tsv (per-sample predicted KEGG pathway abundance table)
- Predicted KO/EC gene-family abundance table (intermediate)

## How to apply

Load the OTU/ASV abundance table (QIIME 2 feature table, BIOM or TSV) together with its representative 16S rDNA sequences. Run PICRUSt2's pipeline (picrust2_pipeline.py) to place ASVs against reference genomes and predict gene-family (KO/EC) abundances with marker-gene copy-number normalization. Convert the predicted gene-family abundances into KEGG pathway abundance estimates via PICRUSt2's pathway inference step (MinPath source code is bundled in the picrust2 repository). Retain the predicted KO/EC abundance table as an intermediate and write the per-sample KEGG pathway abundance table to pathway_abundance.tsv. The review recommends PICRUSt2 within its amplicon analysis pipeline guidance for researchers selecting appropriate tools for microbiome data.

## Related tools

- **PICRUSt2** (Core tool: places ASVs against reference genomes, predicts KO/EC gene-family abundances with copy-number normalization, and infers KEGG pathway abundances (pathway_abundance.tsv)) — https://github.com/picrust/picrust2
- **QIIME 2** (Upstream amplicon analysis platform that produces the feature table and representative sequences fed into PICRUSt2)

## Examples

```
picrust2_pipeline.py -s rep_seqs.fna -i otu_table.biom -o picrust2_out -p 4 && picrust2_out/pathway_abundance.tsv
```

## Evaluation signals

- pathway_abundance.tsv exists as the named output file
- pathway_abundance.tsv contains a non-negative predicted pathway abundance value for every sample present in the input OTU/ASV table
- The predicted KO/EC gene-family abundance table is retained as an intermediate output
- The set of samples in pathway_abundance.tsv matches the sample IDs of the input feature table (no dropped or added samples)
- Predicted gene-family abundances show evidence of marker-gene copy-number normalization (16S copy-number corrected ASV placements against reference genomes)

## Limitations

- Predictions are inferred from 16S rDNA taxonomic composition, not measured; amplicon sequencing alone only yields taxonomic composition, so pathway abundances are estimates
- PICRUSt2 documentation is distributed via the GitHub wiki rather than the README; users are directed to consult the wiki FAQ and users Google group before filing issues
- PICRUSt2 bundles MinPath source code distributed under the GNU General Public License, which may matter for licensing-sensitive reuse

## Evidence

- [intro] The next-generation microbiome analysis pipeline QIIME 2, a reproducible, interactive, efficient, community-supported platform was recently published (Bolyen et al., 2019).: "The next-generation microbiome analysis pipeline QIIME 2, a reproducible, interactive, efficient, community-supported platform was recently published (Bolyen et al., 2019)."
- [readme] Please see the [PICRUSt2 wiki](https://github.com/picrust/picrust2/wiki) for the documentation and tutorials.: [evidence span withheld — anchor status: external_doc]
- [abstract] Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools.: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
- [readme] Note that this repository contains the source code for MinPath, which is distributed under the GNU General Public License.: [evidence span withheld — anchor status: external_doc]
