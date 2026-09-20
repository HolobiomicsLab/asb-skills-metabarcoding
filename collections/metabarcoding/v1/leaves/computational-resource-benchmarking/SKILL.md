---
name: computational-resource-benchmarking
description: Use when you must reproduce or extend resource measurements (peak RAM,
  CPU hours, runtime slowdown factors) for a tool run in both default and memory-limited
  modes, e.g. verifying MetaMaps' claimed 1.2-1.4x slowdown at 28 GB / 16 GB targets.
when_to_use_negative:
- You only need a single production run and have enough RAM (e.g. 262 GB) for default
  mode — benchmarking mode trade-offs adds no value
- Your samples have a high proportion of short reads or complex, poorly characterized
  entities — MetaMaps itself is out of scope, so its resource trade-offs are not the
  right thing to benchmark
- You need wall-clock accounting of streaming/real-time analysis — MetaMaps does not
  support streaming data
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: 'Tool being benchmarked: run in default mode and in memory-efficient mode
    via --maxmemory 28 GB and 16 GB to measure runtime and RAM trade-offs'
  repo: github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying approximate mapping algorithm used by MetaMaps; relevant to understanding
    memory/runtime behavior being benchmarked
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:a1ad6d57bb69082d44232a18ee6ef1aef94585a8344e5591ecdaa7bfc4c802de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# computational-resource-benchmarking

## Summary

Benchmark a metagenomic classifier's runtime, peak RAM, and accuracy trade-offs across its default and memory-constrained execution modes. For MetaMaps, this means quantifying how the heuristic '--maxmemory' memory-efficient mapping mode increases CPU time relative to default mode (which needs 262 GB RAM for a >12,000-genome RefSeq database) while confirming classification accuracy is preserved.

## When to use

Use when you must reproduce or extend resource measurements (peak RAM, CPU hours, runtime slowdown factors) for a tool run in both default and memory-limited modes, e.g. verifying MetaMaps' claimed 1.2-1.4x slowdown at 28 GB / 16 GB targets.

## When NOT to use

- You only need a single production run and have enough RAM (e.g. 262 GB) for default mode — benchmarking mode trade-offs adds no value
- Your samples have a high proportion of short reads or complex, poorly characterized entities — MetaMaps itself is out of scope, so its resource trade-offs are not the right thing to benchmark
- You need wall-clock accounting of streaming/real-time analysis — MetaMaps does not support streaming data

## Inputs

- Long-read (PacBio/ONT) metagenomic FASTQ files from six experiments (simulated and real, e.g. Zymo mock community GridION/PromethION reads)
- RefSeq-based classification database (miniSeq+H, >12,000 microbial genomes plus human reference, ~8 GB compressed)
- MetaMaps binary built via ./bootstrap.sh, ./configure, make metamaps

## Outputs

- resource_benchmark_table.csv: per-experiment peak RAM, CPU hours, runtime factor vs default mode (default vs --maxmemory 28 GB vs --maxmemory 16 GB)
- Species-level read-assignment accuracy and sample-composition correlation (r2) per experiment and mode, mirroring Supplementary Table 1 / Supplementary Data 5
- Per-mode classification outputs (.EM.WIMP, .EM.reads2Taxon) used to derive accuracy metrics

## How to apply

Run the tool (MetaMaps) in default mode on each of six long-read metagenomic experiments against the >12,000-genome RefSeq (miniSeq+H) database, recording peak RAM and total CPU hours per run — the default run requires 262 GB RAM and 16-210 CPU hours. Re-execute the same experiments in memory-efficient mode using the '--maxmemory' parameter set to 28 GB, then to 16 GB, computing the runtime factor relative to default mode for each target (expected ~1.2-1.3x at 28 GB and ~1.3-1.4x at 16 GB on both simulated and real data). Note the README's caveat that memory-efficient mode is heuristic and actual usage will fluctuate and can exceed the target, so recommend ~70% of available memory as the target. For each run, additionally compute species-level read-assignment accuracy and sample-composition correlation (r2) from the '.EM.WIMP' and '.EM.reads2Taxon' outputs to confirm that accuracy is virtually unaffected between modes. Assemble peak RAM, CPU hours, runtime factors, and accuracy metrics per experiment and mode into a resource benchmark table (corresponding to Supplementary Table 1 / Supplementary Data 5).

## Related tools

- **MetaMaps** (Tool being benchmarked: run in default mode and in memory-efficient mode via --maxmemory 28 GB and 16 GB to measure runtime and RAM trade-offs) — github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying approximate mapping algorithm used by MetaMaps; relevant to understanding memory/runtime behavior being benchmarked) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results --maxmemory 16
```

## Evaluation signals

- Default-mode peak RAM matches the reported ~262 GB against the >12,000-genome RefSeq database, with 16-210 CPU hours across the six experiments
- Memory-efficient runtime factors fall in the expected ranges: ~1.2-1.3x at 28 GB peak memory and ~1.3-1.4x at 16 GB, on both simulated and real data
- Memory-efficient runs stay near their --maxmemory target (subject to the README's heuristic-fluctuation caveat), enabling laptop-scale analysis (<16 GB claim)
- Species-level read-assignment accuracy (>94%) and composition r2 (>0.97) are virtually identical between default and memory-efficient modes
- Benchmark table reproduces the structure and values of Supplementary Table 1 / Supplementary Data 5

## Limitations

- The --maxmemory feature is implemented heuristically; actual memory usage will fluctuate and can exceed the specified target, so ~70% of available RAM is recommended as the target
- Multithreading efficiency can degrade; if problems occur, 'unset MALLOC_ARENA_MAX' before calling MetaMaps
- Reads below the minimum length requirement remain unassigned, and samples with a high proportion of short reads are poorly handled, complicating resource comparisons on such inputs
- No support for streaming data, so real-time/resource-constrained clinical use cases are not covered by the benchmark

## Evidence

- [results] In memory-efficient mode, MetaMaps runtimes increase by a factor of 1.2-1.3 at a peak memory usage of 28 GB and by a factor of 1.3-1.4 at a peak memory usage of 16 GB (tested on both simulated and real data).: [evidence span withheld — anchor status: not_found]
- [readme] You can use the '--maxmemory' parameter to specify a target for maximum memory consumption (in gigabytes). Note that this feature is implemented heuristically; actual memory usage will fluctuate and execeed the target.: [evidence span withheld — anchor status: not_found]
- [abstract] capable of mapping a long-read metagenome to a comprehensive RefSeq database with >12,000 genomes in <16 GB or RAM on a laptop computer: "capable of mapping a long-read metagenome to a comprehensive RefSeq database with >12,000 genomes in <16 GB or RAM on a laptop computer"
- [abstract] MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
