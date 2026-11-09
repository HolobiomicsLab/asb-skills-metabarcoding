---
name: sequencing-depth-subsampling-assessment
description: Use when evaluating whether richness estimates and rarefaction curves
  from finished DADA2 ASV tables can be trusted, especially for mock community data
  subsampled across sequencing depths (e.g. 100–1,600,000 read pairs or 100–40,000
  reads).
when_to_use_negative:
- Input is raw reads or quality profiles rather than finished ASV tables — the skill
  evaluates the rarefaction artefact of already-delineated DADA2 ASV tables, not the
  denoising pipeline itself
- The analysis goal is taxonomy assignment or functional prediction (e.g. BASTA or
  PICRUSt2 tasks) rather than richness/rarefaction behaviour across sequencing depths
- Data are not subsampled across sequencing depths (or ground truth on true richness
  is unavailable), so the flattening artefact cannot be quantified against a depth
  series
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: dadasnake
  role: Snakemake pipeline that processes raw fastq-files into taxonomically assigned
    ASV/OTU tables via DADA2 for each sequencing-depth subsample
  repo: https://github.com/a-h-b/dadasnake
- name: seqtk
  role: Performs optional downsampling of reads per sample to generate the sequencing-depth
    subsamples
  repo: https://github.com/lh3/seqtk
- name: DADA2
  role: Algorithm underlying error estimation, denoising, and ASV delineation of each
    depth subsample
- name: vegan
  role: R package (rarefy/rarecurve) used to compute and plot rarefaction curves of
    observed ASV richness from the finished ASV tables
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:b7050f3c8687fd77c47204f28de607ccf273db0b1df05df9c998115571be2744
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - dadasnake
  - seqtk
  - vegan
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sequencing-depth-subsampling-assessment

## Summary

Assess whether rarefaction curves computed from finished DADA2 ASV tables (produced by dadasnake) faithfully reflect how observed ASV richness increases with true sequencing depth, by comparing curves across subsampled sequencing depths of mock community datasets.

## When to use

Use when evaluating whether richness estimates and rarefaction curves from finished DADA2 ASV tables can be trusted, especially for mock community data subsampled across sequencing depths (e.g. 100–1,600,000 read pairs or 100–40,000 reads).

## When NOT to use

- Input is raw reads or quality profiles rather than finished ASV tables — the skill evaluates the rarefaction artefact of already-delineated DADA2 ASV tables, not the denoising pipeline itself
- The analysis goal is taxonomy assignment or functional prediction (e.g. BASTA or PICRUSt2 tasks) rather than richness/rarefaction behaviour across sequencing depths
- Data are not subsampled across sequencing depths (or ground truth on true richness is unavailable), so the flattening artefact cannot be quantified against a depth series

## Inputs

- DADA2-derived ASV tables produced by dadasnake for every sequencing-depth subsample of the mock community datasets (ERR777696: 100–1,600,000 read pairs; PRJNA626434: 100–40,000 reads)
- Ground-truth mock community compositions available from Zenodo [60]

## Outputs

- Rarefaction curves of observed ASV richness vs. reads sampled for all depth subsamples, plotted on common axes (rarefaction_curves.png)
- Quantified plateau/flattenings of each rarefaction curve compared against true richness increase with sequencing depth

## How to apply

Run dadasnake on every sequencing-depth subsample of the mock community datasets — e.g. ERR777696 downsampled from 100 to 1,600,000 read pairs and PRJNA626434 from 100 to 40,000 reads — producing a finished DADA2 ASV table per depth (downsampling of reads per sample is done with seqtk). In R, load each finished ASV table and use vegan's rarefy/rarecurve to compute rarefaction curves of observed ASV richness against the number of reads sampled for each subsample depth. Plot the rarefaction curves for all depth subsamples of each dataset on the same axes, then quantify the flattening artefact by measuring the plateau of each curve and comparing it against the continued increase in true richness at greater sequencing depth. Validation: curves derived from the finished ASV tables must visibly flatten with increasing read numbers while true richness continues to increase with sequencing depth, matching the flattening artefact reported for the mock community use cases.

## Related tools

- **dadasnake** (Snakemake pipeline that processes raw fastq-files into taxonomically assigned ASV/OTU tables via DADA2 for each sequencing-depth subsample) — https://github.com/a-h-b/dadasnake
- **seqtk** (Performs optional downsampling of reads per sample to generate the sequencing-depth subsamples) — https://github.com/lh3/seqtk
- **DADA2** (Algorithm underlying error estimation, denoising, and ASV delineation of each depth subsample)
- **vegan** (R package (rarefy/rarecurve) used to compute and plot rarefaction curves of observed ASV richness from the finished ASV tables)

## Examples

```
Rscript -e 'library(vegan); asv <- read.table("final_table_ASV.txt", header=TRUE); rarecurve(asv, step=1000)'  # then plot curves for all depth subsamples and save as rarefaction_curves.png
```

## Evaluation signals

- Rarefaction curves from the finished ASV tables visibly flatten (reach a plateau) with increasing read numbers
- True richness (against ground-truth mock community compositions) continues to increase at greater sequencing depth, exposing the divergence from the rarefaction plateau
- Curves are plotted for all depth subsamples of each dataset on the same axes, covering the full depth series (100 to 1,600,000 read pairs for ERR777696; 100 to 40,000 reads for PRJNA626434)
- Figure is saved as rarefaction_curves.png and behaviour matches the flattening artefact reported for the mock community use cases

## Limitations

- Rarefaction curves and richness estimates based on finished DADA2 ASV tables can misrepresent how observed richness would grow with additional sequencing depth, so they require caution
- The artefact is demonstrated on mock community datasets; behaviour on natural communities is not established by this evidence
- Downsampling to create depth subsamples uses seqtk, which subsamples reads per sample and is optional in dadasnake; results depend on the chosen depth range

## Evidence

- [supplementary] Rarefaction curves derived from finished ASV tables flatten even though greater real sequencing depth would yield more observed ASVs, so richness estimates and rarefaction curves based on DADA2 datasets require caution.: [evidence span withheld — anchor status: not_found]
- [methods] downsampling of reads per sample - using seqtk: "downsampling of reads per sample - using seqtk"
- [intro] dadasnake is a Snakemake workflow to process amplicon sequencing data, from raw fastq-files to taxonomically assigned "OTU" tables: "dadasnake is a [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow to process amplicon sequencing data, from raw fastq-files to taxonomically assigned "OTU" tables"
- [supplementary] Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60].: "Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60]."
