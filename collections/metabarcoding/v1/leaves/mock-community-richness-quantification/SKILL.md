---
name: mock-community-richness-quantification
description: Use when finished DADA2 ASV tables from depth-subsampled mock community
  sequencing runs (e.g. ERR777696 downsampled from 100 to 1,600,000 read pairs; PRJNA626434
  from 100 to 40,000 reads) are available and the goal is to evaluate how observed
  richness scales with sequencing depth.
when_to_use_negative:
- Input is still raw FASTQ data rather than finished DADA2 ASV tables — run the dadasnake
  pipeline (DADA2 denoising, chimera removal, final_table_filtering) first.
- The analysis goal is taxonomic classification of sequences rather than richness/rarefaction
  behaviour assessment.
- The dataset is not subsampled across sequencing depths, so there is no depth series
  over which rarefaction flattening can be quantified.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: dadasnake
  role: Produces the DADA2-derived ASV tables for each sequencing-depth subsample
    of the mock community datasets
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: Underlying algorithm for error estimation, denoising, and exact sequence variant
    delineation that generates the finished ASV tables analyzed here
- name: vegan
  role: R package used (rarefy/rarecurve) to compute rarefaction curves of observed
    ASV richness against read number
- name: seqtk
  role: Optional downsampling of reads per sample to create the sequencing-depth subsamples
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:2731543fc7faa3e094d0d260b015a619bad543a77a027a07caa96ce01c680588
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - dadasnake
  - seqtk
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mock-community-richness-quantification

## Summary

Quantifies observed ASV richness across sequencing-depth subsamples of mock community datasets and assesses whether rarefaction curves derived from finished DADA2 ASV tables faithfully reflect richness gain with additional sequencing depth. Surfaces the flattening artefact that makes DADA2-based richness estimates and rarefaction curves require caution.

## When to use

Use when finished DADA2 ASV tables from depth-subsampled mock community sequencing runs (e.g. ERR777696 downsampled from 100 to 1,600,000 read pairs; PRJNA626434 from 100 to 40,000 reads) are available and the goal is to evaluate how observed richness scales with sequencing depth.

## When NOT to use

- Input is still raw FASTQ data rather than finished DADA2 ASV tables — run the dadasnake pipeline (DADA2 denoising, chimera removal, final_table_filtering) first.
- The analysis goal is taxonomic classification of sequences rather than richness/rarefaction behaviour assessment.
- The dataset is not subsampled across sequencing depths, so there is no depth series over which rarefaction flattening can be quantified.

## Inputs

- DADA2-derived ASV tables (from dadasnake output formats: biom, phyloseq objects, text files, or R data sets) for each sequencing-depth subsample
- Ground-truth mock community compositions
- Subsample depth metadata (e.g. ERR777696: 100–1,600,000 read pairs; PRJNA626434: 100–40,000 reads)

## Outputs

- Rarefaction curves of observed ASV richness vs. number of reads per subsample depth (rarefaction_curves.png)
- Quantified plateau measures per curve compared against continued richness increase at greater sequencing depth

## How to apply

Load the DADA2-derived ASV tables produced by dadasnake for every sequencing-depth subsample of each mock community dataset. In R, use vegan (rarefy/rarecurve) to compute rarefaction curves of observed ASV richness against the number of reads sampled for each subsample depth, then plot curves for all depth subsamples of a dataset on the same axes. Quantify the flattening artefact by measuring the plateau of each curve and comparing it against the continued increase in true richness at greater sequencing depth, using the ground-truth mock community compositions as the richness reference. Save the figure (e.g. rarefaction_curves.png) and interpret plateaus critically: curves from finished ASV tables flatten even though greater real sequencing depth would yield more observed ASVs.

## Related tools

- **dadasnake** (Produces the DADA2-derived ASV tables for each sequencing-depth subsample of the mock community datasets) — https://github.com/a-h-b/dadasnake
- **DADA2** (Underlying algorithm for error estimation, denoising, and exact sequence variant delineation that generates the finished ASV tables analyzed here)
- **vegan** (R package used (rarefy/rarecurve) to compute rarefaction curves of observed ASV richness against read number)
- **seqtk** (Optional downsampling of reads per sample to create the sequencing-depth subsamples) — https://github.com/lh3/seqtk

## Examples

```
library(vegan); rarecurve(t(asv_table), step = 1000, xlab = 'reads sampled', ylab = 'observed ASVs')
```

## Evaluation signals

- Rarefaction curves derived from the finished ASV tables visibly flatten (plateau) with increasing read numbers.
- True richness continues to increase with sequencing depth even as the rarefaction curves plateau, matching the flattening artefact reported for the mock community use cases.
- Curves are plotted for all depth subsamples of each dataset on the same axes, covering the full subsample ranges (ERR777696: 100–1,600,000 read pairs; PRJNA626434: 100–40,000 reads).
- Output figure rarefaction_curves.png is produced and plateau measurements are quantified rather than only visually reported.

## Limitations

- Rarefaction curves based on finished DADA2 ASV tables flatten even though greater real sequencing depth would yield more observed ASVs, so richness estimates and rarefaction curves from DADA2 datasets require caution.
- Findings are demonstrated on mock community datasets (bacteria/archaea and fungi); behaviour in natural, more diverse communities may differ.
- Downsampling of reads per sample is an optional dadasnake step (seqtk); without a proper depth-subsample series the flattening artefact cannot be quantified.

## Evidence

- [other] Rarefaction curves derived from finished ASV tables flatten even though greater real sequencing depth would yield more observed ASVs, so richness estimates and rarefaction curves based on DADA2 datasets require caution.: [evidence span withheld — anchor status: not_found]
- [abstract] dadasnake wraps DADA2 preprocessing and ASV delineation: "We present dadasnake, a user-friendly, 1-command Snakemake pipeline that wraps the preprocessing of sequencing reads and the delineation of exact sequence variants"
- [supplementary] Mock community processing results and ground-truth compositions on Zenodo: "Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60]."
- [readme] Output formats for integrating ASV tables into R analysis: "Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts."
