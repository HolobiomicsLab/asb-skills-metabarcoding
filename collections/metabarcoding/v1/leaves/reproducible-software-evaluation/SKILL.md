---
name: reproducible-software-evaluation
description: Use when a paper claims a code-quality or software-engineering improvement
  between two versions of a tool (e.g. swarm v3's 'modernized C++ source code' over
  v2) and the claim must be reproduced with a measurable score.
when_to_use_negative:
- The claim to verify concerns runtime performance, memory footprint, or clustering
  results rather than code quality (for swarm v3, speed and memory claims are evaluated
  separately as 'more than 7 times faster' and 'reduced memory footprint by up to
  50%').
- Only one version of the software is available, since the skill requires a paired
  before/after comparison of two source trees under identical SoftWipe configuration.
- The software under study has no obtainable clean source checkout (e.g. binaries
  only), making SoftWipe static source analysis impossible.
tools:
- name: SoftWipe
  role: Static code-quality analysis tool run with identical configuration on each
    source tree to compute the global code quality score out of 10
- name: swarm v2
  role: Baseline source tree whose SoftWipe score (5.2/10) serves as the reference
    for comparison
  repo: https://github.com/torognes/swarm
- name: swarm v3
  role: Treated source tree whose SoftWipe score (6.6/10) is compared against the
    v2 baseline
  repo: https://github.com/torognes/swarm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:4ab96833ab7ffe7e09b807876e33f402f26816905e5918d78506f384da98e655
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
yes_rate: 0.8
n_claims: 5
---

# reproducible-software-evaluation

## Summary

Quantitatively compare code quality across software versions by running SoftWipe (Zapletal et al., 2020) with an identical configuration on clean checkouts of each source tree, then recording one score record per version (e.g. swarm v2 = 5.2/10 vs swarm v3 = 6.6/10) in a CSV for validation.

## When to use

Use when a paper claims a code-quality or software-engineering improvement between two versions of a tool (e.g. swarm v3's 'modernized C++ source code' over v2) and the claim must be reproduced with a measurable score.

## When NOT to use

- The claim to verify concerns runtime performance, memory footprint, or clustering results rather than code quality (for swarm v3, speed and memory claims are evaluated separately as 'more than 7 times faster' and 'reduced memory footprint by up to 50%').
- Only one version of the software is available, since the skill requires a paired before/after comparison of two source trees under identical SoftWipe configuration.
- The software under study has no obtainable clean source checkout (e.g. binaries only), making SoftWipe static source analysis impossible.

## Inputs

- swarm v2 C++ source tree (clean checkout from torognes/swarm)
- swarm v3 C++ source tree (clean checkout from torognes/swarm)
- SoftWipe tool with a fixed configuration

## Outputs

- softwipe_scores.csv with one (version, score) record per source tree
- Global code quality scores out of 10 for each version (swarm v2: 5.2; swarm v3: 6.6)
- Validation verdict confirming v3 score > v2 score and both match reference values within tolerance

## How to apply

Obtain clean checkouts of both source versions from the project's GitHub repository (for swarm, torognes/swarm), so the comparison is not confounded by local edits or build artifacts. Run SoftWipe (Zapletal et al., 2020) on the older source tree first, then run SoftWipe again with an identical configuration on the newer tree to keep the scoring comparable. Collect the two global code quality scores (each out of 10) and write one record per version — version and score — to a CSV file such as softwipe_scores.csv. Validate by checking the older score matches its reference value (swarm v2 = 5.2), the newer score matches its reference value (swarm v3 = 6.6) within tolerance, and that the newer score is strictly greater than the older one. The rationale is that the claimed improvement (after more than 1600 warning fixes) is only verifiable if both trees are scored under the same tool, same settings, and same scoring scale.

## Related tools

- **SoftWipe** (Static code-quality analysis tool run with identical configuration on each source tree to compute the global code quality score out of 10)
- **swarm v2** (Baseline source tree whose SoftWipe score (5.2/10) serves as the reference for comparison) — https://github.com/torognes/swarm
- **swarm v3** (Treated source tree whose SoftWipe score (6.6/10) is compared against the v2 baseline) — https://github.com/torognes/swarm

## Examples

```
git clone https://github.com/torognes/swarm.git && cd swarm/ && make
```

## Evaluation signals

- The swarm v2 global code quality score matches the reference value 5.2 out of 10 within tolerance.
- The swarm v3 global code quality score matches the reference value 6.6 out of 10 within tolerance.
- The swarm v3 score is strictly greater than the swarm v2 score.
- softwipe_scores.csv contains exactly one (version, score) record per analyzed source tree.
- Both SoftWipe runs used an identical configuration, so score differences are attributable to the source trees.

## Limitations

- The scores depend on the SoftWipe version and configuration; results are only reproducible when the same settings are used across both trees.
- The swarm v3 improvement reflects more than 1600 warning fixes and code modernization; the score does not capture runtime performance, memory footprint, or clustering accuracy.
- Source trees must be clean checkouts; local modifications or build artifacts would confound the static analysis.

## Evidence

- [other] Running SoftWipe on both source trees yields a global code quality score of 5.2 out of 10 for swarm v2 and 6.6 out of 10 for swarm v3: [evidence span withheld — anchor status: not_found]
- [abstract] swarm v3 has modernized C++ source code: "swarm v3 has modernized C++ source code"
- [readme] an updated, hardened, and thoroughly tested code (806 carefully crafted black-box tests): "an updated, hardened, and thoroughly tested code (806 carefully crafted black-box tests)"
- [readme] Source code and binaries are available at https://github.com/torognes/swarm: "Source code and binaries are available at https://github.com/torognes/swarm"
