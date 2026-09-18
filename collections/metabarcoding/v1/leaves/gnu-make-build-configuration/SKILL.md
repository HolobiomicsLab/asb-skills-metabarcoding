---
name: gnu-make-build-configuration
description: Use when you need to compile swarm v3 from a cloned git repository (e.g.
  to test a new build, instrument it with --coverage, or verify the swarm-tests suite
  passes) rather than downloading prebuilt binaries.
when_to_use_negative:
- You only need to run swarm for clustering — prebuilt binaries for GNU/Linux, macOS
  or Windows are available on the release page, and swarm is installable via bioconda
  (conda install -c bioconda swarm=3.0.0=hc9558a2_0).
- Your environment lacks GNU make or a compliant compiler (g++ < 4.8.5 or clang <
  9) and you cannot install one — the Makefile uses GNU make syntax and will not build
  under bmake without substituting gmake.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0654
tools:
- name: swarm v3
  role: C++ project compiled from source with GNU make; coverage instrumentation is
    applied to its binary to benchmark the functional test suite
  repo: https://github.com/torognes/swarm
- name: GCC
  role: g++ version 4.8.5 or more recent is one of the two supported compilers for
    the make-based build
- name: clang
  role: alternative compiler (version 9 or more recent), e.g. make CC="clang-9" CXX="clang++-9"
- name: frederic-mahe/swarm-tests
  role: functional black-box test suite (run_all_tests.sh) executed against the instrumented
    build to emit .gcda coverage data
  repo: https://github.com/torognes/swarm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:64ad8114b6476970d446165a3a345b18b569f6b2f09f136e6a09a226f67f86c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - GCC
  - clang
  - frederic-mahe/swarm-tests
  - swarm v3
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.16666666666666666
n_claims: 6
---

# gnu-make-build-configuration

## Summary

Build the swarm v3 C++ amplicon clustering tool from source using GNU make with a compliant compiler (GCC g++ 4.8.5+ or clang 9+), optionally adding coverage instrumentation flags such as --coverage to emit gcov data for test-suite coverage measurement.

## When to use

Use when you need to compile swarm v3 from a cloned git repository (e.g. to test a new build, instrument it with --coverage, or verify the swarm-tests suite passes) rather than downloading prebuilt binaries.

## When NOT to use

- You only need to run swarm for clustering — prebuilt binaries for GNU/Linux, macOS or Windows are available on the release page, and swarm is installable via bioconda (conda install -c bioconda swarm=3.0.0=hc9558a2_0).
- Your environment lacks GNU make or a compliant compiler (g++ < 4.8.5 or clang < 9) and you cannot install one — the Makefile uses GNU make syntax and will not build under bmake without substituting gmake.

## Inputs

- swarm v3 source code cloned from https://github.com/torognes/swarm
- GNU make (or gmake on BSD systems)
- GCC (g++ >= 4.8.5) or clang (>= 9) toolchain
- Optional --coverage instrumentation flags for gcov/lcov coverage collection

## Outputs

- Compiled swarm binary at ./bin/swarm
- Optional man page (swarm.1.gz)
- gcov .gcda runtime coverage files when built with --coverage and exercised by swarm-tests
- Aggregated lcov/JSON coverage report showing aggregate line coverage > 0.95

## How to apply

Clone the official repository (git clone https://github.com/torognes/swarm.git), enter the swarm/ directory, and run GNU make, which is the only build requirement beyond a compiler. Compiling requires GNU make syntax (use gmake on BSD systems where the default make is bmake) and either GCC g++ version 4.8.5 or more recent, or clang version 9 or more recent (e.g. make CC="clang-9" CXX="clang++-9"). For coverage benchmarking of the functional test suite, add coverage instrumentation flags (--coverage) to the build so .gcda runtime data is emitted when the instrumented binary executes the swarm-tests black-box cases. The build produces ./bin/swarm, which can be copied to /usr/local/bin/ for system-wide installation, and the man page can be generated with gzip -c swarm.1 > swarm.1.gz. Validating the build means the instrumented binary runs the hundreds of hand-crafted black-box test cases (d = 1, fastidious mode, dereplication, output options) successfully and yields gcov/lcov data showing aggregate line coverage above 0.95.

## Related tools

- **swarm v3** (C++ project compiled from source with GNU make; coverage instrumentation is applied to its binary to benchmark the functional test suite) — https://github.com/torognes/swarm
- **GCC** (g++ version 4.8.5 or more recent is one of the two supported compilers for the make-based build)
- **clang** (alternative compiler (version 9 or more recent), e.g. make CC="clang-9" CXX="clang++-9")
- **frederic-mahe/swarm-tests** (functional black-box test suite (run_all_tests.sh) executed against the instrumented build to emit .gcda coverage data) — https://github.com/torognes/swarm

## Examples

```
git clone https://github.com/torognes/swarm.git; cd swarm/; make CC="clang-9" CXX="clang++-9"
```

## Evaluation signals

- make completes without errors and produces the ./bin/swarm binary
- ./swarm --version reports the expected swarm 3.x version
- The instrumented binary executes the full swarm-tests suite (bash run_all_tests.sh) with all 806 hand-crafted black-box tests passing
- coverage_report.info exists, parses as a valid lcov/JSON coverage report
- Aggregate line coverage across the swarm source exceeds 0.95, with the remainder being CPU architecture-specific code

## Limitations

- The remainder of code not covered by the test suite is CPU architecture-specific, so 100% coverage is not expected.
- On BSD systems the default make is bmake; the gmake package must be installed and substituted or the build will fail.
- OpenBSD and NetBSD builds should work with the same recipe but are not regularly tested.
- Building requires GNU make syntax and g++ >= 4.8.5 or clang >= 9; older compilers are unsupported.

## Evidence

- [readme] Compiling `swarm` only requires `make`, and GCC (`g++` version 4.8.5 or more recent) or `clang` (version 9 or more recent): "Compiling `swarm` only requires `make`, and GCC (`g++` version 4.8.5 or more recent) or `clang` (version 9 or more recent)"
- [readme] git clone https://github.com/torognes/swarm.git
cd swarm/
make: "git clone https://github.com/torognes/swarm.git
cd swarm/
make"
- [readme] The `Makefile` uses GNU `make` syntax. On BSD systems (where the default `make` is `bmake`), install the `gmake` package and substitute `gmake` for `make`: "The `Makefile` uses GNU `make` syntax. On BSD systems (where the default `make` is `bmake`), install the `gmake` package"
- [readme] an updated, hardened, and thoroughly tested code (806 carefully crafted black-box tests): "an updated, hardened, and thoroughly tested code (806 carefully crafted black-box tests)"
- [other] Compile swarm using GNU make with GCC (g++ version 4.8.5 or more recent) or clang, adding coverage instrumentation flags (--coverage) so that gcov runtime data is emitted: [evidence span withheld — anchor status: not_found]
