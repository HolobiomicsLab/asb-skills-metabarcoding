---
name: eukaryotic-read-unclassification-precision-recall-evaluation
description: Use when testing a metagenomic classifier's robustness to large out-of-database
  contaminant genomes, by quantifying precision/recall of leaving reads unclassified
  rather than misassigning them.
when_to_use_negative:
- The sample is expected to contain only organisms whose genomes or close relatives
  are in the database — use standard species-level assignment accuracy evaluation
  instead
- Input is dominated by short reads; the article notes MetaMaps is not well suited
  to samples with a high proportion of short reads
- The goal is abundance/composition estimation (r2 of EM-frequency estimates) rather
  than robustness to contaminant genomes
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Primary long-read metagenomic classifier under evaluation; run via mapDirectly
    and classify steps against RefSeq to test whether out-of-database eukaryotic reads
    remain unclassified
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Approximate mapping algorithm underlying MetaMaps' minimizer-based mapping
    stage
  repo: https://github.com/marbl/MashMap
- name: Kraken 2
  role: K-mer-based baseline tool whose false-positive classification of contaminant
    reads is compared against MetaMaps' unclassification behavior
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:6b44c912c0b60dfa9fdb62577a567c30e19e995b7b03722a88f98a9f7d1132e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Kraken 2
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# eukaryotic-read-unclassification-precision-recall-evaluation

## Summary

Evaluates whether a long-read metagenomic classifier correctly leaves out-of-database eukaryotic reads unclassified, using simulated contamination sets (e.g., yellow fever mosquito and Toxoplasma gondii reads in MetaMaps experiment e2) to compute precision and recall of non-classification at species, genus, and family levels.

## When to use

Use when testing a metagenomic classifier's robustness to large out-of-database contaminant genomes, by quantifying precision/recall of leaving reads unclassified rather than misassigning them.

## When NOT to use

- The sample is expected to contain only organisms whose genomes or close relatives are in the database — use standard species-level assignment accuracy evaluation instead
- Input is dominated by short reads; the article notes MetaMaps is not well suited to samples with a high proportion of short reads
- The goal is abundance/composition estimation (r2 of EM-frequency estimates) rather than robustness to contaminant genomes

## Inputs

- Simulated ONT/PacBio long-read FASTQ sets containing out-of-database eukaryotic genomes (e.g., e2 yellow fever mosquito and Toxoplasma gondii read sets)
- Comprehensive RefSeq classification database (>12,000 genomes, e.g., miniSeq+H) with taxonomy directory
- Per-read ground-truth labels identifying which reads derive from the out-of-database contaminant

## Outputs

- Per-read classification output (e.g., classification_results.EM.reads2Taxon) marking contaminant reads as classified vs. unclassified
- Precision and recall of leaving out-of-database reads unclassified at species, genus, and family levels (expected >99% for MetaMaps)
- Comparative misclassification rates against k-mer/alignment-based tools (up to ~45% misclassification)

## How to apply

Build the classifier from source (for MetaMaps: ./bootstrap.sh, ./configure --with-boost=/path/to/boost, make metamaps), then run it on simulated long-read sets spiked with out-of-database eukaryotic genomes (experiment e2: yellow fever mosquito and Toxoplasma gondii) against a comprehensive RefSeq reference database with >12,000 genomes. Treat the known contaminant origin of each read as ground truth: a true positive is an out-of-database read left unclassified, and compute precision and recall of non-classification at species, genus, and family levels. Compare against k-mer/alignment-based baselines (Kraken2, Opal, CLARK, MetaOthello), which misclassify up to ~45% of contaminant reads; MetaMaps is expected to keep the large majority unclassified with >99% precision/recall. Optionally follow with per-read mapping/identity analyses (e.g., HMP7 Actinomyces reads and ~82% median-identity mismatch signal on CAMI) to detect database-sample mismatches.

## Related tools

- **MetaMaps** (Primary long-read metagenomic classifier under evaluation; run via mapDirectly and classify steps against RefSeq to test whether out-of-database eukaryotic reads remain unclassified) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Approximate mapping algorithm underlying MetaMaps' minimizer-based mapping stage) — https://github.com/marbl/MashMap
- **Kraken 2** (K-mer-based baseline tool whose false-positive classification of contaminant reads is compared against MetaMaps' unclassification behavior)

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q e2_mosquito.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Precision and recall of leaving out-of-database eukaryotic reads unclassified both exceed 99% at species, genus, and family levels on the e2 mosquito and Toxoplasma gondii sets
- The large majority of contaminant reads receive no taxonomic assignment in classification_results.EM.reads2Taxon rather than spurious assignments
- K-mer/alignment-based baselines (Kraken2, Opal, CLARK, MetaOthello) misclassify up to ~45% of contaminant reads, confirming the comparison axis
- Per-read identity/mapping outputs (classification_results.EM.lengthAndIdentitiesPerMappingUnit) show identity shifts away from the ~0.88 mode for mismatched samples (e.g., ~82% median identity on CAMI)

## Limitations

- Complex samples comprising many non-characterized entities remain challenging
- No explicit term for genomic divergence is integrated into the statistical models, limiting explicit detection of novel strains and species
- Analysis of samples with a high proportion of short reads is not well addressed by MetaMaps

## Evidence

- [results] Finding: out-of-database unclassification precision/recall: [evidence span withheld — anchor status: not_found]
- [intro] Article claim of robustness to contaminant genomes: "our approach is robust against the presence of large “contaminant” genomes, introduced during sample collection and processing"
- [readme] README usage of MetaMaps mapping step: [evidence span withheld — anchor status: not_found]
- [discussion] Article limitation on complex samples: "Scenarios with complex samples comprising many non-characterized entities, however, remain challenging"
