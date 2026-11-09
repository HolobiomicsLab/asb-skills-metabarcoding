---
name: its-region-taxonomic-classification
description: Use when processing paired-end ITS amplicon FASTQ data (e.g. ITS1, 267-sample
  fungal datasets) and you need region-filtered, taxonomically assigned ASV tables
  rather than 16S classification.
when_to_use_negative:
- Input is 16S rRNA amplicon data — use a 16S reference such as SILVA instead of the
  ITSx/mothur-UNITE path
- Input is already an ASV/feature table with taxonomy assigned — no amplicon re-processing
  is needed
- Reads are single-end only — the paired=true workflow and pair merging with min_overlap
  >= 20 bp assume paired-end FASTQ
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: dadasnake
  role: 1-command Snakemake pipeline orchestrating ITSx filtering and mothur classification
    of ITS1 amplicon data
  repo: https://github.com/a-h-b/dadasnake
- name: Snakemake
  role: parallelizes the workflow steps across cores/cluster (e.g. -t 4 vs -t 15)
- name: ITSx
  role: extracts/filters the ITS1 subregion from merged reads before classification
- name: mothur
  role: performs Bayesian taxonomic classification against the UNITE-formatted fungal
    ITS database
- name: DADA2
  role: quality filtering, error estimation, ASV inference, merging and chimera removal
    prior to ITS classification
- name: cutadapt
  role: primer removal (and poly-G tail removal) as the first preprocessing step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:e2535743ad2fe69e1c3b9d88123cf73fb9b15f9d14e113642e9fc7d20cdfe010
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - ITSx
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# ITS-region taxonomic classification

## Summary

Classifies fungal amplicon reads by first excising the ITS1 region with ITSx, then assigning taxonomy via mothur-based Bayesian classification against a fungal ITS (UNITE-formatted) reference database, as implemented in the dadasnake Snakemake pipeline.

## When to use

Use when processing paired-end ITS amplicon FASTQ data (e.g. ITS1, 267-sample fungal datasets) and you need region-filtered, taxonomically assigned ASV tables rather than 16S classification.

## When NOT to use

- Input is 16S rRNA amplicon data — use a 16S reference such as SILVA instead of the ITSx/mothur-UNITE path
- Input is already an ASV/feature table with taxonomy assigned — no amplicon re-processing is needed
- Reads are single-end only — the paired=true workflow and pair merging with min_overlap >= 20 bp assume paired-end FASTQ

## Inputs

- raw paired-end ITS1 FASTQ files (e.g. from NCBI SRA BioProject PRJNA272747)
- tab-separated sample table with 'library', 'r1_file', 'r2_file' headers
- dadasnake config.yaml (paired=true, ITSx region=ITS1, mothur classifier enabled)
- UNITE-formatted fungal ITS reference database (e.g. UNITE https://doi.org//10.15156/BIO/786336)

## Outputs

- taxonomically assigned ITS1 ASV/OTU tables in biom format, phyloseq objects, or text/R data sets
- workflow.done marker file in the output directory confirming completion
- runtime benchmark records (threads, wall-clock hours) e.g. runtime_benchmark.tsv

## How to apply

Write a dadasnake config.yaml with paired=true, enable ITSx with region set to ITS1, and activate mothur-based Bayesian classification against a UNITE-formatted fungal ITS database; set pair_merging min_overlap to at least 20 bp so DADA2-merged pairs survive ITSx extraction. Run the full workflow from primer removal (cutadapt) through quality filtering, error estimation and ASV inference (DADA2), read-pair merging, chimera removal, and finally ITSx filtering followed by mothur classification. Use a dryrun (dadasnake -d) first to validate the sample table with 'library', 'r1_file', 'r2_file' headers and the configuration before committing compute. On a 267-sample, 46.8-million-read dataset the expected wall-clock time is just under 4 hours on four 8 GB cores, halving to about 2x faster at 15 cores via Snakemake parallelization.

## Related tools

- **dadasnake** (1-command Snakemake pipeline orchestrating ITSx filtering and mothur classification of ITS1 amplicon data) — https://github.com/a-h-b/dadasnake
- **Snakemake** (parallelizes the workflow steps across cores/cluster (e.g. -t 4 vs -t 15))
- **ITSx** (extracts/filters the ITS1 subregion from merged reads before classification)
- **mothur** (performs Bayesian taxonomic classification against the UNITE-formatted fungal ITS database)
- **DADA2** (quality filtering, error estimation, ASV inference, merging and chimera removal prior to ITS classification)
- **cutadapt** (primer removal (and poly-G tail removal) as the first preprocessing step)

## Examples

```
dadasnake -t 4 --config paired=true useitsx=true region="ITS1" classifier=mothur
```

## Evaluation signals

- workflow.done marker file present in the output directory after each run
- ITSx filtering retains only reads assigned to the configured ITS region (ITS1)
- mothur classification produces taxonomically assigned ASV/OTU tables against the UNITE database
- recorded runtime matches the reported benchmark: just under 4 hours on four 8 GB cores with ~2x speedup at 15 cores (Fig. 2c and d)
- dryrun (dadasnake -d) passes with no sample-table or configuration errors before the full run

## Limitations

- Runtime scales with cores only up to a point — 15 cores gave only a 2-fold speedup over 4 cores (Fig. 2c and d)
- dadasnake requires Snakemake >= 5.9.1 and has not been tested with earlier versions
- dadasnake has been tested with fungalTraits v1.2; other ITS reference databases may need validation
- Default MAX_THREADS is 50 unless overridden in VARIABLE_CONFIG or at runtime

## Evidence

- [abstract] dadasnake is a user-friendly, 1-command Snakemake pipeline: "We present dadasnake, a user-friendly, 1-command Snakemake pipeline"
- [methods] ITS region processing settings: "and [UNITE](https://doi.org//10.15156/BIO/786336) for ITS"
- [full_text] ITSx and mothur in the ITS classification workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Snakemake parallelization rationale: "By use of Snakemake, dadasnake makes efficient use of high-performance computing infrastructures."
- [readme] thread cap at runtime: "MAX_THREADS - set this to the maximum number of cores you want to be using in a run."
