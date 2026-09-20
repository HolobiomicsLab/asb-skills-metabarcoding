---
name: family-level-taxonomic-collapsing-qiime2
description: Use when verifying or reproducing family-level taxonomic dominance claims
  from a Tourmaline/QIIME 2 DADA2 run, e.g. checking that Sporichthyaceae and SAR11
  Clade III top the 5.0-μm fraction and Cyanobiaceae tops the 0.22-μm fraction.
when_to_use_negative:
- Input is already a collapsed or family-level feature table rather than an ASV-level
  Tourmaline artifact
- Taxonomy was assigned with a different tool/database (e.g. Deblur single-end, PR2,
  or Anacapa's Bowtie 2/BLCA) where family rank labels or SILVA 138.1 lineages differ
- 'The goal is species- or genus-level resolution; collapse_taxalevel: 7 discards
  finer ranks'
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Tourmaline
  role: Produces the taxonomy step outputs (taxonomy.qzv, taxa_barplot.qzv) whose
    family-level relative abundances are collapsed and verified
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Runs the naive-bayes taxonomy classification against SILVA 138.1 and provides
    the taxa collapse/plugin used to collapse ASVs at level 7
- name: QIIME 2 viewer
  role: Used to open taxonomy.qzv and taxa_barplot.qzv and inspect family-level relative
    abundances per sample
- name: DADA2
  role: Denoises the paired-end Lake Erie 16S rRNA reads into the ASV feature table
    being collapsed
  repo: github.com/benjjneb/dada2
- name: Python
  role: Group the collapsed family table by size-fraction metadata, compute relative
    abundances, and write the verification TSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:a2d06307d268bef675c023e90d99a1a599fed5122afb657240d7e6f5528a8f81
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Tourmaline
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# family-level-taxonomic-collapsing-qiime2

## Summary

Collapse QIIME 2 ASV feature tables produced by the Tourmaline taxonomy step (naive-bayes classifier against SILVA 138.1) to family level (taxa_ranks level 7) and compute per-sample or per-metadata-group relative abundances to verify which bacterial families dominate a dataset, e.g. Sporichthyaceae and SAR11 Clade III in the 5.0-μm fraction and Cyanobiaceae in the 0.22-μm fraction of the Western Lake Erie 16S rRNA test dataset.

## When to use

Use when verifying or reproducing family-level taxonomic dominance claims from a Tourmaline/QIIME 2 DADA2 run, e.g. checking that Sporichthyaceae and SAR11 Clade III top the 5.0-μm fraction and Cyanobiaceae tops the 0.22-μm fraction.

## When NOT to use

- Input is already a collapsed or family-level feature table rather than an ASV-level Tourmaline artifact
- Taxonomy was assigned with a different tool/database (e.g. Deblur single-end, PR2, or Anacapa's Bowtie 2/BLCA) where family rank labels or SILVA 138.1 lineages differ
- The goal is species- or genus-level resolution; collapse_taxalevel: 7 discards finer ranks

## Inputs

- Tourmaline unfiltered DADA2-PE feature table (ASV counts) and representative-sequence taxonomy assignments (QIIME 2 artifacts)
- config_03_taxonomy.yaml configuration file (classify_method: naive-bayes, database silva-138_1, collapse_taxalevel: 7)
- Sample metadata file with size-fraction column (5.0-um vs 0.22-um)
- Interactive visualizations taxonomy.qzv and taxa_barplot.qzv

## Outputs

- Family-level (level-7) collapsed abundance table with per-fraction relative abundances
- TSV file of verified family-level abundances with pass/fail verification flags for each expected dominance pattern

## How to apply

Load the taxonomy table and representative-sequence taxonomy assignments from the unfiltered DADA2-PE Tourmaline run (config_03_taxonomy.yaml, classify_method naive-bayes against SILVA 138.1). Inspect the interactive visualizations taxonomy.qzv and taxa_barplot.qzv with QIIME 2 viewer, since the taxa_ranks configuration includes family level. Collapse ASV counts at family level (collapse_taxalevel: 7) with QIIME 2's taxa collapse, then group by size-fraction sample metadata (5.0-um vs 0.22-um) and compute relative abundances per family. Check the 5.0-μm fraction for Sporichthyaceae and SAR11 Clade III dominance and the 0.22-μm fraction for Cyanobiaceae dominance (chloroplast-dominated, with Microcystaceae lower). Write verified family-level abundances and pass/fail flags to a TSV for downstream validation.

## Related tools

- **Tourmaline** (Produces the taxonomy step outputs (taxonomy.qzv, taxa_barplot.qzv) whose family-level relative abundances are collapsed and verified) — github.com/aomlomics/tourmaline
- **QIIME 2** (Runs the naive-bayes taxonomy classification against SILVA 138.1 and provides the taxa collapse/plugin used to collapse ASVs at level 7)
- **QIIME 2 viewer** (Used to open taxonomy.qzv and taxa_barplot.qzv and inspect family-level relative abundances per sample)
- **DADA2** (Denoises the paired-end Lake Erie 16S rRNA reads into the ASV feature table being collapsed) — github.com/benjjneb/dada2
- **Python** (Group the collapsed family table by size-fraction metadata, compute relative abundances, and write the verification TSV)

## Examples

```
./tourmaline.sh --step taxonomy --configfile config_03_taxonomy.yaml --cores 8
```

## Evaluation signals

- Output TSV shows Sporichthyaceae and SAR11 Clade III as the top two families in the 5.0-μm fraction
- Output TSV shows Cyanobiaceae as the top family in the 0.22-μm fraction, with chloroplast sequences as its largest component
- In the 0.22-μm fraction, Microcystaceae relative abundance is lower than Cyanobiaceae
- All pass/fail verification flags in the TSV evaluate to pass against the reported expectations
- Collapsed table rank count matches collapse_taxalevel: 7 (family) and lineages trace to silva-138_1 assignments

## Limitations

- Findings depend on the unfiltered DADA2-PE run and the naive-bayes/SILVA 138.1 classifier; other filtering or classification choices (e.g. representative sequence filtering by taxonomy, length, abundance, prevalence, or ID) can change family dominance
- Cyanobiaceae dominance in the 0.22-μm fraction is chloroplast-dominated, so family-level counts can conflate Cyanobacteria with algal chloroplast 16S sequences
- Verification is only meaningful for the Lake Erie 16S rRNA test dataset; dominance patterns are dataset-specific

## Evidence

- [abstract] assigns taxonomy to the resulting representative sequences: [evidence span withheld — anchor status: not_found]
- [abstract] interactive visualizations via QIIME 2 viewer: "showing its utility for parameter optimization and the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer"
- [readme] Tourmaline workflow steps and config files: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
- [methods] SILVA 138.1 reference database: "database_name: silva-138_1"
