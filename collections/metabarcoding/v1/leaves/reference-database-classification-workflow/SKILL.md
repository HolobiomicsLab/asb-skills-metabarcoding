---
name: reference-database-classification-workflow
description: Use when benchmarking rRNA amplicon classifiers on reads of known identity,
  or when reproducing MAPseq's reported maxima of F½ ≈ 0.86 (genus) and F½ ≈ 0.96
  (98%-identity hOTUs).
when_to_use_negative:
- Input reads lack known-identity truth labels, since the F½ comparison against genus
  and 98%-identity hOTU targets requires ground-truth assignments
- The goal is de novo OTU clustering rather than reference-based classification against
  curated full-length rRNA references
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
tools:
- name: MAPseq
  role: k-mer search classifier assigning taxonomy and hierarchical OTU mappings with
    confidence estimates to rRNA reads
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: generates the 'configure' script used to build MAPseq from source
- name: GNU make
  role: compiles and installs MAPseq via 'make' and 'make install'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:238be03756c0ae1e95950cc7f36355c07edaf881cc30e81afcc17ab4ee4c76e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - GNU make
  - MAPseq
  - autoconf
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# reference-database-classification-workflow

## Summary

Benchmark MAPseq (v1.2.6) reference-based rRNA classification by comparing its taxonomy and hierarchical OTU (hOTU, 98% identity) assignments on reads of known identity against baseline tools, scoring each with the F½ metric.

## When to use

Use when benchmarking rRNA amplicon classifiers on reads of known identity, or when reproducing MAPseq's reported maxima of F½ ≈ 0.86 (genus) and F½ ≈ 0.96 (98%-identity hOTUs).

## When NOT to use

- Input reads lack known-identity truth labels, since the F½ comparison against genus and 98%-identity hOTU targets requires ground-truth assignments
- The goal is de novo OTU clustering rather than reference-based classification against curated full-length rRNA references

## Inputs

- FASTA file of demultiplexed, quality-filtered amplicon rRNA reads of known identity (e.g. rawseqs.fa)
- MAPseq reference rRNA databases (default NCBI taxonomy and OTU taxonomies, or custom <customref.fasta> + <customref.tax> files)
- MAPseq v1.2.6 source from https://github.com/jfmrod/mapseq with autotools/autoconf, svn, wget, git, libncurses5-dev, libtool

## Outputs

- MAPseq .mseq classification files with taxonomy and hOTU assignments plus per-level confidences
- Per-tool F½ scores for genus-level mapping (max ≈ 0.86) and 98%-identity hOTU mapping (max ≈ 0.96)
- fhalf_scores.csv table ranking all tools, reproducing the relative ranking of Fig. 1c-f

## How to apply

Obtain MAPseq v1.2.6 from the jfmrod/mapseq GitHub repository and build it with the GNU autotools toolchain ('./setup.sh', './bootstrap' if cloned, './configure', 'make', 'make install'; use './configure --prefix=$HOME/usr' for a user-local install). Run MAPseq on benchmark amplicon reads of known identity against the bundled reference rRNA databases (NCBI taxonomy and OTU taxonomies) using 'mapseq rawseqs.fa > rawseqs.fa.mseq', optionally with custom reference fasta and taxonomy files that specify per-level identity/confidence cutoffs. Run baseline classification tools on the same reads, then compare each tool's genus-level assignments and assignments against hOTUs clustered at 98% sequence identity to the known-identity truth labels, computing F½ scores. Apply a combined confidence cutoff of 0.4 or 0.5, which the authors state yielded the highest F½-score in their benchmarks. Assemble per-tool F½ scores into a single fhalf_scores.csv table.

## Related tools

- **MAPseq** (k-mer search classifier assigning taxonomy and hierarchical OTU mappings with confidence estimates to rRNA reads) — https://github.com/jfmrod/mapseq
- **autoconf** (generates the 'configure' script used to build MAPseq from source)
- **GNU make** (compiles and installs MAPseq via 'make' and 'make install')

## Examples

```
git clone https://github.com/jfmrod/MAPseq.git; cd MAPseq && ./bootstrap && ./configure && make && make install; mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- MAPseq attains maximum F½ ≈ 0.86 for genus-level mapping at 500 bp, up to 30% better than existing solutions
- MAPseq attains maximum F½ ≈ 0.96 for hOTUs clustered at 98% sequence identity
- fhalf_scores.csv reproduces the relative ranking of Fig. 1c-f with MAPseq first among tools with few exceptions
- MSEQ output lines contain alignment fields (query id, reference id, alignment score, pairwise identity, matches/mismatches/gaps) followed by per-level taxonomy labels and confidences
- Combined confidence cutoff of 0.4 or 0.5 applied, as this value yielded the highest F½-score in the authors' benchmarks

## Limitations

- MAPseq's default reference databases and data files must stay co-located with the binary; moving only the binary elsewhere breaks the installation
- Confidence estimates depend on preoptimized identity cutoffs at each taxonomic level, so custom references require correctly formatted taxonomy headers with '#cutoff' parameters
- Build from source requires multiple system dependencies (svn, wget, git, libncurses5-dev, libtool, autotools)

## Evidence

- [abstract] MAPseq is up to 30% more accurate (F½ score) than existing solutions: "up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [abstract] MAPseq provides taxonomy and OTU mappings in a single run: "providing in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences in both amplicon and shotgun sequencing strategies"
- [readme] README installation steps for source build: "./setup.sh      # downloads eutils and the reference data files
./bootstrap     # this step is only needed if you cloned the repository, you will also need to install autotools/autoconf"
- [readme] README recommended confidence cutoff: [evidence span withheld — anchor status: external_doc]
- [readme] README basic classification invocation: [evidence span withheld — anchor status: external_doc]
