---
name: lcov-coverage-report-generation
description: Use when you must quantify the fraction of a C++ program's source lines
  covered by its functional test suite, e.g. verifying a tracked benchmark such as
  swarm v3's >95% code-coverage claim.
when_to_use_negative:
- The goal is unit-test-level statement coverage of individual functions rather than
  whole-program coverage from a black-box functional suite
- The target code is not compiled from source with an instrumenting compiler (no way
  to emit .gcda data)
- You only need the article's reported coverage figure without reproducing it — the
  finding (>95%, remainder CPU architecture-specific) is already published
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3377
tools:
- name: swarm v3
  role: C++ program whose source is instrumented with --coverage and executed under
    its functional test suite to emit coverage data
  repo: https://github.com/torognes/swarm
- name: GCC
  role: compiler used with g++ 4.8.5+ (or clang 9+) to build swarm with coverage instrumentation
    flags
- name: swarm-tests
  role: black-box functional test suite (806 hand-crafted tests) run against the instrumented
    binary to generate .gcda files
  repo: https://github.com/frederic-mahe/swarm-tests
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:4d66f4403f7136e05871d5d114b6966ca567b78962fffd482e04ec3de1703824
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - GCC
  - swarm v3
  - swarm-tests
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3377
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# lcov-coverage-report-generation

## Summary

Instrument a C++ codebase (e.g. swarm v3) with GCC/clang coverage flags, run its black-box functional test suite, and aggregate .gcda runtime data into a machine-readable lcov/gcov line-coverage report to quantify how much of the source is exercised by the tests.

## When to use

Use when you must quantify the fraction of a C++ program's source lines covered by its functional test suite, e.g. verifying a tracked benchmark such as swarm v3's >95% code-coverage claim.

## When NOT to use

- The goal is unit-test-level statement coverage of individual functions rather than whole-program coverage from a black-box functional suite
- The target code is not compiled from source with an instrumenting compiler (no way to emit .gcda data)
- You only need the article's reported coverage figure without reproducing it — the finding (>95%, remainder CPU architecture-specific) is already published

## Inputs

- swarm v3 C++ source code from https://github.com/torognes/swarm
- swarm-tests functional test suite (test_input.sh, test_options.sh, fixed_bugs.sh driven by run_all_tests.sh)
- compiler toolchain: GNU make with GCC (g++ 4.8.5+) or clang (9+)

## Outputs

- .gcda gcov runtime coverage data files generated during test execution
- coverage_report.info — a valid lcov/JSON aggregate coverage report
- aggregate line-coverage fraction across the swarm source (expected > 0.95)

## How to apply

Clone the source (e.g. git clone https://github.com/torognes/swarm.git) and the companion test suite (frederic-mahe/swarm-tests). Compile with GNU make using GCC (g++ >= 4.8.5) or clang (>= 9), adding --coverage instrumentation flags so gcov runtime data (.gcda files) is emitted on execution. Run the black-box functional tests (bash run_all_tests.sh from swarm-tests, covering d = 1, fastidious mode, dereplication, and output options) against the instrumented binary. Aggregate the per-file coverage data with gcov/lcov (or an equivalent JSON emitter) into a single machine-readable report such as coverage_report.info. Validate that the report parses and that aggregate line coverage exceeds the expected threshold (>0.95 for swarm v3, with the remainder being CPU architecture-specific code).

## Related tools

- **swarm v3** (C++ program whose source is instrumented with --coverage and executed under its functional test suite to emit coverage data) — https://github.com/torognes/swarm
- **GCC** (compiler used with g++ 4.8.5+ (or clang 9+) to build swarm with coverage instrumentation flags)
- **swarm-tests** (black-box functional test suite (806 hand-crafted tests) run against the instrumented binary to generate .gcda files) — https://github.com/frederic-mahe/swarm-tests

## Examples

```
git clone https://github.com/torognes/swarm.git && cd swarm && make CXXFLAGS="--coverage" && bash ../swarm-tests/run_all_tests.sh && lcov --capture --directory . --output-file coverage_report.info
```

## Evaluation signals

- coverage_report.info exists and parses as a valid lcov/JSON coverage report
- aggregate line coverage across the swarm source exceeds 0.95
- .gcda runtime coverage files are produced after running the swarm-tests suite
- uncovered remainder is attributable to CPU architecture-specific code, consistent with the project's tracked benchmark
- all tests in run_all_tests.sh pass (present and future versions of swarm should pass all tests)

## Limitations

- The remaining uncovered code is CPU architecture-specific, so a true 100% is not attainable on a single platform
- Coverage instrumentation with --coverage can slow execution; the README notes a 15% slowdown was observed for d >= 2 under certain conditions (18SV9 dataset) in another context, so runtime may increase
- swarm-tests requires bash version 4 or higher (valgrind optional)

## Evidence

- [methods] GCC build flags: "GCC (`g++` version 4.8.5 or more recent) or `clang`"
- [readme] test suite invocation: "To test a new version of swarm, simply launch:
sh
bash run_all_tests.sh
"
- [readme] clone instructions: "git clone https://github.com/torognes/swarm.git
cd swarm/
make"
- [abstract] test robustness claim: [evidence span withheld — anchor status: not_found]
- [readme] thoroughly tested code: "an updated, hardened, and thoroughly tested code (806 carefully
  crafted black-box tests)"
