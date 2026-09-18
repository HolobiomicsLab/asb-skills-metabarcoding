---
name: amplicon-taxonomic-abundance-verification
description: Use when a Tourmaline taxonomy step output (taxonomy.qzv, taxa_barplot.qzv,
  collapsed feature table) must be checked against reported dominant-taxa findings
  per sample group, e.g., size-fraction or treatment comparisons.
when_to_use_negative:
- The input is raw FASTQ data that has not yet passed the Tourmaline qaqc and repseqs
  steps — taxonomy cannot be verified before denoising and taxonomy assignment.
- The run used a filtered representative-sequence table (taxonomy/length/abundance/prevalence
  filtering changed), since the expected abundances were established on the unfiltered
  DADA2-PE run and filtered runs may not reproduce the same dominance patterns.
- 'The taxonomic rank of interest is not family — collapse_taxalevel: 7 and the taxa_ranks
  inspection are family-specific; other ranks require a different collapse level.'
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0632
tools:
- name: Tourmaline
  role: Runs the taxonomy step (config_03_taxonomy.yaml) via tourmaline.sh/Snakemake,
    producing taxonomy.qzv and taxa_barplot.qzv
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2
  role: 'Executes naive-Bayes taxonomic classification against SILVA 138.1 and taxonomy
    collapse (collapse_taxalevel: 7)'
- name: DADA2
  role: Produces the representative sequences and ASV table (DADA2-PE run) that taxonomy
    is assigned to and verified from
  repo: github.com/benjjneb/dada2
- name: QIIME 2 viewer
  role: Interactive inspection of taxonomy.qzv and taxa_barplot.qzv for family-level
    relative abundances per sample
- name: Python
  role: Groups family-collapsed ASV counts by size-fraction metadata, computes relative
    abundances, and writes the verification TSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:bb147e0076b4c9c1baf586d703010079f3cb2e2364b5f6200fe986cd6cb752ab
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# amplicon-taxonomic-abundance-verification

## Summary

Verifies expected family-level taxonomic dominance patterns in Tourmaline/QIIME 2 amplicon runs by collapsing ASV counts to a target rank (e.g., collapse_taxalevel: 7), grouping by sample metadata such as size fraction, computing relative abundances, and comparing against reported expectations (e.g., Sporichthyaceae and SAR11 Clade III dominating a 5.0-μm fraction).

## When to use

Use when a Tourmaline taxonomy step output (taxonomy.qzv, taxa_barplot.qzv, collapsed feature table) must be checked against reported dominant-taxa findings per sample group, e.g., size-fraction or treatment comparisons.

## When NOT to use

- The input is raw FASTQ data that has not yet passed the Tourmaline qaqc and repseqs steps — taxonomy cannot be verified before denoising and taxonomy assignment.
- The run used a filtered representative-sequence table (taxonomy/length/abundance/prevalence filtering changed), since the expected abundances were established on the unfiltered DADA2-PE run and filtered runs may not reproduce the same dominance patterns.
- The taxonomic rank of interest is not family — collapse_taxalevel: 7 and the taxa_ranks inspection are family-specific; other ranks require a different collapse level.

## Inputs

- Tourmaline taxonomy-table artifact and representative-sequence taxonomy assignments from the unfiltered DADA2-PE run
- taxonomy.qzv and taxa_barplot.qzv interactive visualizations from the Tourmaline HTML report
- Sample metadata file containing the grouping column (e.g., size fraction: 5.0-um vs 0.22-um)
- config_03_taxonomy.yaml (classify_method: naive-bayes, database: SILVA 138.1, collapse_taxalevel: 7)

## Outputs

- Family-level relative abundance table per sample group (e.g., per size fraction)
- TSV file with verified family-level abundances and pass/fail verification flags for each expected dominance pattern

## How to apply

Run the Tourmaline taxonomy step (config_03_taxonomy.yaml) with classify_method naive-bayes against SILVA 138.1, then inspect taxonomy.qzv and taxa_barplot.qzv in the QIIME 2 viewer to confirm family-level (taxa_ranks includes family) relative abundances per sample. Collapse ASV counts at family level using collapse_taxalevel: 7, then group the collapsed table by the sample-metadata field distinguishing the fractions of interest (e.g., 5.0-μm vs 0.22-μm) and compute per-family relative abundances with Python. Check each expected dominance pattern: e.g., confirm Sporichthyaceae and SAR11 Clade III are the two most abundant families in the 5.0-μm fraction, and that Cyanobiaceae is most abundant in the 0.22-μm fraction, chloroplast sequences are its largest component, and Microcystaceae is less abundant. Write the verified family-level abundances plus pass/fail flags for each expected pattern to a TSV file.

## Related tools

- **Tourmaline** (Runs the taxonomy step (config_03_taxonomy.yaml) via tourmaline.sh/Snakemake, producing taxonomy.qzv and taxa_barplot.qzv) — github.com/aomlomics/tourmaline
- **QIIME 2** (Executes naive-Bayes taxonomic classification against SILVA 138.1 and taxonomy collapse (collapse_taxalevel: 7))
- **DADA2** (Produces the representative sequences and ASV table (DADA2-PE run) that taxonomy is assigned to and verified from) — github.com/benjjneb/dada2
- **QIIME 2 viewer** (Interactive inspection of taxonomy.qzv and taxa_barplot.qzv for family-level relative abundances per sample)
- **Python** (Groups family-collapsed ASV counts by size-fraction metadata, computes relative abundances, and writes the verification TSV)

## Examples

```
./tourmaline.sh --step taxonomy --configfile config_03_taxonomy.yaml --cores 8
```

## Evaluation signals

- Output TSV shows Sporichthyaceae and SAR11 Clade III as the top two families in the 5.0-μm fraction.
- Output TSV shows Cyanobiaceae as the top family in the 0.22-μm fraction, with chloroplast sequences as its largest component.
- Within the 0.22-μm fraction, Microcystaceae relative abundance is lower than Cyanobiaceae.
- Pass/fail verification flags in the TSV match all reported expectations for both size fractions.
- Relative abundances per group sum to 1 (or 100%) within each size-fraction group.

## Limitations

- Verification is against expectations from the specific unfiltered DADA2-PE test run of the Western Lake Erie 16S rRNA dataset; other denoising methods (e.g., Deblur single-end), filtering settings, or reference databases may shift family dominance.
- Naive-Bayes classification against SILVA 138.1 assigns chloroplast sequences to Cyanobiaceae, so family-level dominance in the 0.22-μm fraction is confounded by chloroplast origin and must be inspected at finer rank.
- Only family rank (collapse_taxalevel: 7) is covered; genus- or species-level verification needs additional resolution that the naive-Bayes classifier may not deliver confidently.

## Evidence

- [other] The most abundant bacterial families in the 5.0-μm fraction are Sporichthyaceae and SAR11 Clade III, while the most abundant bacterial family in the 0.22-μm fraction is Cyanobiaceae.: [evidence span withheld — anchor status: not_found]
- [abstract] Automatic determination of trimming parameters and DADA2 denoising: "automatic determination of trimming parameters using quality scores"
- [abstract] Interactive visualizations viewable via QIIME 2 viewer: "the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer"
- [readme] tourmaline.sh step-based execution: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
- [abstract] Taxonomy assignment step in the workflow: "assigns taxonomy to the resulting representative sequences"
- [methods] Databases used for classification: "database_name: silva-138_1"
