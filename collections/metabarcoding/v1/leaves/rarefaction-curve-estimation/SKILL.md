---
name: rarefaction-curve-estimation
description: Use when DADA2-derived ASV tables exist for a series of sequencing-depth
  subsamples (e.g. ERR777696 downsampled from 100 to 1,600,000 read pairs; PRJNA626434
  from 100 to 40,000 reads) and you must assess whether rarefaction-based richness
  estimates are trustworthy.
when_to_use_negative:
- Input is raw FASTQ reads with no finished ASV table — dadasnake/DADA2 processing
  must run first (including quality filtering, chimera removal, and final_table_filtering)
  before rarefaction curves can be derived
- The dataset has only a single sequencing depth, so the flattening artefact (curve
  plateau vs. continued true richness increase) cannot be quantified across subsamples
- The goal is taxonomic classification or OTU clustering rather than assessing whether
  rarefaction-based richness estimates reflect how observed richness would increase
  with additional sequencing depth
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: dadasnake
  role: Snakemake pipeline that produced the finished DADA2 ASV tables for each sequencing-depth
    subsample; supports optional downsampling of reads per sample using seqtk
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: Algorithm underlying dadasnake that delineates the exact sequence variants
    (ASVs) whose observed richness is rarefied
  repo: http://benjjneb.github.io/dada2/
- name: vegan
  role: R package providing rarefy/rarecurve to compute rarefaction curves of observed
    ASV richness against number of reads sampled
- name: seqtk
  role: Tool used within dadasnake for optional downsampling of reads per sample to
    generate the sequencing-depth subsamples
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:d0f13ee56d74feb6b80ff8339a0fe6e019c15c807c76705962b3ff94c0e37087
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - dadasnake
  - seqtk
  - vegan
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# rarefaction-curve-estimation

## Summary

Computes and compares rarefaction curves of observed ASV richness across sequencing-depth subsamples of mock community datasets to expose the flattening artefact of rarefaction curves derived from finished DADA2 ASV tables.

## When to use

Use when DADA2-derived ASV tables exist for a series of sequencing-depth subsamples (e.g. ERR777696 downsampled from 100 to 1,600,000 read pairs; PRJNA626434 from 100 to 40,000 reads) and you must assess whether rarefaction-based richness estimates are trustworthy.

## When NOT to use

- Input is raw FASTQ reads with no finished ASV table — dadasnake/DADA2 processing must run first (including quality filtering, chimera removal, and final_table_filtering) before rarefaction curves can be derived
- The dataset has only a single sequencing depth, so the flattening artefact (curve plateau vs. continued true richness increase) cannot be quantified across subsamples
- The goal is taxonomic classification or OTU clustering rather than assessing whether rarefaction-based richness estimates reflect how observed richness would increase with additional sequencing depth

## Inputs

- DADA2-derived ASV tables from dadasnake for each sequencing-depth subsample (e.g. biom format, phyloseq objects, or flexible text files / R data sets)
- Read-depth metadata for each subsample (e.g. ERR777696 subsampled 100–1,600,000 read pairs; PRJNA626434 subsampled 100–40,000 reads)
- Ground-truth mock community compositions (available on Zenodo [60]) for comparison against true richness

## Outputs

- rarefaction_curves.png: rarefaction curves of observed ASV richness vs. reads sampled for all depth subsamples of each dataset
- Quantified plateau measurements of each rarefaction curve compared against the continued increase in true richness at greater sequencing depth

## How to apply

Load the DADA2-derived ASV tables produced by dadasnake for every sequencing-depth subsample of the mock community dataset(s). In R, use vegan's rarefy/rarecurve to compute rarefaction curves of observed ASV richness against the number of reads sampled for each subsample depth, then plot the curves for all depth subsamples of each dataset on the same axes. Quantify the flattening artefact by measuring the plateau of each curve and comparing it against the continued increase in true richness at greater real sequencing depth. Rationale: dadasnake offers optional downsampling of reads per sample using seqtk, so depth subsampling and ASV delineation are reproducible, and the final tables are delivered in flexible text files or R data sets that feed directly into vegan.

## Related tools

- **dadasnake** (Snakemake pipeline that produced the finished DADA2 ASV tables for each sequencing-depth subsample; supports optional downsampling of reads per sample using seqtk) — https://github.com/a-h-b/dadasnake
- **DADA2** (Algorithm underlying dadasnake that delineates the exact sequence variants (ASVs) whose observed richness is rarefied) — http://benjjneb.github.io/dada2/
- **vegan** (R package providing rarefy/rarecurve to compute rarefaction curves of observed ASV richness against number of reads sampled)
- **seqtk** (Tool used within dadasnake for optional downsampling of reads per sample to generate the sequencing-depth subsamples) — https://github.com/lh3/seqtk

## Examples

```
Rscript -e 'library(vegan); asv <- t(read.delim("asv_table.txt", row.names=1)); rarecurve(asv, step=1000, sample=NA); png("rarefaction_curves.png")'
```

## Evaluation signals

- Rarefaction curves derived from the finished ASV tables must visibly flatten with increasing read numbers while true richness continues to increase with sequencing depth, matching the flattening artefact reported for the mock community use cases
- One rarefaction curve is plotted per sequencing-depth subsample of each dataset (ERR777696: 100–1,600,000 read pairs; PRJNA626434: 100–40,000 reads) on the same axes
- Curve x-axis is the number of reads sampled and y-axis is observed ASV richness from the finished DADA2 table, not OTU-clustered richness
- The plateau of each curve is quantified numerically, not just inspected visually
- The output figure rarefaction_curves.png is produced for each mock community dataset

## Limitations

- Rarefaction curves derived from finished DADA2 ASV tables flatten even though greater real sequencing depth would yield more observed ASVs, so richness estimates and rarefaction curves based on DADA2 datasets require caution
- The artefact was demonstrated on mock community datasets (ERR777696 and PRJNA626434) with known ground-truth compositions; extrapolation to natural communities is not directly validated
- The curve-flattening behaviour is a property of the finished ASV tables, so any prior dadasnake filtering (final_table_filtering) will shape the rarefaction result

## Evidence

- [methods] optional downsampling of reads per sample using seqtk: "optional downsampling of reads per sample - using seqtk"
- [supplementary] mock community processing results and visualization scripts available on Zenodo: "Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60]."
- [readme] dadasnake output formats feed R analysis scripts: "Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts."
- [intro] DADA2 pipeline output used for rarefaction: "dadasnake is a [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow to process amplicon sequencing data, from raw fastq-files to taxonomically assigned "OTU" tables"
