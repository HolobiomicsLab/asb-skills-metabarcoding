---
name: microbiome-community-composition-validation
description: Use when you have a DADA2-denoised QIIME 2 feature table from a Tourmaline
  run with sample metadata (e.g. 5.0-um vs 0.22-um size fractions) and must verify
  which taxa dominate each fraction before publishing or reusing the findings.
when_to_use_negative:
- Input is not yet a feature table (e.g. raw FASTQ has not been denoised with DADA2
  in the qaqc/repseqs/taxonomy steps)
- The analysis goal is differential composition testing or beta diversity rather than
  reproducing/verifying reported dominance patterns
- The run was filtered (taxonomy, length, abundance, prevalence) or used Deblur rather
  than the unfiltered DADA2-PE run the findings were reported from
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: Tourmaline
  role: Python/Snakemake workflow that runs the QIIME 2 taxonomy step (config_03_taxonomy.yaml)
    producing the taxonomy table and .qzv visualizations consumed for verification
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Runs DADA2 denoising and taxonomy assignment (naive-bayes, SILVA 138.1) and
    provides taxa-barplot/interactive visualizations for family-level abundance inspection
- name: DADA2
  role: Denoising algorithm generating the ASV feature table being summarized at family
    level
  repo: https://github.com/benjjneb/dada2
- name: QIIME 2 viewer
  role: Web tool used to inspect taxonomy.qzv and taxa_barplot.qzv interactive visualizations
    of per-sample family relative abundances
- name: Python
  role: Language for the verification script that collapses the table at family level,
    groups by size fraction, computes relative abundances, and writes the TSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:7e18ad8031e59bfbb5f32403011cc4c6a1d20ae427cf2d3195129a2737fdf83f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - DADA2
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Tourmaline
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# microbiome-community-composition-validation

## Summary

Reproduce and verify family-level taxonomic dominance patterns in size-fractionated 16S rRNA amplicon data by collapsing Tourmaline/QIIME 2 ASV tables at family level (collapse_taxalevel: 7) and comparing relative abundances per size fraction against expected findings.

## When to use

Use when you have a DADA2-denoised QIIME 2 feature table from a Tourmaline run with sample metadata (e.g. 5.0-um vs 0.22-um size fractions) and must verify which taxa dominate each fraction before publishing or reusing the findings.

## When NOT to use

- Input is not yet a feature table (e.g. raw FASTQ has not been denoised with DADA2 in the qaqc/repseqs/taxonomy steps)
- The analysis goal is differential composition testing or beta diversity rather than reproducing/verifying reported dominance patterns
- The run was filtered (taxonomy, length, abundance, prevalence) or used Deblur rather than the unfiltered DADA2-PE run the findings were reported from

## Inputs

- Unfiltered DADA2-PE Tourmaline feature table (ASV counts, e.g. from the Lake Erie 16S rRNA test dataset)
- Representative-sequence taxonomy assignments (QIIME 2 taxonomy artifact from naive-bayes classification against SILVA 138.1)
- Sample metadata file with size-fraction annotations (5.0-um vs 0.22-um)
- Interactive visualizations taxonomy.qzv and taxa_barplot.qzv

## Outputs

- Family-level (taxalevel 7) relative-abundance table grouped by size fraction
- TSV file with verified family-level abundances and pass/fail verification flags for each expected dominance pattern

## How to apply

Load the taxonomy table and representative-sequence taxonomy assignments from the unfiltered DADA2-PE Tourmaline run produced by the taxonomy step (config_03_taxonomy.yaml, classify_method naive-bayes against SILVA 138.1). Inspect taxonomy.qzv and taxa_barplot.qzv in QIIME 2 viewer to view family-level (taxa_ranks includes family) relative abundances per sample. Collapse ASV counts at family level (collapse_taxalevel: 7), group by the size-fraction metadata column (5.0-um vs 0.22-um), and compute per-family relative abundances. Verify the expected patterns: in the 5.0-um fraction Sporichthyaceae and SAR11 Clade III should be the two most abundant families; in the 0.22-um fraction Cyanobiaceae should dominate, with chloroplast sequences as its largest component and Microcystaceae less abundant. Write the verified family-level abundances plus pass/fail flags for each expected dominance pattern to a TSV file.

## Related tools

- **Tourmaline** (Python/Snakemake workflow that runs the QIIME 2 taxonomy step (config_03_taxonomy.yaml) producing the taxonomy table and .qzv visualizations consumed for verification) — https://github.com/aomlomics/tourmaline
- **QIIME 2** (Runs DADA2 denoising and taxonomy assignment (naive-bayes, SILVA 138.1) and provides taxa-barplot/interactive visualizations for family-level abundance inspection)
- **DADA2** (Denoising algorithm generating the ASV feature table being summarized at family level) — https://github.com/benjjneb/dada2
- **QIIME 2 viewer** (Web tool used to inspect taxonomy.qzv and taxa_barplot.qzv interactive visualizations of per-sample family relative abundances)
- **Python** (Language for the verification script that collapses the table at family level, groups by size fraction, computes relative abundances, and writes the TSV)

## Examples

```
./tourmaline.sh --step taxonomy --configfile config_03_taxonomy.yaml --cores 8
```

## Evaluation signals

- Output TSV shows Sporichthyaceae and SAR11 Clade III as the two most abundant families in the 5.0-um fraction
- Output TSV shows Cyanobiaceae as the most abundant family in the 0.22-um fraction, with chloroplast sequences as its largest component
- Microcystaceae relative abundance in the 0.22-um fraction is lower than Cyanobiaceae
- Family-level collapse used taxalevel 7 and relative abundances sum to 1 within each size-fraction group
- Verification flags in the TSV match the reported expectations for every expected dominance pattern

## Limitations

- Conclusions are specific to the unfiltered DADA2-PE run; Tourmaline's representative-sequence filtering (taxonomy, length, abundance, prevalence, or ID) and alternative denoisers (Deblur) can shift dominance patterns
- Naive-Bayes classification against SILVA 138.1 assigns chloroplast 16S sequences to Cyanobiaceae, so the 0.22-um dominance partly reflects chloroplast reads rather than cyanobacterial cells
- Verification depends on the naive-bayes classifier and SILVA 138.1 reference; other classify methods or databases may produce different family assignments

## Evidence

- [abstract] it uses QIIME 2 to run either the DADA2 or Deblur denoising algorithm: "it uses QIIME 2 to run either the DADA2 or Deblur denoising algorithm"
- [abstract] assigns taxonomy to the resulting representative sequences: "assigns taxonomy to the resulting representative sequences"
- [abstract] We ran Tourmaline on a 16S ribosomal RNA amplicon data set from Lake Erie surface water: "We ran Tourmaline on a 16S ribosomal RNA amplicon data set from Lake Erie surface water"
- [abstract] easily view interactive visualizations through the HTML report, QIIME 2 viewer: "showing its utility for parameter optimization and the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer"
- [methods] SILVA 138.1 database name: "database_name: silva-138_1"
- [abstract] representative sequence filtering options: "representative sequence filtering (taxonomy, length, abundance, prevalence, or ID)"
- [readme] tourmaline.sh usage for the taxonomy step: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
