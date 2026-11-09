---
name: blast-output-parsing
description: Use when a profiler's taxonomic assignment looks suspicious (e.g. 16S
  reads of a known mock community member mapped to an unexpected genus) and you need
  to verify, at the sequence level, whether a reference library entry is contaminated.
when_to_use_negative:
- The goal is profiling reads against the reference library itself (use PathoScope
  2.0 or Kraken 2), not auditing individual library entries for contamination
- Input is a 16S OTU/ASV feature table without any suspect sequence or reference entry
  to align
- The reference library has already been processed to remove contamination (the paper
  recommends using older, cleaned RefSeq libraries rather than re-auditing newer ones
  entry-by-entry)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: BLAST
  role: Pairwise blastn alignment of the contaminant query sequence against the suspect
    RefSeq scaffold, producing the tabular report that is parsed
- name: MetaScope
  role: Its download_refseq() function provides up-to-date RefSeq indices whose entries
    may need this contamination check
  repo: https://compbiomed.github.io/metascope-docs/index.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:abe6a7015fcee9bc80c05ab8dc9770a794cbe173de3473d1bb4f5a62f9e53c3a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - MetaScope
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# blast-output-parsing

## Summary

Runs and parses pairwise BLAST alignments to quantify sequence-level contamination in reference databases, e.g. detecting an E. coli 16S rRNA contaminant embedded in a RefSeq T. flagellatus scaffold by extracting percent identity, alignment length, and bit score of the top HSP.

## When to use

Use when a profiler's taxonomic assignment looks suspicious (e.g. 16S reads of a known mock community member mapped to an unexpected genus) and you need to verify, at the sequence level, whether a reference library entry is contaminated.

## When NOT to use

- The goal is profiling reads against the reference library itself (use PathoScope 2.0 or Kraken 2), not auditing individual library entries for contamination
- Input is a 16S OTU/ASV feature table without any suspect sequence or reference entry to align
- The reference library has already been processed to remove contamination (the paper recommends using older, cleaned RefSeq libraries rather than re-auditing newer ones entry-by-entry)

## Inputs

- FASTA file of the query sequence suspected as contaminant (e.g. E. coli 16S rRNA gene)
- FASTA file of the RefSeq scaffold/entry under investigation (e.g. T. flagellatus NZ_JMIR01000093)
- Taxonomic assignment table from a profiler (e.g. PathoScope 2.0 or Kraken 2 output) flagging the suspicious taxon

## Outputs

- Tabular BLAST alignment report (outfmt 6: qseqid, sseqid, pident, length, bitscore)
- Parsed top-HSP metrics: percent identity (e.g. 100%), alignment length (e.g. 911 bp), bit score
- Contamination verdict for the reference library entry, suitable for filtering/correcting the library

## How to apply

Retrieve the exact FASTA records for the suspicious scaffold (e.g. T. flagellatus scaffold NZ_JMIR01000093 from RefSeq) and the query sequence suspected as the contaminant (e.g. the E. coli 16S rRNA gene). Run a pairwise blastn alignment with NCBI BLAST+ using the query against the scaffold as subject, requesting tabular output so percent identity, alignment length, and bit score can be parsed programmatically. Parse the output to extract the top high-scoring segment pair and judge contamination by identity and coverage: in the paper's case, the top hit exhibited 100% identity over 911 bp, confirming an E. coli 16S rRNA contaminant embedded in the T. flagellatus scaffold that accounted for PathoScope's Tumebacillus misassignments (Pearson r = 0.959 with expected E. coli abundance). The rationale is that contamination in public repositories (over 2,000,000 contaminated GenBank entries cited) propagates directly into profiler reference libraries like RefSeq 2020, so sequence-level verification is needed before trusting taxonomic calls.

## Related tools

- **BLAST** (Pairwise blastn alignment of the contaminant query sequence against the suspect RefSeq scaffold, producing the tabular report that is parsed)
- **MetaScope** (Its download_refseq() function provides up-to-date RefSeq indices whose entries may need this contamination check) — https://compbiomed.github.io/metascope-docs/index.html

## Examples

```
blastn -query ecoli_16s_rrna.fasta -subject NZ_JMIR01000093.fasta -outfmt '6 qseqid sseqid pident length bitscore' -out hits.tsv
```

## Evaluation signals

- Top HSP from the parsed BLAST report reaches 100% percent identity over ~911 bp for a true 16S rRNA contaminant (as in the E. coli vs T. flagellatus case)
- Percent identity, alignment length, and bit score fields are all non-empty and numerically parseable from the outfmt-6 report
- The contaminant signal aligns with independent evidence, e.g. PathoScope's Tumebacillus misassignments correlating with expected E. coli abundance (Pearson r = 0.959)
- The suspect scaffold accession matches the exact RefSeq entry used by the profiler's library (e.g. NZ_JMIR01000093), not an off-target record

## Limitations

- Contamination is an ongoing problem in public repositories: the paper notes this 'demands additional filtering and correction efforts after directly retrieving libraries from the public repository'
- A single pairwise BLAST only validates one query–subject pair; scaling to full-library QC would require read-mapping approaches such as Kraken 2 or PathoScope
- The check is specific to the suspect taxon; unflagged contaminated entries elsewhere in the library will go undetected

## Evidence

- [discussion] Pairwise BLAST of E. coli 16S rRNA gene against T. flagellates scaffolds to identify database contamination: [evidence span withheld — anchor status: not_found]
- [intro] RefSeq library recommendation motivating contamination checks: "The current PathoScope reference library recommendation is to download the complete RefSeq representative genome database"
- [readme] Obtaining up-to-date RefSeq indices for QC: "Up-to-date RefSeq indices may be obtained by using the `download_refseq()` function from the [MetaScope package]"
