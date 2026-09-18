---
name: code-metric-scoring
description: Use when comparing code-quality refactorings between two releases of
  a C++ (or similar) research codebase, and you need a reproducible numeric global
  quality score per version rather than a qualitative code review.
when_to_use_negative:
- The goal is to measure runtime performance, memory footprint, or clustering accuracy
  rather than source-code quality — swarm v3's 7x speedup and 50% memory reduction
  are separate claims, not captured by SoftWipe scores.
- The two source trees were scored under different SoftWipe configurations, making
  the resulting numbers non-comparable.
- The codebase is not a statically analyzable source tree (e.g. only binaries or notebooks
  are available).
tools:
- name: SoftWipe
  role: Static code-quality tool run on each source tree to produce the global code
    quality score out of 10 (Zapletal et al., 2020)
- name: swarm v2
  role: Baseline C++ source tree whose score (5.2/10) is the reference against which
    the refactored version is compared
  repo: https://github.com/torognes/swarm
- name: swarm v3
  role: Modernized C++ source tree whose score (6.6/10) demonstrates the quality improvement
  repo: https://github.com/torognes/swarm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:4c9c7dfd9b603ff78c61b74b9254d50c7cd1e9a9fa55c22dc06cd38c5eaed003
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - SoftWipe
  - swarm v2
  - swarm v3
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.36363636363636365
n_claims: 11
---

# code-metric-scoring

## Summary

Compute and compare automated source-code quality scores (e.g. SoftWipe, Zapletal et al., 2020) across two software versions, using the swarm v2 vs swarm v3 C++ trees as the reference case where v3 scores 6.6/10 versus v2's 5.2/10 after more than 1600 warning fixes.

## When to use

Use when comparing code-quality refactorings between two releases of a C++ (or similar) research codebase, and you need a reproducible numeric global quality score per version rather than a qualitative code review.

## When NOT to use

- The goal is to measure runtime performance, memory footprint, or clustering accuracy rather than source-code quality — swarm v3's 7x speedup and 50% memory reduction are separate claims, not captured by SoftWipe scores.
- The two source trees were scored under different SoftWipe configurations, making the resulting numbers non-comparable.
- The codebase is not a statically analyzable source tree (e.g. only binaries or notebooks are available).

## Inputs

- swarm v2 C++ source tree (clean checkout from github.com/torognes/swarm)
- swarm v3 C++ source tree (clean checkout from github.com/torognes/swarm)
- SoftWipe tool (Zapletal et al., 2020) with an identical configuration for both runs

## Outputs

- softwipe_scores.csv with one record per version: (version, global code quality score)
- Global code quality scores: 5.2/10 for swarm v2 and 6.6/10 for swarm v3

## How to apply

Obtain clean checkouts of both source trees from the project repository (here, swarm v2 and swarm v3 from torognes/swarm on GitHub). Run SoftWipe (Zapletal et al., 2020) on each source tree with an identical configuration so the scores are comparable, and record one (version, score) pair per version. For swarm, expect the modernized, hardened v3 code — which was reworked with more than 1600 warning fixes and ships 806 carefully crafted black-box tests — to score 6.6/10 versus 5.2/10 for v2. Validate by checking both scores against the reference values within tolerance and confirming that the newer version's score is strictly greater than the older version's.

## Related tools

- **SoftWipe** (Static code-quality tool run on each source tree to produce the global code quality score out of 10 (Zapletal et al., 2020))
- **swarm v2** (Baseline C++ source tree whose score (5.2/10) is the reference against which the refactored version is compared) — https://github.com/torognes/swarm
- **swarm v3** (Modernized C++ source tree whose score (6.6/10) demonstrates the quality improvement) — https://github.com/torognes/swarm

## Evaluation signals

- The swarm v2 score matches the reference value 5.2 out of 10 within tolerance.
- The swarm v3 score matches the reference value 6.6 out of 10 within tolerance.
- The v3 score is strictly greater than the v2 score (6.6 > 5.2).
- Both source trees were scored by SoftWipe runs with identical configuration.
- One record per version (version, score) is written to softwipe_scores.csv.

## Limitations

- The improvement is relative to a specific tool (SoftWipe, Zapletal et al., 2020) and configuration; other quality metrics may rank the versions differently.
- The score gain reflects code hygiene (e.g. warning fixes, more than 1600 fixes for swarm v3) and does not directly measure scientific correctness; swarm v3's correctness evidence is its extensive robustness and logic testing (806 black-box tests in the swarm-tests repository).
- Scores must be reproduced on clean checkouts of the tagged versions; mixing modified or untagged trees invalidates the comparison.

## Evidence

- [abstract] swarm v3 has modernized C++ source code: "swarm v3 has modernized C++ source code"
- [abstract] tested for robustness and logic: "it has been extensively tested for its robustness and logic"
- [readme] hardened and thoroughly tested code: "an updated, hardened, and thoroughly tested code (806 carefully crafted black-box tests)"
- [readme] swarm-tests unit tests: [evidence span withheld — anchor status: not_found]
- [abstract] repo source availability: "Source code and binaries are available at https://github.com/torognes/swarm."
