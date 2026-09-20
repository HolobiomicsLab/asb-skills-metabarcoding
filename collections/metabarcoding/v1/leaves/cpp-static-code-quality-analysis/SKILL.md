---
name: cpp-static-code-quality-analysis
description: Use when comparing code quality between two or more C++ codebase versions,
  or when validating that a refactoring/warning-fix campaign measurably improved static
  code quality.
when_to_use_negative:
- The goal is functional/regression testing of behaviour rather than static code quality
  — the swarm project ships 806 black-box tests via the frederic-mahe/swarm-tests
  suite for that purpose
- The source tree is not C++ or the comparison targets are not distinct tagged versions,
  so the version-over-version score comparison is meaningless
tools:
- name: SoftWipe
  role: Static code-quality analyzer that computes the global code quality score (out
    of 10) for each C++ source tree (Zapletal et al., 2020)
- name: swarm v3
  role: Newer C++ source tree under evaluation; its modernized code scores 6.6/10
  repo: https://github.com/torognes/swarm
- name: swarm v2
  role: Baseline C++ source tree; scores 5.2/10 as the comparison reference
  repo: https://github.com/torognes/swarm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:18c596cb253579fd98c40a25dede17b9b83cb262fa5146abf258e57d5cd16e12
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
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cpp-static-code-quality-analysis

## Summary

Quantify the static code quality of C++ source trees across software versions using SoftWipe (Zapletal et al., 2020), which computes a global code quality score out of 10. Used in the swarm v3 paper (DOI 10.1093/bioinformatics/btab493) to demonstrate that v3's modernized C++ source code raised the score from 5.2 (swarm v2) to 6.6 (swarm v3).

## When to use

Use when comparing code quality between two or more C++ codebase versions, or when validating that a refactoring/warning-fix campaign measurably improved static code quality.

## When NOT to use

- The goal is functional/regression testing of behaviour rather than static code quality — the swarm project ships 806 black-box tests via the frederic-mahe/swarm-tests suite for that purpose
- The source tree is not C++ or the comparison targets are not distinct tagged versions, so the version-over-version score comparison is meaningless

## Inputs

- Clean git checkouts of the C++ source trees to compare (e.g. swarm v2 and swarm v3 from torognes/swarm)
- A single SoftWipe configuration applied identically to every source tree

## Outputs

- softwipe_scores.csv with one record per version (version, global code quality score out of 10)
- Per-version SoftWipe global code quality scores (swarm v2 = 5.2/10, swarm v3 = 6.6/10) and their comparison

## How to apply

Obtain clean checkouts of the two C++ source trees to compare (here, swarm v2 and swarm v3 from the torognes/swarm GitHub repository). Run the SoftWipe tool (Zapletal et al., 2020) on the older tree first, then re-run it with an identical configuration on the newer tree so scores are directly comparable. Collect one record per version (version, score) into softwipe_scores.csv. Validate by confirming the older score matches the reference value (swarm v2 = 5.2 out of 10) and the newer score matches its reference (swarm v3 = 6.6 out of 10) within tolerance, and that the newer score is strictly greater than the older one; the v3 improvement follows from more than 1600 warning fixes during code modernization.

## Related tools

- **SoftWipe** (Static code-quality analyzer that computes the global code quality score (out of 10) for each C++ source tree (Zapletal et al., 2020))
- **swarm v3** (Newer C++ source tree under evaluation; its modernized code scores 6.6/10) — https://github.com/torognes/swarm
- **swarm v2** (Baseline C++ source tree; scores 5.2/10 as the comparison reference) — https://github.com/torognes/swarm

## Examples

```
git clone https://github.com/torognes/swarm.git && cd swarm && make
```

## Evaluation signals

- swarm v2 SoftWipe score matches the reference value 5.2 out of 10 within tolerance
- swarm v3 SoftWipe score matches the reference value 6.6 out of 10 within tolerance
- swarm v3 score is strictly greater than the swarm v2 score (6.6 > 5.2)
- Both scores were produced with an identical SoftWipe configuration on clean source checkouts
- softwipe_scores.csv contains exactly one record per evaluated version with (version, score) fields

## Limitations

- Scores are only comparable when SoftWipe is run with identical configuration on clean checkouts; differing configs invalidate the comparison
- The global code quality score reflects static code properties (warning counts etc.) and does not certify functional correctness, which the swarm project addresses separately via 806 carefully crafted black-box tests
- Reference values (5.2 and 6.6) are tied to this specific swarm v2/v3 comparison and may not generalize to other codebases

## Evidence

- [abstract] Swarm v3 abstract — modernized code: "When compared with previous swarm versions, swarm v3 has modernized C++ source code"
- [other] Task card finding — SoftWipe scores: [evidence span withheld — anchor status: not_found]
- [readme] README — hardened and tested code: "an updated, hardened, and thoroughly tested code (806 carefully crafted black-box tests)"
- [readme] README — compilation toolchain: "Compiling `swarm` only requires `make`, and GCC (`g++` version 4.8.5 or more recent) or `clang`"
