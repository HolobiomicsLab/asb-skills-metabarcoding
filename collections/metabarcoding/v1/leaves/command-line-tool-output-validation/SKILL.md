---
name: command-line-tool-output-validation
description: Use when a bioinformatics CLI tool (e.g. swarm v3 clustering millions
  of amplicons) has just been executed and you must confirm the output file exists,
  follows the tool's documented format, and that measured wall-clock runtime is within
  an expected range.
when_to_use_negative:
- The tool produced no output file or crashed mid-run — validate the run itself before
  validating output formatting
- Input FASTA entries lack abundance annotations (e.g. '_1000' suffixes), so swarm
  cannot have run correctly and output validation is moot until dereplication is redone
- The clustering was run with non-default settings (e.g. --fastidious, d >= 2) whose
  expected runtime and cluster structure differ from the d = 1 baseline
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: swarm v3
  role: Clustering tool whose output file (amplicons.swarms) and runtime are being
    validated; run with default d = 1 and --threads 16
  repo: https://github.com/torognes/swarm
- name: swarm-tests
  role: Upstream black-box test suite (806 tests) covering input formats, option values,
    expected output, and regressions; a model for output validation checks
  repo: https://github.com/frederic-mahe/swarm-tests
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:28d094b08df97baba5149032aafa581c8d0a1438717d7c3f9d3f6723a67bb389
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - swarm v3
  - swarm-tests
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# command-line-tool-output-validation

## Summary

Validate that a command-line clustering tool (swarm v3) produced a well-formed output file and a plausible runtime, by checking output format conventions (space-separated amplicon identifiers per cluster, most abundant amplicon first as seed) and timing the run against expected ranges.

## When to use

Use when a bioinformatics CLI tool (e.g. swarm v3 clustering millions of amplicons) has just been executed and you must confirm the output file exists, follows the tool's documented format, and that measured wall-clock runtime is within an expected range.

## When NOT to use

- The tool produced no output file or crashed mid-run — validate the run itself before validating output formatting
- Input FASTA entries lack abundance annotations (e.g. '_1000' suffixes), so swarm cannot have run correctly and output validation is moot until dereplication is redone
- The clustering was run with non-default settings (e.g. --fastidious, d >= 2) whose expected runtime and cluster structure differ from the d = 1 baseline

## Inputs

- swarm output file (e.g. amplicons.swarms) written with the -o option
- runtime record file (e.g. runtime_record) produced by /usr/bin/time or shell time
- dereplicated amplicon FASTA (amplicons.fasta) with abundance-annotated headers, used as run reference

## Outputs

- validated cluster file amplicons.swarms in swarm output format (space-separated identifiers per cluster, seed first)
- recorded wall-clock runtime (runtime_record) checked against expected range (~30–90 min on 16 cores for ~324M sequences)

## How to apply

After running swarm v3 on a dereplicated amplicon FASTA (each entry carrying an abundance annotation such as '_1000' as swarm requires), capture wall-clock time with /usr/bin/time or the shell 'time' builtin and record it (e.g. to runtime_record). Then validate the output: confirm amplicons.swarms exists and follows the swarm output format — clusters as space-separated amplicon identifiers, with the most abundant amplicon first as seed. Anchor expectations to prior evidence: on the UniEuk SSU-rRNA V4 dataset (~324 million unique sequences) with default parameters (local threshold d = 1, built-in cluster breaking) and --threads 16, the recorded runtime should fall within roughly 30–90 minutes. Sanity-check parameters echoed in the run against the documented defaults (d = 1, mandatory strict dereplication) before accepting the result.

## Related tools

- **swarm v3** (Clustering tool whose output file (amplicons.swarms) and runtime are being validated; run with default d = 1 and --threads 16) — https://github.com/torognes/swarm
- **swarm-tests** (Upstream black-box test suite (806 tests) covering input formats, option values, expected output, and regressions; a model for output validation checks) — https://github.com/frederic-mahe/swarm-tests

## Examples

```
/usr/bin/time -v swarm --threads 16 -o amplicons.swarms amplicons.fasta 2> runtime_record
```

## Evaluation signals

- amplicons.swarms exists and is non-empty after the run
- Each line of amplicons.swarms is a space-separated list of amplicon identifiers, with the most abundant amplicon first as the cluster seed
- Recorded wall-clock runtime falls within the expected ~30–90 minute range for the UniEuk V4 dataset (~324M unique sequences) on a 16-core system with --threads 16
- Sequence identifiers in the output match those in the input dereplicated FASTA (unique IDs ending with occurrence counts, e.g. '_1000')
- Total amplicons across all clusters equals the number of input entries (no sequences lost or duplicated between clusters)

## Limitations

- Runtime expectations are dataset- and hardware-specific: the 30–90 minute range applies to the UniEuk SSU-rRNA V4 dataset (~324M unique sequences) on 16 cores; other datasets or thread counts need recalibrated ranges
- Memory footprint of swarm is roughly 0.6 times the input FASTA size and can increase significantly with the fastidious option, so validation on very large inputs must also confirm the process was not killed by OOM
- A 15% slowdown was observed with d >= 2 in certain conditions (18SV9 dataset), so runtime baselines differ for non-default d values

## Evidence

- [results] Using default parameters, swarm v3 clustered the UniEuk SSU-rRNA V4 dataset of nearly 324 million unique sequences in 50 minutes on a 16-core system.: [evidence span withheld — anchor status: not_found]
- [other] verify that amplicons.swarms exists and follows the swarm output format (space-separated amplicon identifiers per cluster, most abundant amplicon first as seed): [evidence span withheld — anchor status: not_found]
- [methods] The chain-breaking, which used to be performed in a second step in **swarm** 1.0, is now built-in and performed by default.: "The chain-breaking, which used to be performed in a second step in **swarm** 1.0, is now built-in and performed by default."
- [readme] Swarm **requires** each fasta entry to present a number of occurrences to work properly.: "Swarm **requires** each fasta entry to present a number of occurrences to work properly."
- [abstract] it has been extensively tested for its robustness and logic: "it has been extensively tested for its robustness and logic"
