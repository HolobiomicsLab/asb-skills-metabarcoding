---
name: cpp-code-coverage-instrumentation
description: Use when you need to measure what fraction of a C++ tool's source is
  executed by its test suite, e.g. to reproduce or verify a reported code-coverage
  benchmark such as swarm's >95% functional-test coverage.
when_to_use_negative:
- You want unit-test style white-box coverage of individual functions rather than
  black-box functional coverage of the assembled binary
- Your interest is runtime performance benchmarking (e.g. the 7x speedup or 50% memory
  reduction claims) — instrumentation flags distort timing and are the wrong instrument
- The code under test is not C++ built with GCC/clang and GNU make, so --coverage/gcov
  aggregation does not apply directly
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: swarm v3
  role: C++ program under test; compiled with --coverage instrumentation and executed
    against the functional test suite
  repo: https://github.com/torognes/swarm
- name: swarm-tests
  role: black-box functional test suite (806 carefully crafted black-box tests) run
    against the instrumented binary to generate coverage data
  repo: frederic-mahe/swarm-tests
- name: GCC
  role: compiler used with --coverage flags so gcov runtime data is emitted during
    execution
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:6c516138b51595650ae95ba0a0162391897449eeb4fc7da46b3ad5d84b512251
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - GCC
  - swarm v3
  - swarm-tests
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# C++ code coverage instrumentation

## Summary

Compile a C++ tool (here swarm v3) with coverage instrumentation (--coverage under GCC/clang), run its black-box functional test suite, and aggregate gcov/lcov data to quantify the fraction of source code exercised by the tests.

## When to use

Use when you need to measure what fraction of a C++ tool's source is executed by its test suite, e.g. to reproduce or verify a reported code-coverage benchmark such as swarm's >95% functional-test coverage.

## When NOT to use

- You want unit-test style white-box coverage of individual functions rather than black-box functional coverage of the assembled binary
- Your interest is runtime performance benchmarking (e.g. the 7x speedup or 50% memory reduction claims) — instrumentation flags distort timing and are the wrong instrument
- The code under test is not C++ built with GCC/clang and GNU make, so --coverage/gcov aggregation does not apply directly

## Inputs

- swarm v3 C++ source code from https://github.com/torognes/swarm
- swarm-tests functional test suite (test_input.sh, test_options.sh, fixed_bugs.sh) from frederic-mahe/swarm-tests
- GCC (g++ >= 4.8.5) or clang (>= 9) toolchain and GNU make

## Outputs

- .gcda gcov runtime coverage files produced by the instrumented binary
- coverage_report.info — a single aggregated lcov/JSON machine-readable coverage report
- aggregate line coverage value across the swarm source (expected > 0.95)

## How to apply

Obtain the source with git clone https://github.com/torognes/swarm.git and compile with GNU make using GCC (g++ 4.8.5 or more recent) or clang (version 9 or more recent), adding --coverage instrumentation flags so gcov runtime data (.gcda files) is emitted during execution. Run the companion frederic-mahe/swarm-tests black-box functional suite (hundreds of hand-crafted test cases covering d = 1, fastidious mode, dereplication, and output options) via bash run_all_tests.sh, which requires bash 4 or higher and optionally valgrind. Aggregate the per-file .gcda data with gcov/lcov into a single machine-readable coverage_report.info report. The rationale is that swarm v3's tracked codecov benchmark claims its test suite covers more than 95% of the code, with the remainder being CPU architecture-specific, so aggregate line coverage should exceed 0.95.

## Related tools

- **swarm v3** (C++ program under test; compiled with --coverage instrumentation and executed against the functional test suite) — https://github.com/torognes/swarm
- **swarm-tests** (black-box functional test suite (806 carefully crafted black-box tests) run against the instrumented binary to generate coverage data) — frederic-mahe/swarm-tests
- **GCC** (compiler used with --coverage flags so gcov runtime data is emitted during execution)

## Examples

```
git clone https://github.com/torognes/swarm.git && cd swarm && make CXXFLAGS='--coverage' && cd ../swarm-tests && bash run_all_tests.sh
```

## Evaluation signals

- coverage_report.info exists and parses as a valid lcov/JSON coverage report
- aggregate line coverage across the swarm source exceeds 0.95, consistent with the project's tracked code-coverage benchmark (codecov badge on the repository)
- the instrumented binary still passes the full functional suite launched with bash run_all_tests.sh
- all .gcda runtime files are produced for the executed source files and aggregated per-file by gcov/lcov

## Limitations

- The residual ~5% of uncovered code is CPU architecture-specific, so aggregate coverage can vary with the build target and platform
- The swarm-tests suite requires bash version 4 or higher, and BSD systems must substitute gmake for make in the build
- Coverage reflects the black-box functional tests only; the README also mentions valgrind as an optional checker, which is a different quality gate

## Evidence

- [other] swarm v3 test suite covers more than 95% of the code, with the remainder being CPU architecture-specific: [evidence span withheld — anchor status: not_found]
- [readme] an updated, hardened, and thoroughly tested code (806 carefully crafted black-box tests): "an updated, hardened, and thoroughly tested code (806 carefully crafted black-box tests)"
- [readme] Compiling swarm only requires make, and GCC (g++ version 4.8.5 or more recent) or clang (version 9 or more recent): "Compiling `swarm` only requires `make`, and GCC (`g++` version 4.8.5 or more recent) or `clang` (version 9 or more recent)"
- [methods] GCC (g++ version 4.8.5 or more recent) or clang from the methods section: "GCC (`g++` version 4.8.5 or more recent) or `clang`"
- [readme] To test a new version of swarm, simply launch: bash run_all_tests.sh: "To test a new version of swarm, simply launch:
sh
bash run_all_tests.sh
"
