---
name: memory-constrained-execution-tuning
description: Use when a default-mode MetaMaps run against the >12,000-genome miniSeq+H/RefSeq
  database needs ~262 GB RAM but the target machine offers far less (e.g. 28 GB or
  16 GB), and a small runtime penalty is acceptable.
when_to_use_negative:
- The machine has enough RAM (~262 GB) for default-mode mapping, since memory-efficient
  mode only adds 1.2-1.4x runtime overhead without memory-pressure benefits
- The sample contains a high proportion of short reads or reads below the minimum
  length requirement, which MetaMaps handles poorly regardless of memory mode
- Real-time/streaming analysis is required — MetaMaps has no streaming-data support,
  and this skill addresses memory capping, not latency
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: The metagenomic classifier whose mapDirectly step is executed with the --maxmemory
    parameter to cap peak memory usage in gigabytes
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying minimizer-based approximate mapping algorithm whose memory footprint
    the --maxmemory heuristic regulates
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:8abb785b3a47e2ae85d105b94569f2f70d3aa61d9b4511318ad49dcb2962c749
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# memory-constrained-execution-tuning

## Summary

Run MetaMaps' mapping step in memory-efficient mode using the --maxmemory parameter to cap peak RAM (e.g. 28 GB or 16 GB) when a default-mode run against the >12,000-genome RefSeq database (~262 GB RAM) exceeds available machine memory, accepting a modest 1.2-1.4x runtime increase with virtually no loss of assignment accuracy.

## When to use

Use when a default-mode MetaMaps run against the >12,000-genome miniSeq+H/RefSeq database needs ~262 GB RAM but the target machine offers far less (e.g. 28 GB or 16 GB), and a small runtime penalty is acceptable.

## When NOT to use

- The machine has enough RAM (~262 GB) for default-mode mapping, since memory-efficient mode only adds 1.2-1.4x runtime overhead without memory-pressure benefits
- The sample contains a high proportion of short reads or reads below the minimum length requirement, which MetaMaps handles poorly regardless of memory mode
- Real-time/streaming analysis is required — MetaMaps has no streaming-data support, and this skill addresses memory capping, not latency

## Inputs

- Long-read (PacBio/Oxford Nanopore) metagenomic FASTQ files
- MetaMaps classification database (e.g. miniSeq+H: >12,000 RefSeq microbial genomes plus human reference, DB.fa)
- User-specified peak memory target in gigabytes (e.g. 28 or 16)
- Baseline default-mode peak-RAM and CPU-hour measurements for runtime-factor comparison

## Outputs

- classification_results.EM.WIMP sample-composition table (Absolute, EMFrequency, PotFrequency columns)
- classification_results.EM.reads2Taxon per-read taxon assignments
- classification_results.EM complete approximate mapping file (mapping locations, identities, mapping qualities)
- Resource benchmark table of peak RAM, CPU hours, and runtime factor per experiment and mode (Supplementary Table 1 / Supplementary Data 5 style CSV)

## How to apply

Re-run the same long-read metagenomic mapping command in memory-efficient mode by adding the --maxmemory flag to 'metamaps mapDirectly', specifying the peak-memory target in gigabytes (e.g. --maxmemory 28 or --maxmemory 16); the README recommends targeting roughly 70% of available machine memory. Note the cap is heuristic — actual usage fluctuates and may exceed the target — so still record observed peak RAM per run. Measure the runtime factor relative to default mode (expected ~1.2-1.3x at 28 GB and ~1.3-1.4x at 16 GB on simulated and real data) and verify species-level read-assignment accuracy (>94%) and sample-composition correlation (r2 > 0.97) are virtually unaffected before adopting the constrained run's outputs. Multithreading (-t) can be combined with the memory cap to offset some of the runtime cost.

## Related tools

- **MetaMaps** (The metagenomic classifier whose mapDirectly step is executed with the --maxmemory parameter to cap peak memory usage in gigabytes) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying minimizer-based approximate mapping algorithm whose memory footprint the --maxmemory heuristic regulates) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results --maxmemory 16
```

## Evaluation signals

- Runtime factor relative to default mode falls in the expected range: ~1.2-1.3x at a 28 GB target and ~1.3-1.4x at a 16 GB target, on both simulated and real data
- Species-level read-assignment accuracy remains >94% and sample-composition r2 remains >0.97 compared with default mode
- Observed peak RAM stays near the specified target (with awareness that usage fluctuates and can exceed the heuristic cap)
- classification_results.EM output files (WIMP, reads2Taxon, mappings) are produced with the documented column schema (e.g. Absolute/EMFrequency/PotFrequency in WIMP)

## Limitations

- The --maxmemory feature is implemented heuristically; actual memory usage will fluctuate and can exceed the specified target
- The README recommends targeting ~70% of available machine memory (e.g. 20G on a 32G machine) rather than the full capacity
- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned, independent of memory mode

## Evidence

- [readme] You can use the '--maxmemory' parameter to specify a target for maximum memory consumption (in gigabytes). Note that this feature is implemented heuristically; actual memory usage will fluctuate and execeed the target.: [evidence span withheld — anchor status: not_found]
- [readme] We recommend using around 70% of the available memory as a target amount (for example, 20G when you have a 32G machine).: [evidence span withheld — anchor status: not_found]
- [abstract] capable of mapping a long-read metagenome to a comprehensive RefSeq database with >12,000 genomes in <16 GB or RAM on a laptop computer: "capable of mapping a long-read metagenome to a comprehensive RefSeq database with >12,000 genomes in <16 GB or RAM on a laptop computer"
- [supplementary] In memory-efficient mode, MetaMaps runtimes increase by a factor of 1.2-1.3 at a peak memory usage of 28 GB and by a factor of 1.3-1.4 at a peak memory usage of 16 GB: [evidence span withheld — anchor status: not_found]
