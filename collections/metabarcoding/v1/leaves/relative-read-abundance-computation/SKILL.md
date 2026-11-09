---
name: relative-read-abundance-computation
description: Use when you have a DADA2-derived ASV abundance table for a mock community
  with known composition and need to quantify how observed relative read abundances
  deviate from expected taxon proportions.
when_to_use_negative:
- Input is raw sequencing reads or pre-denoising data — the skill operates on final
  ASV tables produced after DADA2 denoising, quality filtering, and chimera removal
- Samples are environmental/natural communities with no known ground-truth composition
  — there is no expected composition to join against
- You need absolute taxon abundances rather than within-sample relative read abundances
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: dadasnake
  role: produces the DADA2-based final ASV abundance and taxonomic classification
    tables that are aggregated to taxa
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: upstream algorithm that delineates exact sequence variants (ASVs) and assigns
    taxonomy feeding the abundance tables
  repo: http://benjjneb.github.io/dada2/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:95644a51b16a0db0bba2ac662b78dafb7020444d8adebde7aefa3d659fe8eeb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - dadasnake
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# relative-read-abundance-computation

## Summary

Compute per-taxon relative read abundances from dadasnake/DADA2 ASV tables of mock community runs (prokaryotic ERR777696, fungal PRJNA626434) and compare them against ground-truth community compositions to assess whether read abundances reflect true taxon abundances.

## When to use

Use when you have a DADA2-derived ASV abundance table for a mock community with known composition and need to quantify how observed relative read abundances deviate from expected taxon proportions.

## When NOT to use

- Input is raw sequencing reads or pre-denoising data — the skill operates on final ASV tables produced after DADA2 denoising, quality filtering, and chimera removal
- Samples are environmental/natural communities with no known ground-truth composition — there is no expected composition to join against
- You need absolute taxon abundances rather than within-sample relative read abundances

## Inputs

- dadasnake/DADA2 final ASV abundance tables for the prokaryotic mock community run ERR777696
- dadasnake/DADA2 final ASV abundance tables for the fungal mock community run (BioProject PRJNA626434)
- DADA2 taxonomic classification output from the dadasnake pipeline
- Ground-truth mock community compositions deposited at Zenodo [60]

## Outputs

- mock_community_relative_abundances.csv — per-taxon relative read abundances joined to expected compositions for both mock communities

## How to apply

Load the dadasnake/DADA2-produced final ASV abundance tables for the prokaryotic (ERR777696) and fungal (PRJNA626434) mock community runs from the deposited processing results (Zenodo [60]). Aggregate ASV read counts to assigned taxa using the DADA2-based taxonomic classification output, then normalize per-taxon counts to relative read abundances within each mock community. Load the ground-truth mock community compositions from Zenodo [60] and join them to observed abundances by taxon. Quantify observed-vs-expected correspondence: for the fungal community (equal input proportions) check that relative abundances span several orders of magnitude; for the prokaryotic community (unequal known composition) check that observed relative abundances show no correlation with true taxon abundances. Write the combined observed-vs-expected per-taxon table to mock_community_relative_abundances.csv.

## Related tools

- **dadasnake** (produces the DADA2-based final ASV abundance and taxonomic classification tables that are aggregated to taxa) — https://github.com/a-h-b/dadasnake
- **DADA2** (upstream algorithm that delineates exact sequence variants (ASVs) and assigns taxonomy feeding the abundance tables) — http://benjjneb.github.io/dada2/

## Examples

```
Rscript -e 'obs <- read.csv("asv_table.csv"); agg <- aggregate(count ~ taxon, obs, sum); agg$rel <- agg$count/sum(agg$count); exp <- read.csv("ground_truth.csv"); out <- merge(agg, exp, by="taxon"); write.csv(out, "mock_community_relative_abundances.csv", row.names=FALSE)'
```

## Evaluation signals

- The output mock_community_relative_abundances.csv contains per-taxon relative read abundances for both mock communities covering all ground-truth taxa
- Fungal mock community relative abundances (equal input proportions) span several orders of magnitude
- Prokaryotic mock community relative read abundances show no correlation with the expected unequal composition
- Per-taxon relative abundances sum to 1 within each mock community

## Limitations

- Read abundances misrepresent true taxon abundances, so relative read abundance should not be interpreted as true relative biomass for these mock communities
- Conclusions are specific to the dadasnake/DADA2 default processing demonstrated in the paper; results depend on the pipeline's taxonomic classification and filtering settings

## Evidence

- [abstract] The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi.: "The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi."
- [supplementary] Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60].: "Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60]."
- [readme] Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts.: "Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts."
