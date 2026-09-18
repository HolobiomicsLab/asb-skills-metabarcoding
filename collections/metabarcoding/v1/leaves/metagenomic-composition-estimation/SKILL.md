---
name: metagenomic-composition-estimation
description: Use when long-read (PacBio/Oxford Nanopore) metagenomic FASTQ reads must
  be assigned per-read to taxa and converted into strain- or species-level sample
  composition estimates against a reference database such as miniSeq+H/RefSeq.
when_to_use_negative:
- The sample has a high proportion of short reads — reads below the chosen minimum
  length end up with minimizer sets too small to reliably determine mapping locations
  and remain unassigned
- The sample comprises many non-characterized (novel) entities absent from the reference
  database — database-sample incongruity detection is limited and remains an open
  problem
- Real-time/streaming analysis is required (e.g. clinical applications with live sequencing
  control) — MetaMaps does not support streaming data
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: 'Primary tool: approximate mapping, probabilistic mapping scoring, and EM-based
    sample composition estimation for long reads'
  repo: github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying minimizer-based approximate mapping algorithm used by MetaMaps
    to generate candidate mapping locations
  repo: https://github.com/marbl/MashMap
- name: Centrifuge
  role: 'Comparison baseline: per-read classifications are aggregated into taxon-level
    composition profiles for r2/L1 benchmarking'
- name: Bracken
  role: 'Comparison baseline: per-read classifications are aggregated into taxon-level
    composition profiles for r2/L1 benchmarking'
- name: MEGAN
  role: 'Comparison baseline: LAST+MEGAN-LR per-read classifications are aggregated
    into composition profiles for benchmarking'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:8d31d3d27fc6e10481797b5c731e43fad743e8363d45d5a18d68fd7f7f41f3f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Bracken
  - Centrifuge
  - LAST
  - MEGAN
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# metagenomic-composition-estimation

## Summary

Estimate taxonomic sample composition from long-read (PacBio/ONT) metagenomic data by combining minimizer-based approximate mapping with probabilistic mapping-quality scoring and EM-based estimation of overall sample composition, as implemented in MetaMaps.

## When to use

Use when long-read (PacBio/Oxford Nanopore) metagenomic FASTQ reads must be assigned per-read to taxa and converted into strain- or species-level sample composition estimates against a reference database such as miniSeq+H/RefSeq.

## When NOT to use

- The sample has a high proportion of short reads — reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- The sample comprises many non-characterized (novel) entities absent from the reference database — database-sample incongruity detection is limited and remains an open problem
- Real-time/streaming analysis is required (e.g. clinical applications with live sequencing control) — MetaMaps does not support streaming data

## Inputs

- Long-read metagenomic reads in FASTQ format (PacBio or Oxford Nanopore, e.g. Zymo GridION/PromethION mock community data)
- MetaMaps classification database (e.g. miniSeq+H: >12,000 microbial genomes plus the human reference genome, ~8G compressed, with FASTA and taxonomy sub-directory)
- Ground-truth sample compositions for benchmarking (strain-level for simulated i100/p25; species-level for HMP7, Zymo, CAMI)

## Outputs

- classification_results.EM.WIMP: sample composition per taxonomic level with Absolute, EMFrequency, and PotFrequency columns
- classification_results.EM.reads2Taxon: per-read taxon assignment (read ID to taxon ID)
- classification_results.EM.reads2Taxon.krona: per-read assignments in Krona format with quality values
- classification_results.EM.contigCoverage: 1000 bp window read coverage per contig
- classification_results.EM: complete set of approximate mappings with estimated alignment identity and mapping quality (mapping qualities of all mappings per read sum to 1)
- composition_metrics.csv and composition_profiles.csv (reproduction workflow: Pearson r2 and L1-distance per tool-dataset combination, and per-tool estimated compositions)

## How to apply

Run MetaMaps in two steps: first './metamaps mapDirectly' to generate approximate mapping locations for each long read (based on MashMap minimizer mapping, with optional --maxmemory in GB and -t multithreading), then './metamaps classify' to score each mapping location probabilistically and estimate total sample composition with the EM algorithm, which borrows information across reads to distinguish closely related genomes. Read the '.EM.WIMP' output where the 'definedGenomes' level gives strain-level resolution (the level at which the EM is carried out); use 'EMFrequency' (pre-unmapped-read correction) and 'PotFrequency' (final frequency after correcting for unmapped reads) as composition estimates, and '.EM.reads2Taxon' for per-read assignments (taxon IDs starting with 'x' resolve via the database's extended taxonomy). Validate accuracy by computing Pearson r2 and L1 distance between estimated and true compositions at strain level (e.g., i100, p25) or species level (e.g., HMP7, Zymo, CAMI), expecting strain-level r2 of ~0.88 (i100) and ~0.78 (p25), rising to >0.99 at higher taxonomic levels and species-level r2 of 0.98 (HMP7) and 0.97 (Zymo).

## Related tools

- **MetaMaps** (Primary tool: approximate mapping, probabilistic mapping scoring, and EM-based sample composition estimation for long reads) — github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying minimizer-based approximate mapping algorithm used by MetaMaps to generate candidate mapping locations) — https://github.com/marbl/MashMap
- **Centrifuge** (Comparison baseline: per-read classifications are aggregated into taxon-level composition profiles for r2/L1 benchmarking)
- **Bracken** (Comparison baseline: per-read classifications are aggregated into taxon-level composition profiles for r2/L1 benchmarking)
- **MEGAN** (Comparison baseline: LAST+MEGAN-LR per-read classifications are aggregated into composition profiles for benchmarking)

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Strain-level Pearson r2 between estimated and true abundances reaches 0.88 on i100 and 0.78 on p25, increasing to >0.99 at higher taxonomic levels
- Species-level r2 of 0.98 on HMP7 and 0.97 on Zymo; species-level read assignment accuracy >94% when sample genomes or close relatives are in the database
- In classification_results.EM.WIMP, EMFrequency values precede unmapped-read correction while PotFrequency values are the final frequencies after correcting for unmapped reads — check the correction was applied
- Mapping qualities of all mappings for one read sum to 1 in classification_results.EM
- Reproduced metrics match Fig. 2, Fig. 5, and Supplementary Data 3 of the original paper

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Analysis of samples with a high proportion of short reads is not well addressed
- Complex samples comprising many non-characterized entities remain challenging; database-sample mismatches (e.g. in HMP7 and CAMI) require visual analysis or automatic thresholding to detect
- No explicit term for genomic divergence in the statistical models limits explicit detection of novel strains and species; detection of novelty from long reads remains an open question
- No support for streaming data, which would be important for clinical applications and real-time sequencing control
- The --maxmemory memory target is heuristic; actual memory usage fluctuates and can exceed the target

## Evidence

- [abstract] MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
- [intro] each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm: "each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm"
- [discussion] Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations.: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
- [readme] The level "definedGenomes" represents strain-level resolution (i.e., the defined genomes in the classification database). The EM algorithm is carried out at this level.: [evidence span withheld — anchor status: not_found]
- [readme] PotFrequency specifies the estimated final frequency of the taxonomic entity (i.e. after correcting for unmapped reads).: [evidence span withheld — anchor status: not_found]
