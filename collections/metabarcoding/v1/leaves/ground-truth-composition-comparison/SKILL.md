---
name: ground-truth-composition-comparison
description: Use when processed ASV abundance tables exist for a mock community with
  a known composition and the goal is to quantify how well relative read abundances
  reflect the expected taxon proportions.
when_to_use_negative:
- The dataset is a natural environmental community with unknown true composition —
  no ground-truth table exists to join by taxon.
- Input is raw FASTQ rather than pipeline-produced ASV/OTU abundance tables — run
  dadasnake's preprocessing and ASV delineation first.
- Input is already a comparison table of observed vs expected abundances — proceed
  directly to correlation/dispersion checks.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: dadasnake
  role: Snakemake workflow that produced the mock community processing results and
    final ASV abundance tables being validated
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: Algorithm underlying ASV delineation and taxonomic assignment whose abundance
    outputs are compared against ground truth
  repo: http://benjjneb.github.io/dada2/
- name: Snakemake
  role: Engine underlying dadasnake's reproducible processing of the mock community
    datasets
  repo: https://snakemake.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:132311b887a0c47e4e3a9624bfae7ff64ebc9e6d4a410f47bf7912dca419e5e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - Snakemake
  - dadasnake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ground-truth-composition-comparison

## Summary

Validates amplicon-sequencing processing results by comparing observed relative read abundances of ASVs in mock communities with known ground-truth taxon compositions. It was used to show that dadasnake/DADA2 read abundances misrepresent true taxon abundances in the prokaryotic (ERR777696) and fungal (PRJNA626434) mock community datasets.

## When to use

Use when processed ASV abundance tables exist for a mock community with a known composition and the goal is to quantify how well relative read abundances reflect the expected taxon proportions.

## When NOT to use

- The dataset is a natural environmental community with unknown true composition — no ground-truth table exists to join by taxon.
- Input is raw FASTQ rather than pipeline-produced ASV/OTU abundance tables — run dadasnake's preprocessing and ASV delineation first.
- Input is already a comparison table of observed vs expected abundances — proceed directly to correlation/dispersion checks.

## Inputs

- dadasnake/DADA2 final ASV abundance table for the prokaryotic mock community run ERR777696
- dadasnake/DADA2 final ASV abundance table for the fungal mock community run from BioProject PRJNA626434
- Taxonomic classification output of the DADA2-based pipeline (e.g. SILVA 16S / UNITE ITS assignments)
- Ground-truth mock community composition tables deposited at Zenodo [60]

## Outputs

- Combined per-taxon observed-vs-expected relative abundance table (mock_community_relative_abundances.csv)
- Correspondence assessment per mock community (correlation/dispersion of observed vs expected proportions)

## How to apply

Load the dadasnake/DADA2-produced final ASV abundance tables for the prokaryotic (ERR777696) and fungal (PRJNA626434) mock community runs from the deposited processing results on Zenodo [60]. Aggregate ASV read counts to assigned taxa using the pipeline's taxonomic classification output (SILVA for 16S, UNITE for ITS), then normalize per-taxon counts to relative read abundances within each mock community. Join these observed relative abundances to the ground-truth compositions also deposited at Zenodo [60] by taxon, and quantify the correspondence per taxon. Decision point: in the prokaryotic community (unequal known composition) check whether relative read abundance correlates with expected abundance; in the fungal community (equal input proportions) check whether relative abundances span several orders of magnitude instead of being uniform. Write the combined observed-vs-expected per-taxon table to mock_community_relative_abundances.csv.

## Related tools

- **dadasnake** (Snakemake workflow that produced the mock community processing results and final ASV abundance tables being validated) — https://github.com/a-h-b/dadasnake
- **DADA2** (Algorithm underlying ASV delineation and taxonomic assignment whose abundance outputs are compared against ground truth) — http://benjjneb.github.io/dada2/
- **Snakemake** (Engine underlying dadasnake's reproducible processing of the mock community datasets) — https://snakemake.readthedocs.io/en/stable/

## Evaluation signals

- Output table contains per-taxon relative read abundances for both mock communities covering the ground-truth taxa.
- Fungal mock community relative abundances span several orders of magnitude despite equal input proportions.
- Prokaryotic mock community relative abundances show no correlation with the expected unequal taxon composition.
- Per-taxa relative read abundances within each mock community sum to 1 after normalization.
- Every ground-truth taxon in the deposited composition tables is matched and joined by taxon in the observed-vs-expected table.

## Limitations

- Read abundances misrepresent true taxon abundances — the finding is that prokaryotic relative read abundance does not correlate with ground truth, so this comparison demonstrates bias rather than validating exact recovery.
- Validation is only demonstrated on the mock communities included in the study (bacteria/archaea ERR777696 and fungi PRJNA626434); other community types or primer sets may behave differently.
- Results depend on the frozen dadasnake version and database versions (SILVA v138, UNITE, fungalTraits v1.2) used for taxonomic classification.

## Evidence

- [abstract] The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi.: "The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi."
- [supplementary] Zenodo processing results and ground truth: "Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60]."
- [readme] Reporting intermediary statistics: "dadasnake reports on intermediary steps and statistics in intuitive figures and tables."
