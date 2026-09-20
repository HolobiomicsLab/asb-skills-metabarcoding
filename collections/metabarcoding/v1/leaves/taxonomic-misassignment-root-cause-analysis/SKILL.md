---
name: taxonomic-misassignment-root-cause-analysis
description: Use when a metagenomic/16S profiler reports high-confidence assignments
  to an unexpected taxon whose abundance correlates with a known community member
  (here Pearson r = 0.959), suggesting the reference library—not the reads—carries
  a contaminant.
when_to_use_negative:
- The profiler's outputs already agree with the mock community's expected composition
  — there is no misassignment to root-cause.
- You are benchmarking profiler/reference-library accuracy on clean libraries rather
  than diagnosing a specific anomalous taxonomic call.
- The unexpected taxon call lacks a plausible donor organism in the sample whose abundance
  could correlate with it (no contamination hypothesis to test).
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: BLAST
  role: Pairwise blastn alignment of the E. coli 16S rRNA query against the T. flagellatus
    scaffold to identify the embedded contaminant
- name: PathoScope 2
  role: Whole-genome metagenomic profiler whose Tumebacillus misassignments (correlating
    r = 0.959 with expected E. coli abundance) trigger and validate the root-cause
    analysis
- name: MetaScope
  role: R/Bioconductor package providing download_refseq() for retrieving and rebuilding
    custom PathoScope RefSeq libraries to replace contaminated ones
  repo: https://compbiomed.github.io/metascope-docs/index.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:8cddeecc9d34aaf18c3b42d4478678ffd7f02226d1ad18db17fefb30f65d98fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - MetaScope
  - PathoScope 2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# taxonomic-misassignment-root-cause-analysis

## Summary

A diagnostic procedure for tracing unexpected taxonomic calls (e.g., PathoScope's Tumebacillus flagellatus misassignments in 16S mock-community data) to contamination embedded in the reference database itself, via pairwise BLAST of a suspect taxon's marker gene against the exact RefSeq scaffold.

## When to use

Use when a metagenomic/16S profiler reports high-confidence assignments to an unexpected taxon whose abundance correlates with a known community member (here Pearson r = 0.959), suggesting the reference library—not the reads—carries a contaminant.

## When NOT to use

- The profiler's outputs already agree with the mock community's expected composition — there is no misassignment to root-cause.
- You are benchmarking profiler/reference-library accuracy on clean libraries rather than diagnosing a specific anomalous taxonomic call.
- The unexpected taxon call lacks a plausible donor organism in the sample whose abundance could correlate with it (no contamination hypothesis to test).

## Inputs

- RefSeq scaffold FASTA for the misassigned taxon (e.g., T. flagellatus NZ_JMIR01000093)
- FASTA of the presumed contaminant's 16S rRNA gene sequence (e.g., E. coli)
- Per-sample taxonomic abundance table from the profiler (e.g., PathoScope 2.0 outputs on mock community samples)

## Outputs

- BLAST tabular/text alignment report (percent identity, alignment length, bit score of the top HSP)
- Confirmed contamination call (e.g., 100% identity over 911 bp) documenting the reference-database defect
- Correlation statistic between misassigned-taxon and expected-contaminant abundances (Pearson r)

## How to apply

Retrieve the exact RefSeq entries in FASTA format: the suspect scaffold (T. flagellatus NZ_JMIR01000093) and the presumed contaminant's 16S rRNA gene sequence (E. coli). Run a pairwise blastn (NCBI BLAST+) with the E. coli 16S rRNA as query and the T. flagellatus scaffold as subject, outputting a tabular alignment report. Parse the top high-scoring segment pair for percent identity, alignment length, and bit score; a 100% identity hit over 911 bp confirms an E. coli 16S rRNA contaminant embedded in the scaffold. Cross-check that the misassigned taxon's abundance tracks the expected abundance of the contaminating organism (Pearson r = 0.959 for E. coli vs. Tumebacillus calls) to close the causal loop. This is warranted because large-scale searches have identified more than 2,000,000 contaminated entries in GenBank underlying such reference libraries.

## Related tools

- **BLAST** (Pairwise blastn alignment of the E. coli 16S rRNA query against the T. flagellatus scaffold to identify the embedded contaminant)
- **PathoScope 2** (Whole-genome metagenomic profiler whose Tumebacillus misassignments (correlating r = 0.959 with expected E. coli abundance) trigger and validate the root-cause analysis)
- **MetaScope** (R/Bioconductor package providing download_refseq() for retrieving and rebuilding custom PathoScope RefSeq libraries to replace contaminated ones) — https://compbiomed.github.io/metascope-docs/index.html

## Examples

```
blastn -query ecoli_16s_rrna.fa -subject NZ_JMIR01000093.fa -outfmt '6 qseqid sseqid pident length bitscore' -out contamination_check.tsv
```

## Evaluation signals

- Top BLAST HSP between the E. coli 16S rRNA query and the T. flagellatus scaffold NZ_JMIR01000093 shows 100% percent identity over 911 bp.
- Pearson correlation between PathoScope's Tumebacillus calls and expected E. coli abundance is r = 0.959, confirming the causal link.
- The BLAST report was generated against the exact RefSeq entry (not a proxy database version).
- Identical library lineage is documented (the README notes the Kraken Standard library 'is comparable to the PathoScope RefSeq 2020 library used in the paper'), so findings transfer across comparable index builds.

## Limitations

- Contamination in public reference databases is widespread and ongoing: more than 2,000,000 contaminated entries were identified in GenBank, so individual BLAST checks do not scale to full-library decontamination.
- Recommended practice is to use older RefSeq libraries until newer versions have been processed to remove contamination.
- Additional filtering and correction efforts are required after directly retrieving libraries from the public repository; contamination may persist in freshly downloaded indices.

## Evidence

- [discussion] a pairwise BLAST comparing E. coli’s 16S rRNA gene sequence to the T. flagellates scaffolds using the exact RefSeq entry: "a pairwise BLAST comparing E. coli’s 16S rRNA gene sequence to the T. flagellates scaffolds using the exact RefSeq entry"
- [discussion] this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository: "this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository"
- [readme] Note that this library is comparable to the PathoScope RefSeq 2020 library used in the paper.: "Note that this library is comparable to the PathoScope RefSeq 2020 library used in the paper."
- [discussion] metagenomic read-mapping approaches such as Kraken 2 and PathoScope afford the potential for the development of novel quality control pipelines for RefSeq: "metagenomic read-mapping approaches such as Kraken 2 and PathoScope afford the potential for the development of novel quality control pipelines for RefSeq"
