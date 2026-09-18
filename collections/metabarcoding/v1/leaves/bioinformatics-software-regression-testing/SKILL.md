---
name: bioinformatics-software-regression-testing
description: Use when a new version of a clustering or sequence-analysis tool (e.g.
  swarm v3) must be checked against its companion test suite, or when a machine-readable
  code-coverage report (gcov/lcov) must be produced to confirm a tracked coverage
  benchmark.
when_to_use_negative:
- The goal is statistical validation of clustering output quality rather than software
  regression testing — swarm's clustering behaviour (d = 1, fastidious) is assessed
  on biological datasets, not by code coverage.
- The tool under test has no companion functional test suite or coverage instrumentation
  support, so gcov/lcov aggregation cannot be grounded.
- Only per-option unit checks are needed on an already-installed binary without recompiling
  from source.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: swarm v3
  role: C++ amplicon clustering program whose source is instrumented with --coverage
    and exercised by the test suite
  repo: https://github.com/torognes/swarm
- name: swarm-tests
  role: companion black-box functional test suite (run_all_tests.sh) gathering test_input.sh,
    test_options.sh and fixed_bugs.sh
  repo: https://github.com/torognes/swarm
- name: GCC
  role: compiler (g++ 4.8.5 or more recent) used to build swarm with coverage instrumentation
    flags
- name: clang
  role: alternative compiler (version 9 or more recent) accepted by swarm's Makefile
- name: valgrind
  role: optional memory-checking dependency when running the swarm-tests suite
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:85874c172af8551dc938e30e49128508f60d5d3c4ea54f43b5c1d4348c1cb6bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - GCC
  - clang
  - swarm v3
  - swarm-tests
  - valgrind
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.38461538461538464
n_claims: 13
---

# bioinformatics-software-regression-testing

## Summary

Reproduce and validate the functional test suite of a bioinformatics C++ tool (here swarm v3) by compiling with coverage instrumentation and running hand-crafted black-box tests, so that regressions and the >95% code coverage benchmark can be verified on a new version.

## When to use

Use when a new version of a clustering or sequence-analysis tool (e.g. swarm v3) must be checked against its companion test suite, or when a machine-readable code-coverage report (gcov/lcov) must be produced to confirm a tracked coverage benchmark.

## When NOT to use

- The goal is statistical validation of clustering output quality rather than software regression testing — swarm's clustering behaviour (d = 1, fastidious) is assessed on biological datasets, not by code coverage.
- The tool under test has no companion functional test suite or coverage instrumentation support, so gcov/lcov aggregation cannot be grounded.
- Only per-option unit checks are needed on an already-installed binary without recompiling from source.

## Inputs

- swarm v3 source code from git clone https://github.com/torognes/swarm.git
- frederic-mahe/swarm-tests functional test suite (test_input.sh, test_options.sh, fixed_bugs.sh)
- GCC (g++ >= 4.8.5) or clang (>= 9) toolchain with GNU make
- instrumented swarm binary built with --coverage flags

## Outputs

- pass/fail results from bash run_all_tests.sh across all three test categories
- .gcda runtime coverage files emitted by the instrumented binary
- coverage_report.info: a valid lcov/JSON coverage report with aggregate line coverage > 0.95

## How to apply

Clone the tool's official source (https://github.com/torognes/swarm) and compile with GNU make using GCC (g++ 4.8.5 or more recent) or clang (version 9 or more recent), adding --coverage flags so gcov emits .gcda runtime data during execution. Run the companion frederic-mahe/swarm-tests functional suite — hundreds of hand-crafted black-box tests spanning d = 1, fastidious mode, mandatory dereplication, input formats (test_input.sh), option values (test_options.sh) and fixed bugs (fixed_bugs.sh) — by launching bash run_all_tests.sh under bash version 4 or higher, optionally under valgrind. Aggregate per-file .gcda coverage data with gcov/lcov into a single machine-readable coverage_report.info. Validate that the report parses and that aggregate line coverage exceeds 0.95, with the remainder being CPU architecture-specific code.

## Related tools

- **swarm v3** (C++ amplicon clustering program whose source is instrumented with --coverage and exercised by the test suite) — https://github.com/torognes/swarm
- **swarm-tests** (companion black-box functional test suite (run_all_tests.sh) gathering test_input.sh, test_options.sh and fixed_bugs.sh) — https://github.com/torognes/swarm
- **GCC** (compiler (g++ 4.8.5 or more recent) used to build swarm with coverage instrumentation flags)
- **clang** (alternative compiler (version 9 or more recent) accepted by swarm's Makefile)
- **valgrind** (optional memory-checking dependency when running the swarm-tests suite)

## Examples

```
git clone https://github.com/torognes/swarm.git; cd swarm && make; bash run_all_tests.sh
```

## Evaluation signals

- coverage_report.info exists and parses as a valid lcov/JSON coverage report
- aggregate line coverage across the swarm source exceeds 0.95, consistent with the project's tracked benchmark
- all 806 hand-crafted black-box tests pass: bash run_all_tests.sh exits without failures across input formats, option values and fixed-bug regressions
- per-file .gcda runtime data files were actually emitted during test execution, confirming instrumentation was active
- remaining uncovered lines are confined to CPU architecture-specific code

## Limitations

- Residual uncovered code is CPU architecture-specific, so aggregate coverage will not reach 100% by design.
- The test suite requires bash version 4 or higher; valgrind checking is only optional, not part of the base run.
- Compiling requires GNU make syntax — on BSD systems gmake must substitute for make — and g++ 4.8.5+ or clang 9+; older toolchains are unsupported.

## Evidence

- [readme] an updated, hardened, and thoroughly tested code (806 carefully crafted black-box tests): "an updated, hardened, and thoroughly tested code (806 carefully crafted black-box tests)"
- [readme] Tests are gathered into three categories: [evidence span withheld — anchor status: not_found]
- [readme] Compile only requires make and GCC: "Compiling `swarm` only requires `make`, and GCC (`g++` version 4.8.5 or more recent) or `clang` (version 9 or more recent)"
- [methods] GCC or clang build requirement from article methods: "GCC (`g++` version 4.8.5 or more recent) or `clang` (version 9 or more recent)"
- [readme] Tests must pass for present and future versions: [evidence span withheld — anchor status: not_found]
