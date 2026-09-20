---
name: read-assignment-accuracy-evaluation
description: Use when MetaMaps (or a comparable long-read metagenomic classifier)
  output must be quality-checked against known ground truth, e.g. simulated data or
  mock communities, to quantify per-read assignment accuracy and composition fidelity.
when_to_use_negative:
- Samples with no ground truth (novel environmental samples) — accuracy and r2 cannot
  be computed, only composition estimated
- Samples with a high proportion of short reads, which MetaMaps explicitly does not
  handle well (reads below the minimum length remain unassigned)
- Complex samples comprising many non-characterized entities, where database-sample
  incongruity invalidates accuracy interpretation
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Long-read metagenomic classifier whose mapping ('mapDirectly') and classification
    ('classify') outputs are scored for read-assignment accuracy and composition r2
  repo: https://github.com/DiltheyLab/MetaMaps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:d2b4b008a718465408e5f696f8da3428593d52c4118bb04b183ff40701b3fb5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# read-assignment-accuracy-evaluation

## Summary

Evaluate MetaMaps classification quality by computing species-level read-assignment accuracy and sample-composition correlation (r2) from its EM classification outputs, e.g. when comparing default vs memory-efficient runs. This verifies that resource-saving modes do not degrade assignment accuracy.

## When to use

Use when MetaMaps (or a comparable long-read metagenomic classifier) output must be quality-checked against known ground truth, e.g. simulated data or mock communities, to quantify per-read assignment accuracy and composition fidelity.

## When NOT to use

- Samples with no ground truth (novel environmental samples) — accuracy and r2 cannot be computed, only composition estimated
- Samples with a high proportion of short reads, which MetaMaps explicitly does not handle well (reads below the minimum length remain unassigned)
- Complex samples comprising many non-characterized entities, where database-sample incongruity invalidates accuracy interpretation

## Inputs

- MetaMaps classification output files (classification_results.EM.WIMP, classification_results.EM.reads2Taxon)
- Long-read metagenomic FASTQ datasets (PacBio/Oxford Nanopore), simulated or real mock community
- Ground-truth species composition / per-read species labels from simulation or mock community (e.g. Zymo mock community)

## Outputs

- Species-level read-assignment accuracy per experiment and mode
- Sample-composition correlation (r2) per experiment and mode
- Resource benchmark table (peak RAM, CPU hours, runtime factors, accuracy metrics), e.g. resource_benchmark_table.csv

## How to apply

Run MetaMaps on each long-read metagenomic experiment (simulated and/or real data) against the >12,000-genome RefSeq-based miniSeq+H database, producing the '*.EM.WIMP' composition file and '*.EM.reads2Taxon' per-read assignments. For experiments with known ground truth, parse reads2Taxon to compute species-level read-assignment accuracy (fraction of reads assigned to the correct species) and compare the WIMP-computed composition against the true composition using the coefficient of determination r2. Repeat the computation for each run mode (default vs '--maxmemory 28' vs '--maxmemory 16') so accuracy metrics can be compared across modes. The paper reports >94% species-level assignment accuracy and r2 > 0.97 when sample genomes or close relatives are in the database; accuracy should be 'virtually unaffected' between default and memory-efficient modes. Assemble per-experiment metrics alongside peak RAM and CPU hours into a resource benchmark table.

## Related tools

- **MetaMaps** (Long-read metagenomic classifier whose mapping ('mapDirectly') and classification ('classify') outputs are scored for read-assignment accuracy and composition r2) — https://github.com/DiltheyLab/MetaMaps

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results --maxmemory 16; ./metamaps classify -t 5 --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Species-level read-assignment accuracy exceeds 94% on simulated and real data when sample genomes or close relatives are in the database
- Composition correlation r2 exceeds 0.97 against known sample composition
- Assignment accuracy and composition r2 are virtually unchanged between default mode and memory-efficient runs ('--maxmemory 28' and '--maxmemory 16')
- reads2Taxon taxon IDs beginning with 'x' resolve correctly against the extended database taxonomy (sub-directory 'taxonomy' of the database directory)
- WIMP output contains the expected columns (Absolute, EMFrequency, PotFrequency) with EMFrequency computed at the 'definedGenomes' level

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned, depressing apparent accuracy for short-read-rich samples
- Database-sample mismatches (sample genomes or close relatives absent from the database) reduce assignment accuracy; visual analysis or automatic thresholding can help detect these cases
- Complex samples with many non-characterized entities remain challenging, and the '--maxmemory' target is heuristic so actual memory usage will fluctuate and exceed the target

## Evidence

- [abstract] >94% accuracy for species-level read assignment and r2 > 0.97 for estimation of sample composition on simulated and real data: [evidence span withheld — anchor status: not_found]
- [full_text] memory-efficient runtime factors: [evidence span withheld — anchor status: not_found]
- [full_text] accuracy unaffected by memory-efficient mode: [evidence span withheld — anchor status: not_found]
- [readme] maxmemory parameter in README: [evidence span withheld — anchor status: not_found]
- [discussion] short reads unassigned: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
