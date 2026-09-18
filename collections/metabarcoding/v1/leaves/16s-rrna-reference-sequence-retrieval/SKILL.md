---
name: 16s-rrna-reference-sequence-retrieval
description: Use when a 16S profiling result assigns reads to an unexpected taxon
  and you need the exact reference sequences to test whether database contamination,
  not true biology, explains the assignment.
when_to_use_negative:
- When the goal is routine library acquisition rather than verifying a specific suspect
  assignment — use MetaScope's download_refseq() instead
- When input is already a genus/species abundance feature table with no suspicious
  taxonomic calls to investigate
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0654
tools:
- name: BLAST
  role: Pairwise blastn (NCBI BLAST+) alignment of the E. coli 16S rRNA query against
    the T. flagellatus scaffold to expose the contaminant HSP
- name: MetaScope
  role: Provides the download_refseq() function for obtaining up-to-date RefSeq indices
    used in the profiling whose calls are being verified
  repo: https://compbiomed.github.io/metascope-docs/index.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:86c7c95987bcc36ce302633ce093dfa3289280ff4a3338120916dbde58fc1e64
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - MetaScope
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 4
---

# 16s-rrna-reference-sequence-retrieval

## Summary

Retrieves the exact RefSeq FASTA entries needed to verify whether a suspicious taxonomic call (e.g., Tumebacillus flagellatus in RefSeq 2020) is explained by an E. coli 16S rRNA contaminant embedded in the reference scaffold, using a pairwise BLAST alignment as the diagnostic.

## When to use

Use when a 16S profiling result assigns reads to an unexpected taxon and you need the exact reference sequences to test whether database contamination, not true biology, explains the assignment.

## When NOT to use

- When the goal is routine library acquisition rather than verifying a specific suspect assignment — use MetaScope's download_refseq() instead
- When input is already a genus/species abundance feature table with no suspicious taxonomic calls to investigate

## Inputs

- FASTA of the suspect RefSeq scaffold (e.g., T. flagellatus NZ_JMIR01000093)
- FASTA of the putative contaminant 16S rRNA gene (E. coli 16S rRNA gene sequence from RefSeq)
- RefSeq 2020 reference library indices used by the profiler (PathoScope RefSeq 2020 / Kraken 2 Standard library, 8/20/2020 download)

## Outputs

- Pairwise blastn tabular/text alignment report with percent identity, alignment length, and bit score of the top HSP
- Confirmation (or rejection) that a 16S rRNA contaminant is embedded in the reference scaffold (expected: 100% identity over 911 bp)

## How to apply

Retrieve the exact RefSeq entries for the suspect scaffold (T. flagellatus scaffold NZ_JMIR01000093) and the putative contaminant (E. coli 16S rRNA gene sequence) in FASTA format. Run a pairwise blastn (NCBI BLAST+) alignment of the E. coli 16S rRNA query against the T. flagellatus scaffold as subject, emitting a tabular/text report. Parse the output to extract the top high-scoring segment pair's percent identity, alignment length, and bit score. The check passes if the top hit shows 100% identity over 911 bp, confirming an E. coli 16S rRNA contaminant embedded in the scaffold — which accounts for PathoScope's Tumebacillus misassignments correlating with expected E. coli abundance (Pearson r = 0.959). Rationale: contamination in public reference libraries can compromise taxonomic assignments, so verification against the exact reference entry is required before trusting a call.

## Related tools

- **BLAST** (Pairwise blastn (NCBI BLAST+) alignment of the E. coli 16S rRNA query against the T. flagellatus scaffold to expose the contaminant HSP)
- **MetaScope** (Provides the download_refseq() function for obtaining up-to-date RefSeq indices used in the profiling whose calls are being verified) — https://compbiomed.github.io/metascope-docs/index.html

## Examples

```
blastn -query ecoli_16s_rrna.fasta -subject NZ_JMIR01000093.fasta -outfmt '6 qseqid sseqid pident length bitscore'
```

## Evaluation signals

- Top HSP exhibits 100% percent identity over 911 bp between the E. coli 16S rRNA query and the T. flagellatus scaffold NZ_JMIR01000093
- BLAST tabular output contains the three parsed metrics (percent identity, alignment length, bit score) for the top HSP
- PathoScope Tumebacillus misassignment counts correlate with expected E. coli abundance at Pearson r = 0.959, consistent with the identified contaminant
- The retrieved sequences match the exact RefSeq entries (accession NZ_JMIR01000093 for the scaffold)

## Limitations

- Contamination in public sequence databases is widespread — a large-scale search identified more than 2,000,000 contaminated entries in GenBank — so a single scaffold check does not certify a whole library
- The article recommends using older RefSeq libraries until newer versions have been processed to remove contamination
- Additional filtering and correction of contamination is required after directly retrieving libraries from the public repository

## Evidence

- [discussion] a pairwise BLAST comparing E. coli’s 16S rRNA gene sequence to the T. flagellates scaffolds using the exact RefSeq entry: "a pairwise BLAST comparing E. coli’s 16S rRNA gene sequence to the T. flagellates scaffolds using the exact RefSeq entry"
- [discussion] this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository: "this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository"
- [other] Retrieve the exact RefSeq entries for the T. flagellatus scaffold NZ_JMIR01000093 and the E. coli 16S rRNA gene sequence in FASTA format.: [evidence span withheld — anchor status: not_found]
- [other] Confirm the top hit exhibits 100% identity over 911 bp, consistent with an E. coli 16S rRNA contaminant embedded in the T. flagellatus scaffold: [evidence span withheld — anchor status: not_found]
- [readme] Up-to-date RefSeq indices may be obtained by using the download_refseq() function from the [MetaScope package](https://compbiomed.github.io/metascope-docs/index.html).: "Up-to-date RefSeq indices may be obtained by using the `download_refseq()` function from the [MetaScope package]"
