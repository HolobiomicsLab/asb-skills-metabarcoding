---
name: pairwise-blast-alignment
description: Use when a taxonomic profiler (e.g., PathoScope) assigns reads from a
  mock community to an unexpected organism in a known-composition sample, and you
  suspect an embedded contaminant in the underlying public reference (GenBank/RefSeq)
  entry rather than a biological hit.
when_to_use_negative:
- Input is raw amplicon reads needing denoising/ASV inference first (use DADA2, QIIME
  2, or Mothur rather than a scaffold-vs-gene BLAST).
- The goal is broad library-wide contamination screening rather than validating a
  single suspicious taxonomic assignment; the article frames this as a targeted pairwise
  check of one exact RefSeq entry.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: BLAST
  role: NCBI BLAST+ blastn used for the pairwise alignment of the E. coli 16S rRNA
    query against the T. flagellatus scaffold to detect the embedded contaminant
- name: PathoScope 2.0
  role: Profiler whose reads-to-reference alignment against the contaminated RefSeq
    2020 library produced the Tumebacillus misassignments being diagnosed
  repo: https://github.com/aubreyodom/16SBenchmarking
- name: MetaScope
  role: R package whose download_refseq() function retrieves up-to-date RefSeq indices,
    the library layer the contamination check targets
  repo: https://compbiomed.github.io/metascope-docs/index.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:d10420ca8adacfb633e05b687900f214f223a4c0b15c8152a3e3c95569fd6917
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aubreyodom/16SBenchmarking
  tools:
  - BLAST
  - MetaScope
  - PathoScope 2.0
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# pairwise-blast-alignment

## Summary

Runs a targeted pairwise BLAST (NCBI BLAST+) alignment between a known contaminant sequence (e.g., E. coli 16S rRNA gene) and a suspect reference-library scaffold to prove database contamination that drives taxonomic misassignment, as done to confirm the E. coli contaminant embedded in the Tumebacillus flagellatus RefSeq scaffold NZ_JMIR01000093.

## When to use

Use when a taxonomic profiler (e.g., PathoScope) assigns reads from a mock community to an unexpected organism in a known-composition sample, and you suspect an embedded contaminant in the underlying public reference (GenBank/RefSeq) entry rather than a biological hit.

## When NOT to use

- Input is raw amplicon reads needing denoising/ASV inference first (use DADA2, QIIME 2, or Mothur rather than a scaffold-vs-gene BLAST).
- The goal is broad library-wide contamination screening rather than validating a single suspicious taxonomic assignment; the article frames this as a targeted pairwise check of one exact RefSeq entry.

## Inputs

- FASTA file of the suspect RefSeq scaffold (e.g., NZ_JMIR01000093 from T. flagellatus)
- FASTA file of the query 16S rRNA gene sequence (e.g., E. coli 16S rRNA gene)
- Profiler taxonomic assignment table (e.g., PathoScope output) linking misassigned taxa to sample abundances

## Outputs

- Tabular BLAST report (percent identity, alignment length, bit score of the top HSP)
- Contamination verdict: e.g., 100% identity over 911 bp confirming an E. coli 16S rRNA contaminant in the scaffold
- Correlation of contaminant-linked misassignments with expected source-taxon abundance (Pearson r = 0.959)

## How to apply

Retrieve the exact FASTA for the suspect RefSeq scaffold (T. flagellatus NZ_JMIR01000093) and the query 16S rRNA gene sequence (E. coli) from the same RefSeq release underlying the profiler's library (RefSeq 2020 here). Run a pairwise blastn alignment (NCBI BLAST+) with the 16S gene as query and the scaffold as subject, outputting a tabular/text report. Parse the top high-scoring segment pair for percent identity, alignment length, and bit score; in this case 100% identity over 911 bp confirmed an E. coli 16S rRNA contaminant embedded in the scaffold, explaining PathoScope's Tumebacillus misassignments (Pearson r = 0.959 with expected E. coli abundance). Conclude by tracing the hit's taxonomic hierarchy in the profiler (PathoScope, QIIME 2, and Mothur all permit this) to confirm the false assignment derives from the contaminated entry.

## Related tools

- **BLAST** (NCBI BLAST+ blastn used for the pairwise alignment of the E. coli 16S rRNA query against the T. flagellatus scaffold to detect the embedded contaminant)
- **PathoScope 2.0** (Profiler whose reads-to-reference alignment against the contaminated RefSeq 2020 library produced the Tumebacillus misassignments being diagnosed) — https://github.com/aubreyodom/16SBenchmarking
- **MetaScope** (R package whose download_refseq() function retrieves up-to-date RefSeq indices, the library layer the contamination check targets) — https://compbiomed.github.io/metascope-docs/index.html

## Examples

```
blastn -query ecoli_16s_rrna.fasta -subject NZ_JMIR01000093.fasta -outfmt "6 qseqid sseqid pident length bitscore" -out contamination_check.tsv
```

## Evaluation signals

- Top HSP of the blastn report shows 100% percent identity over 911 bp between the E. coli 16S rRNA query and the T. flagellatus scaffold.
- The BLAST alignment length and bit score are reported for the top HSP extracted from the parsed tabular output.
- Correlation between Tumebacillus misassignment counts and expected E. coli relative abundance yields Pearson r = 0.959.
- The exact RefSeq entry used for the BLAST matches the version underlying the profiler library (RefSeq 2020, comparable to the Kraken Standard index dated 9/19/2020).
- Taxonomic hierarchy tracing in PathoScope/QIIME 2/Mothur connects the contaminant-containing scaffold to the erroneous genus-level call.

## Limitations

- Contamination is an ongoing problem in public repositories: more than 2,000,000 contaminated entries have been identified in GenBank, so a single pairwise BLAST fixes one entry, not the library.
- The article recommends using older RefSeq libraries until newer versions have been processed to remove contamination; additional filtering and correction is required after directly retrieving libraries from the public repository.
- Metagenomic read-mapping approaches such as Kraken 2 and PathoScope would need to be developed into novel quality-control pipelines for RefSeq for systematic detection rather than one-off BLAST checks.

## Evidence

- [discussion] a pairwise BLAST comparing E. coli’s 16S rRNA gene sequence to the T. flagellates scaffolds using the exact RefSeq entry: "a pairwise BLAST comparing E. coli’s 16S rRNA gene sequence to the T. flagellates scaffolds using the exact RefSeq entry"
- [discussion] this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository: "this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository"
- [readme] The RefSeq 2020 indices can be downloaded [here.](https://rutgers.box.com/s/b3fxv2n7ol4fcwmbe6ihmcq0eyb978lo): "The RefSeq 2020 indices can be downloaded [here.](https://rutgers.box.com/s/b3fxv2n7ol4fcwmbe6ihmcq0eyb978lo)"
- [discussion] metagenomic read-mapping approaches such as Kraken 2 and PathoScope afford the potential for the development of novel quality control pipelines for RefSeq: "metagenomic read-mapping approaches such as Kraken 2 and PathoScope afford the potential for the development of novel quality control pipelines for RefSeq"
