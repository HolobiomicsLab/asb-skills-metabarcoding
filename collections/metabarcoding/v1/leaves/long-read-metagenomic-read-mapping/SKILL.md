---
name: long-read-metagenomic-read-mapping
description: Use when you need strain-/species-level assignment and sample composition
  from long-read (PacBio/ONT) metagenomic FASTQ data, especially with reads ≥1000
  bp and a comprehensive reference database.
when_to_use_negative:
- 'Samples dominated by short reads: reads below the minimum length (1000 bp) end
  up with minimizer sets too small to reliably determine mapping locations and remain
  unassigned'
- Complex samples comprising many non-characterized entities not represented in the
  reference database
- Applications requiring real-time/streaming analysis — MetaMaps does not support
  streaming data
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: 'Primary tool: performs minimizer-based approximate mapping (mapDirectly),
    probabilistic mapping quality scoring, and EM-based sample composition estimation
    (classify)'
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying approximate-mapping algorithm whose winnowed-minhash (minimizer
    + MinHash) strategy generates candidate mapping locations and identity estimates
  repo: https://github.com/marbl/MashMap
- name: Kraken
  role: k-mer-based baseline classifier used for comparison of precision/recall on
    the same read sets
- name: Kraken 2
  role: k-mer-based baseline classifier used for comparison of precision/recall on
    the same read sets
- name: MEGAN
  role: Alignment-based baseline (LAST+MEGAN-LR) used for comparison of read-assignment
    accuracy
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_004
  - task_006
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:060b0fd4f517d6798a7c7684e8723c289d9f3b56a761950b094ef313f3af4cc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Kraken
  - Kraken 2
  - MEGAN
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# long-read-metagenomic-read-mapping

## Summary

A two-stage skill for assigning long PacBio/Oxford Nanopore metagenomic reads: minimizer-based approximate mapping (MashMap-derived) against a comprehensive RefSeq database, followed by probabilistic scoring of mapping locations and EM-based sample composition estimation, yielding per-read locations, alignment identities, and mapping qualities.

## When to use

Use when you need strain-/species-level assignment and sample composition from long-read (PacBio/ONT) metagenomic FASTQ data, especially with reads ≥1000 bp and a comprehensive reference database.

## When NOT to use

- Samples dominated by short reads: reads below the minimum length (1000 bp) end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Complex samples comprising many non-characterized entities not represented in the reference database
- Applications requiring real-time/streaming analysis — MetaMaps does not support streaming data

## Inputs

- Long-read (PacBio/ONT) metagenomic FASTQ file, reads ideally ≥1000 bp (e.g. HMP7, CAMI, i100, p25, Zymo datasets)
- MetaMaps classification database (e.g. miniSeq+H: >12,000 RefSeq microbial genomes + human reference, with FASTA and taxonomy directories)

## Outputs

- classification_results.EM: complete set of approximate mappings (read ID, coordinates, strand, contig, estimated identity via Poisson and binomial models, MinHash intersection/union sizes, mapping quality summing to 1 per read)
- classification_results.EM.WIMP: sample composition at multiple taxonomic levels (Absolute, EMFrequency, PotFrequency columns)
- classification_results.EM.reads2Taxon: per-read taxon assignment (strain-level at 'definedGenomes' level)
- classification_results.EM.reads2Taxon.krona: per-read assignments in Krona format
- classification_results.EM.contigCoverage: 1000-bp-window read coverage per contig
- classification_results.EM.lengthAndIdentitiesPerMappingUnit: read length and estimated identity per read

## How to apply

Build MetaMaps from source (./bootstrap.sh; ./configure --with-boost=/path/to/boost; make metamaps) and download a database such as miniSeq+H (>12,000 microbial genomes plus the human reference). Run mapping with 'metamaps mapDirectly' to generate approximate mapping locations using a minimizer-based strategy tuned to a minimum read length of 1000 bp, optionally with '--maxmemory' (e.g. 20 on a 32G machine) to cap peak RAM and '-t' for multithreading. Then run 'metamaps classify' to score each candidate location with the binomial mapping quality model and estimate sample composition via the EM algorithm. Candidate locations with estimated identity of at least 80% are retained; per-location posterior mapping qualities for each read sum to 1. Rationale: approximate mapping makes MetaMaps much faster than alignment-based methods while remaining robust against large contaminant genomes that cause false positives in k-mer-based classifiers.

## Related tools

- **MetaMaps** (Primary tool: performs minimizer-based approximate mapping (mapDirectly), probabilistic mapping quality scoring, and EM-based sample composition estimation (classify)) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying approximate-mapping algorithm whose winnowed-minhash (minimizer + MinHash) strategy generates candidate mapping locations and identity estimates) — https://github.com/marbl/MashMap
- **Kraken** (k-mer-based baseline classifier used for comparison of precision/recall on the same read sets)
- **Kraken 2** (k-mer-based baseline classifier used for comparison of precision/recall on the same read sets)
- **MEGAN** (Alignment-based baseline (LAST+MEGAN-LR) used for comparison of read-assignment accuracy)

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results --maxmemory 20; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- For every read, the mapping qualities of all mappings in classification_results.EM sum up to 1
- Every retained candidate mapping location has an estimated alignment identity of at least 80%
- Species-level precision/recall ≥99% and strain-level metrics ~94% (i100) / 89% (p25) when compared to ground truth, outperforming Centrifuge by >30% at strain level
- Sample composition estimation yields r2 > 0.97 against known composition on simulated and real data
- Peak memory ≤16 GB (or respects the '--maxmemory' target, accepting ~1.2–1.4x runtime increase) when mapping against the >12,000-genome database

## Limitations

- Reads that fall below the chosen minimum length end up with minimizer sets too small to reliably determine their mapping locations
- Analysis of samples with a high proportion of short reads is not well addressed by MetaMaps
- Complex samples comprising many non-characterized entities remain challenging
- No support for streaming data, important for clinical applications and real-time sequencing control
- No explicit term for genomic divergence integrated into the statistical models, limiting explicit detection of novel strains and species
- The '--maxmemory' memory target is implemented heuristically; actual memory usage will fluctuate and exceed the target

## Evidence

- [intro] two-stage procedure: "Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [intro] minimizer-based approximate mapping: "First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29."
- [abstract] accuracy claim: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
- [readme] mappings output format: [evidence span withheld — anchor status: not_found]
- [readme] memory-efficient mapping: [evidence span withheld — anchor status: not_found]
- [discussion] short-read limitation: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
