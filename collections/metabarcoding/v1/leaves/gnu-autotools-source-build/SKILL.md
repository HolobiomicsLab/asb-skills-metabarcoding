---
name: gnu-autotools-source-build
description: Use when MAPseq v1.2.6 (or later) must be compiled from the jfmrod/MAPseq
  repository rather than installed from a binary release, e.g. before reproducing
  runtime benchmarks or mock-community abundance comparisons.
when_to_use_negative:
- A pre-built binary package from the GitHub Releases page (e.g. mapseq-1.2-linux.tar.gz
  or mapseq-1.2-macosx.tar.gz) is sufficient — unpacking it avoids the whole autotools
  build chain
- The target system lacks the required build prerequisites (autoconf, libtool, libncurses5-dev,
  gcc); installing the binary package or a container image is the better route
- The task only requires running MAPseq on already-classified .mseq outputs (e.g.
  'mapseq -otucounts' or '-otutable' summarisation) with no rebuild needed
tools:
- name: MAPseq
  role: Software being built from source (v1.2.6); reference-based k-mer rRNA classifier
    with confidence estimates produced by the build
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: Generates the './configure' script from 'configure.ac' during the bootstrap
    step
- name: GNU make
  role: Drives the compile ('make'), self-test ('make check'), and install ('make
    install') steps of the standard GNU build sequence
- name: gcc
  role: C compiler invoked by the generated Makefiles to compile MAPseq
merged_aliases:
- autotools-source-build-configuration
merged_alias_records:
- alias: autotools-source-build-configuration
  slug: autotools-source-build-configuration
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:9b97363a65b57d4cbcecea8e080a53c9416b5902e5f5541bb1a3c43626af8ebf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - GNU make
  - MAPseq
  - autoconf
  - gcc
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 7
---

# gnu-autotools-source-build

## Summary

Build the MAPseq rRNA classification tool from its GitHub source using the standard GNU autotools sequence — ./setup.sh/bootstrap, ./configure, make, make install — so the reference-based k-mer search with confidence estimates can be reproduced for benchmarks and mock-community analyses.

## When to use

Use when MAPseq v1.2.6 (or later) must be compiled from the jfmrod/MAPseq repository rather than installed from a binary release, e.g. before reproducing runtime benchmarks or mock-community abundance comparisons.

## When NOT to use

- A pre-built binary package from the GitHub Releases page (e.g. mapseq-1.2-linux.tar.gz or mapseq-1.2-macosx.tar.gz) is sufficient — unpacking it avoids the whole autotools build chain
- The target system lacks the required build prerequisites (autoconf, libtool, libncurses5-dev, gcc); installing the binary package or a container image is the better route
- The task only requires running MAPseq on already-classified .mseq outputs (e.g. 'mapseq -otucounts' or '-otutable' summarisation) with no rebuild needed

## Inputs

- MAPseq v1.2.6 source code cloned from https://github.com/jfmrod/mapseq (git clone)
- Build environment with autoconf/autotools, GNU make, gcc, svn, wget, git, libncurses5-dev, and libtool

## Outputs

- Compiled and installed 'mapseq' executable with its accompanying reference data files (default at /usr/local/, or $HOME/usr/bin/mapseq with --prefix=$HOME/usr)
- Optional self-test results from 'make check'

## How to apply

Clone the repository with 'git clone https://github.com/jfmrod/MAPseq.git'. Because the repository (unlike the release tarballs) lacks a generated 'configure' script, first run './setup.sh' (downloads eutils and the reference data files) and './bootstrap' to generate the configure script via autoconf; this requires autotools/autoconf, svn, wget, git, libncurses5-dev, libtool, and GNU make (on Ubuntu: 'sudo apt-get install build-essential wget subversion git libncurses5-dev libtool'). Then run './configure' (add '--prefix=$HOME/usr' to install into the home directory instead of the default system-wide /usr/local/), followed by 'make' to compile and 'make install' to install the binaries; optionally run 'make check' to execute the package self-tests. Rationale: the article and README specify this exact GNU build sequence for MAPseq v1.2.6 (24 Mar 2020), and the data files are searched for relative to the binary's path, so the whole installed directory must be kept together rather than moving only the binary.

## Related tools

- **MAPseq** (Software being built from source (v1.2.6); reference-based k-mer rRNA classifier with confidence estimates produced by the build) — https://github.com/jfmrod/mapseq
- **autoconf** (Generates the './configure' script from 'configure.ac' during the bootstrap step)
- **GNU make** (Drives the compile ('make'), self-test ('make check'), and install ('make install') steps of the standard GNU build sequence)
- **gcc** (C compiler invoked by the generated Makefiles to compile MAPseq)

## Examples

```
git clone https://github.com/jfmrod/MAPseq.git; cd MAPseq; ./setup.sh; ./bootstrap; ./configure --prefix=$HOME/usr; make; make install
```

## Evaluation signals

- './configure' and 'make' complete without errors and produce the 'mapseq' binary in the expected location (e.g. $HOME/usr/bin/mapseq when --prefix=$HOME/usr was used)
- 'make check' (package self-tests) passes, per the standard GNU build instructions cited in the article
- The reference data files remain co-located with the binary: the README warns that 'Moving only the binary elsewhere will break the installation though, as the data files are searched for in relation to the binary's path'
- The installed binary reports MAPseq v1.2.6 (24 Mar 2020) or later, matching the version used in the source cards' workflows
- A smoke test such as 'mapseq rawseqs.fa > rawseqs.fa.mseq' produces .mseq classification output against the default reference dataset

## Limitations

- Compiling from a fresh git clone requires the './bootstrap' step because 'configure' is not pre-generated, and the full prerequisite set (svn, autotools/autoconf, wget, git, libncurses5-dev, libtool) must be present; binary releases avoid this
- Relocating only the mapseq binary breaks the installation because data files are resolved relative to the binary path — the whole directory must be moved
- The build is machine-specific: the article's GNU instructions note that to compile for a different kind of computer you must 'make distclean' first, and multi-architecture builds (e.g. via 'lipo') may need one architecture at a time

## Evidence

- [methods] 'cd' to the directory containing the package's source code and type './configure' to configure the package for your system.: "'cd' to the directory containing the package's source code and type './configure' to configure the package for your system."
- [methods] Type 'make' to compile the package.: "Type 'make' to compile the package."
- [methods] Optionally, type 'make check' to run any self-tests that come with the package: "Optionally, type 'make check' to run any self-tests that come with the package"
- [readme] ./setup.sh      # downloads eutils and the reference data files
./bootstrap     # this step is only needed if you cloned the repository, you will also need to install autotools/autoconf packages: [evidence span withheld — anchor status: external_doc]
- [readme] Moving only the binary elsewhere will break the installation though, as the data files are searched for in relation to the binary's path.: [evidence span withheld — anchor status: external_doc]
- [abstract] Source code and binaries are freely available at https://github.com/jfmrod/mapseq.: "Source code and binaries are freely available at https://github.com/jfmrod/mapseq."
