---
name: silva-reference-database-configuration
description: Use when running dadasnake (or DADA2/mothur-based classification) on
  16S/ITS amplicon FASTQ data and the reference database paths for taxonomic assignment
  must be set in the config before classification.
when_to_use_negative:
- Input data are fungal ITS amplicons that require UNITE rather than the SILVA SSU
  reference
- Classification is to be run against a custom or non-16S database (e.g. NCBI 16S
  ribosomal RNA at ../DBs/ncbi_16S_ribosomal_RNA) outside dadasnake's tested SILVA
  defaults
edam_operation: http://edamontology.org/operation_0392
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3168
tools:
- name: dadasnake
  role: Workflow whose configuration file and config.default.yaml hold the reference
    database paths used for taxonomic classification
  repo: https://github.com/a-h-b/dadasnake
- name: mothur
  role: Bayesian classifier (cutoff 60) that consumes the SILVA SSU reference to assign
    taxonomy to ASV sequences
- name: BLAST
  role: Used alongside mothur per the default configuration to support classification
    of ASVs against the configured reference database
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:fb3ccebb2fef0d92bdf042f8d7e7443d9bb82f5c3dbfe3f8ec7899f135f298e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - BLAST
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: http://edamontology.org/operation_0392
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# silva-reference-database-configuration

## Summary

Configures dadasnake's taxonomic classification with the correct reference database — SILVA v.138 (e.g. silva_nr99_v138_train_set.fa.gz, SILVA_SSU_r138_2019.RData) for 16S bacterial/archaeal amplicons or UNITE for ITS — by setting database paths in config.default.yaml and the sample/config files.

## When to use

Use when running dadasnake (or DADA2/mothur-based classification) on 16S/ITS amplicon FASTQ data and the reference database paths for taxonomic assignment must be set in the config before classification.

## When NOT to use

- Input data are fungal ITS amplicons that require UNITE rather than the SILVA SSU reference
- Classification is to be run against a custom or non-16S database (e.g. NCBI 16S ribosomal RNA at ../DBs/ncbi_16S_ribosomal_RNA) outside dadasnake's tested SILVA defaults

## Inputs

- 16S rRNA gene amplicon sequencing FASTQ files (e.g. ENA ERR777696)
- Downloaded SILVA v.138 reference files (silva_nr99_v138_train_set.fa.gz, SILVA_SSU_r138_2019.RData, silva_species_assignment_v138.fa.gz)
- dadasnake config.default.yaml and human-readable configuration file (yaml)

## Outputs

- Configuration with valid reference-database paths consumed by the classification step
- Taxonomically assigned ASV/OTU tables (mothur Bayesian classifications at cutoff 60, with BLAST/BASTA summaries)

## How to apply

Before running dadasnake, edit config.default.yaml once to point the classification steps at the downloaded reference databases: use the SILVA database (current exports from arb-silva.de, e.g. silva_nr99_v138_train_set.fa.gz and SILVA_SSU_r138_2019.RData for DADA2, SILVA_138 SSU for the mothur Bayesian classifier with cutoff 60) for 16S data, and UNITE for ITS data. For mock-community benchmarking like ERR777696, the config additionally names silva_species_assignment_v138.fa.gz for species assignment and enables BLAST/BASTA summarization of mothur results per the default configuration. Database paths are set alongside file-name keys such as '../DBs/ncbi_16S_ribosomal_RNA' in the configuration file, and the correctness of these default configurations was demonstrated on bacterial, archaeal, and fungal mock community data. Choosing the SILVA v.138 SSU reference is what allows ASVs to be classified to genus level for comparison against the 59-species ground-truth mock composition.

## Related tools

- **dadasnake** (Workflow whose configuration file and config.default.yaml hold the reference database paths used for taxonomic classification) — https://github.com/a-h-b/dadasnake
- **mothur** (Bayesian classifier (cutoff 60) that consumes the SILVA SSU reference to assign taxonomy to ASV sequences)
- **BLAST** (Used alongside mothur per the default configuration to support classification of ASVs against the configured reference database)

## Evaluation signals

- config.default.yaml contains valid paths to the downloaded SILVA v.138 files (e.g. silva_nr99_v138_train_set.fa.gz, SILVA_SSU_r138_2019.RData) and the run completes the classification step without missing-database errors
- mothur classification runs with cutoff 60 and produces taxonomy strings for all inferred ASVs (59 prokaryotic ASVs in the ERR777696 mock community benchmark)
- Genus-level assignments can be compared against the ground-truth mock community composition, with only the four expected contaminants (Corynebacterium, Staphylococcus, Rhizobiaceae, Sphingomonas) flagged as mismatches
- BLAST/BASTA summaries are generated alongside the mothur classifications when the default configuration is left enabled

## Limitations

- dadasnake has been tested with specific database versions (SILVA v.138 for 16S, UNITE and fungalTraits v1.2 for ITS); untested database versions or formats are not guaranteed to work
- Database paths must be set once in config.default.yaml and the file is advised to be made non-writable for multi-user setups; misconfigured paths silently break classification
- Classification accuracy is reference-dependent — mock-community recovery of 59 ASVs relies on SILVA v.138 covering all 59 species

## Evidence

- [methods] SILVA database for 16S data: "I'd suggest to use the [SILVA database](https://www.arb-silva.de/no_cache/download/archive/current/Exports/) for 16S data"
- [readme] Database paths set in config.default.yaml: "The same goes for config.default.yaml once you've set the paths to the databases you want to use (see below)."
- [abstract] Default configurations validated on mock communities: "The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi."
- [methods] SILVA v138 training set file configured for DADA2 classification: ""silva_nr99_v138_train_set.fa.gz""
