---
name: refseq-scaffold-contamination-verification
description: Use when a profiling tool assigns 16S amplicon reads to an unexpected
  organism (e.g., Tumebacillus flagellatus calls correlating with E. coli abundance),
  suggesting the reference genome/library entry itself contains contaminant sequence.
when_to_use_negative:
- The goal is general 16S profiler accuracy benchmarking on mock communities rather
  than validating a specific reference entry's integrity
- The reference library has already been screened/corrected for contamination (e.g.,
  a cleaned RefSeq version) and no anomalous taxonomic calls are present
- Input is amplicon read data needing taxonomic assignment, not a reference scaffold
  needing contamination verification
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: BLAST (NCBI BLAST+)
  role: Pairwise blastn alignment of the E. coli 16S rRNA query against the T. flagellatus
    scaffold to detect the embedded contaminant
- name: MetaScope (PathoScope-based)
  role: Source of the download_refseq() function for obtaining up-to-date RefSeq indices,
    and the PathoScope pipeline whose misassignments motivated the contamination check
  repo: https://compbiomed.github.io/metascope-docs/index.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:02e301c53549418fecc7fddf763fe7edbd4da9f30b43a20c4787d4efbb4f7412
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST (NCBI BLAST+)
  - MetaScope (PathoScope-based)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# refseq-scaffold-contamination-verification

## Summary

Verifies contamination of public reference database entries (RefSeq/GenBank) by running a pairwise BLAST of a suspect taxon's marker sequence (e.g., E. coli 16S rRNA gene) against the exact scaffold from another organism's RefSeq entry (e.g., T. flagellatus NZ_JMIR01000093), to explain taxonomic misassignments in 16S profiling.

## When to use

Use when a profiling tool assigns 16S amplicon reads to an unexpected organism (e.g., Tumebacillus flagellatus calls correlating with E. coli abundance), suggesting the reference genome/library entry itself contains contaminant sequence.

## When NOT to use

- The goal is general 16S profiler accuracy benchmarking on mock communities rather than validating a specific reference entry's integrity
- The reference library has already been screened/corrected for contamination (e.g., a cleaned RefSeq version) and no anomalous taxonomic calls are present
- Input is amplicon read data needing taxonomic assignment, not a reference scaffold needing contamination verification

## Inputs

- FASTA sequence of the suspect RefSeq scaffold (e.g., NZ_JMIR01000093)
- FASTA sequence of the putative contaminant marker gene (e.g., E. coli 16S rRNA gene)
- Profiler taxonomic assignment table (e.g., PathoScope genus-level calls) for corroboration

## Outputs

- Tabular/text BLAST alignment report with percent identity, alignment length, and bit score of the top HSP
- Contamination verdict for the reference scaffold (contaminant identity and embedded marker length, e.g., 100% identity over 911 bp)

## How to apply

Retrieve the exact RefSeq FASTA entries for the suspect scaffold (e.g., T. flagellatus NZ_JMIR01000093) and the putative contaminant marker gene (e.g., E. coli 16S rRNA). Run a pairwise blastn (NCBI BLAST+) with the marker gene as query and the scaffold as subject, outputting a tabular alignment report. Parse the top high-scoring segment pair for percent identity, alignment length, and bit score; the diagnostic signature in the paper is a 100% identity hit over 911 bp, indicating an embedded E. coli 16S rRNA contaminant that accounts for PathoScope's Tumebacillus misassignments (Pearson r = 0.959 with expected E. coli abundance). Rationale: the paper found >2,000,000 contaminated GenBank entries, so libraries retrieved from public repositories require this additional filtering/correction before trusted taxonomic assignment.

## Related tools

- **BLAST (NCBI BLAST+)** (Pairwise blastn alignment of the E. coli 16S rRNA query against the T. flagellatus scaffold to detect the embedded contaminant)
- **MetaScope (PathoScope-based)** (Source of the download_refseq() function for obtaining up-to-date RefSeq indices, and the PathoScope pipeline whose misassignments motivated the contamination check) — https://compbiomed.github.io/metascope-docs/index.html

## Examples

```
blastn -query ecoli_16s_rrna.fasta -subject NZ_JMIR01000093.fasta -outfmt 6 -out contamination_check.tsv
```

## Evaluation signals

- Top HSP of the blastn alignment shows 100% percent identity, consistent with the expected 911 bp E. coli 16S rRNA contaminant in the scaffold
- Bit score and alignment length of the top hit are extracted and reported from the parsed tabular BLAST output
- Scaffold-specific misassignment rate (e.g., Tumebacillus calls) correlates with the contaminant organism's expected abundance (paper: Pearson r = 0.959)
- The exact RefSeq accession (NZ_JMIR01000093) is retrieved and used as the subject, matching the entry that produced the misassignment

## Limitations

- This is an ongoing problem: contaminated entries in GenBank/RefSeq demand additional filtering and correction after directly retrieving libraries from the public repository
- The paper recommends using older RefSeq libraries until newer versions have been processed to remove contamination
- Contamination identified in a marker-gene region invalidates taxonomic calls for that entry but requires per-scaffold verification; read-mapping QC (Kraken 2, PathoScope) is proposed as a future scalable alternative

## Evidence

- [discussion] Pairwise BLAST of E. coli 16S rRNA gene against T. flagellates scaffolds to identify database contamination: "a pairwise BLAST comparing E. coli’s 16S rRNA gene sequence to the T. flagellates scaffolds using the exact RefSeq entry"
- [discussion] Contamination requires post-retrieval filtering of public libraries: "this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository"
- [discussion] Metagenomic read-mappers as future RefSeq QC: "metagenomic read-mapping approaches such as Kraken 2 and PathoScope afford the potential for the development of novel quality control pipelines for RefSeq"
- [readme] RefSeq indices provenance in the paper's repository: "Up-to-date RefSeq indices may be obtained by using the `download_refseq()` function from the [MetaScope package]"
