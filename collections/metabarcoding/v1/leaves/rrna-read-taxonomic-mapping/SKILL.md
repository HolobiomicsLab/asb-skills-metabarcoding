---
name: rrna-read-taxonomic-mapping
description: Use when demultiplexed, quality-filtered rRNA FASTA/FASTQ reads must
  be classified against a full-length rRNA reference (NCBI taxonomy and/or custom
  OTU taxonomies) with per-level confidence estimates in a single run.
when_to_use_negative:
- Input is already a .mseq output or a count/feature table — use the -otucounts/-otutable
  summarization modes instead of re-running classification
- You need to classify non-rRNA sequence data or require a protein/domain classifier;
  MAPseq is scoped to ribosomal RNA reference-based analysis
- The reference database has been heavily de-redundanted by removing near-identical
  sequences — this measurably degrades F½ mapping accuracy
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MAPseq
  role: 'Core k-mer search classifier: assigns taxonomy and hierarchical OTU labels
    with confidence estimates to rRNA reads in a single run; also provides -otucounts
    and -otutable summarization modes'
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: Generates the 'configure' script used to build MAPseq v1.2.6 from source ('./configure
    && make && make install')
- name: GNU make
  role: Compiles and installs MAPseq from the configured source tree
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:1f74269b477ada5d5d60c575c3ffa3836e48c56eb71b7ae7bf518e63f74679e0
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# rrna-read-taxonomic-mapping

## Summary

Assign taxonomy and hierarchical OTU labels to rRNA reads (amplicon or shotgun) using MAPseq's k-mer search with confidence estimates against a full-length rRNA reference database. MAPseq is reported up to 30% more accurate (F½ score) and up to 100x faster than competing read classifiers.

## When to use

Use when demultiplexed, quality-filtered rRNA FASTA/FASTQ reads must be classified against a full-length rRNA reference (NCBI taxonomy and/or custom OTU taxonomies) with per-level confidence estimates in a single run.

## When NOT to use

- Input is already a .mseq output or a count/feature table — use the -otucounts/-otutable summarization modes instead of re-running classification
- You need to classify non-rRNA sequence data or require a protein/domain classifier; MAPseq is scoped to ribosomal RNA reference-based analysis
- The reference database has been heavily de-redundanted by removing near-identical sequences — this measurably degrades F½ mapping accuracy

## Inputs

- FASTA file of demultiplexed, quality-filtered rRNA reads (amplicon or shotgun), e.g. rawseqs.fa
- Default bundled full-length rRNA reference database (mapref, ~1.5 million sequences) or a custom reference FASTA plus one or more tab-separated taxonomy files with a #cutoff header

## Outputs

- .mseq classification file (simple or confidences format) with alignment fields and taxonomy/confidence values at each taxonomic and OTU level
- Taxonomy/OTU count summary per sample (`mapseq -otucounts <sample1.mseq>`)
- Multi-sample OTU/taxa count table as TSV (`mapseq -otutable <sample1.mseq> <sample2.mseq> ...`) importable into R

## How to apply

Run `mapseq rawseqs.fa > rawseqs.fa.mseq` to classify reads against the bundled default reference (NCBI taxonomy plus OTU taxonomies), or supply a custom nucleotide reference FASTA with one or more taxonomy files where each file starts with a `#cutoff:` header (e.g. '#cutoff: 0.00:0.08 0.70:0.35 ...') and gives tab-separated accession-to-taxonomy lines. Adjust thread count with `-nthreads`. Filter assignments using the combined confidence in the 'confidences' output format; the authors recommend a combined confidence cutoff of 0.4 or 0.5, which yielded the highest F1/2-score in their benchmarks. Summarize a single .mseq file with `mapseq -otucounts` or build a multi-sample OTU/taxa count table with `mapseq -otutable` (select taxonomy with -ti and level with -tl), then import the TSV into R via `read.table(..., sep="\t", header=TRUE)`. Avoid removing near-identical sequences from the reference to reduce redundancy — the source-card benchmarks show a significant accuracy trade-off (F½ degradation, Fig. 1g-h) for MAPseq and all competing tools.

## Related tools

- **MAPseq** (Core k-mer search classifier: assigns taxonomy and hierarchical OTU labels with confidence estimates to rRNA reads in a single run; also provides -otucounts and -otutable summarization modes) — https://github.com/jfmrod/mapseq
- **autoconf** (Generates the 'configure' script used to build MAPseq v1.2.6 from source ('./configure && make && make install'))
- **GNU make** (Compiles and installs MAPseq from the configured source tree)

## Examples

```
mapseq rawseqs.fa > rawseqs.fa.mseq && mapseq -otutable rawseqs.fa.mseq > map.otutable
```

## Evaluation signals

- Compute F½ classification accuracy against a known-identity read benchmark set (ground-truth assignments); MAPseq should reach up to 30% higher F½ than competing tools at the recommended confidence cutoff
- Check that the .mseq output parses with 13 alignment fields before the first empty field, followed by alternating taxonomy/confidence pairs per level
- Verify assignments retain a combined confidence of at least 0.4–0.5, the range the authors report as giving the highest F1/2-score
- Tabulate accuracy versus reference redundancy level and confirm that clustering out near-identical reference sequences degrades F½ (per Fig. 1g-h / Supplementary Fig. S2 trend)
- Confirm `mapseq -otutable` output includes counts from all supplied samples, not just the first (a bug fixed in v1.2.6)

## Limitations

- Reducing reference database redundancy (removing near-identical sequences) causes a significant trade-off cost in mapping accuracy for MAPseq and all competing methods
- Moving only the mapseq binary elsewhere breaks the installation because data files are searched relative to the binary's path
- The -ignoreEmptyTax option (v1.2.4) is off by default pending thorough benchmarks, so second hits with missing taxonomic labels can decrease confidence of the top-hit assignment

## Evidence

- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] This will classify all the sequences found in rawseqs against the standard reference dataset provided with MAPseq.: "mapseq rawseqs.fa > rawseqs.fa.mseq

This will classify all the sequences found in rawseqs against the standard reference dataset provided with MAPseq."
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
- [readme] mapseq rawseqs.fa <customref.fasta> <customref.tax> [customref.tax2 ...] > rawseqs.fa.mseq: [evidence span withheld — anchor status: external_doc]
