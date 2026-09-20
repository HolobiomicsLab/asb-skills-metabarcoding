---
name: pacbio-ccs-read-alignment
description: Use when you have PacBio CCS reads of full-length (~1500 bp) 16S amplicons
  plus reference 16S sequences, and you need per-read error or per-position substitution
  profiles (e.g., CCS pass-number error curves, homopolymer-deletion mapping, or 16S
  SNP detection).
when_to_use_negative:
- The input is short-read sequence data from a 16S variable region (e.g., V4) rather
  than full-length PacBio CCS reads — the paper shows variable regions cannot match
  full-gene taxonomic accuracy
- The goal is to resolve insertions/deletions between intragenomic 16S copies — PacBio
  CCS indel errors are systematic, so the paper's SNP-calling approach ignores indels
  entirely
- An isolate has fewer than 200 aligned reads to its most-abundant sequence — the
  paper's coverage filter excludes such isolates from SNP calling
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Cross_match
  role: Pairwise aligns PacBio CCS reads to 16S reference sequences using minscore
    750, penalty -9, masklevel 0; its alignments are parsed to tabulate substitution/indel
    errors and per-position substitution frequencies
- name: PacBio
  role: Sequencing platform producing the CCS full-length 16S reads and circular-consensus
    pass numbers analyzed by this skill
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the paper's per-figure analysis
    code in the weinstock_full_length_16s repository
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:97a086dce76720ce40fb608996bfebd5cf1ff2294514fae4bdda4e510b67f9fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cross_match
  - PacBio
  - ruffus
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# pacbio-ccs-read-alignment

## Summary

Align PacBio CCS full-length 16S amplicon reads to reference 16S gene sequences (mock community references or the most-abundant per-isolate sequence) with Cross_match using the paper's parameters (minscore 750, penalty -9, masklevel 0), then parse the alignments to quantify substitutions, insertions, and deletions for error profiling or intragenomic SNP calling.

## When to use

Use when you have PacBio CCS reads of full-length (~1500 bp) 16S amplicons plus reference 16S sequences, and you need per-read error or per-position substitution profiles (e.g., CCS pass-number error curves, homopolymer-deletion mapping, or 16S SNP detection).

## When NOT to use

- The input is short-read sequence data from a 16S variable region (e.g., V4) rather than full-length PacBio CCS reads — the paper shows variable regions cannot match full-gene taxonomic accuracy
- The goal is to resolve insertions/deletions between intragenomic 16S copies — PacBio CCS indel errors are systematic, so the paper's SNP-calling approach ignores indels entirely
- An isolate has fewer than 200 aligned reads to its most-abundant sequence — the paper's coverage filter excludes such isolates from SNP calling

## Inputs

- PacBio CCS full-length 16S amplicon read set (FASTA/FASTQ) from the mock community or 381 gut isolates
- Mock reference 16S gene sequences, or the most-abundant unique full-length 16S sequence per isolate
- Read pass-number (circular-consensus pass count) metadata for grouping error rates

## Outputs

- Per-read substitution, insertion, and deletion error counts grouped by CCS pass number, with a combined error-frequency vs. pass-number curve (Supplementary Fig. 5)
- Deletion positions mapped to the 16S reference with homopolymer-run (>=4 identical consecutive bases) overlap statistics (Supplementary Figs. 6-7)
- Per-isolate SNP calls with substitution frequencies, IQR-filtered positions, and summary counts (OTUs at 99% identity, isolates with >=1 SNP, unique SNP profiles, e.g., Fig. 4a)

## How to apply

Download the PacBio CCS read set and reference 16S gene sequences from github.com/TheJacksonLaboratory/weinstock_full_length_16s, and run Cross_match with minscore 750, penalty -9, and masklevel 0 to align CCS reads to references. For error profiling, parse the Cross_match alignments to tabulate substitution, insertion, and deletion errors per read, group reads by circular-consensus pass number, and compute the combined error frequency per pass — the paper's curve reaches a minimum below 1.0% at ten passes (Supplementary Fig. 5); additionally map deletions to reference positions and test overlap with homopolymer runs (>=4 identical consecutive bases). For intragenomic SNP detection, align each isolate's reads to its most-abundant full-length 16S sequence and retain only isolates with at least 200 aligned reads, then compute per-position substitution frequencies, deliberately ignoring insertions/deletions because PacBio CCS indel errors are systematic. Flag SNP positions whose substitution frequency is a 9-IQR outlier as sequencing-error artifacts, discard OTUs clustered at 99% identity whose SNP positions show >3% impurity across member isolates, and expect ~349/381 isolates (54/61 OTUs) with >=1 SNP and 205 unique SNP profiles.

## Related tools

- **Cross_match** (Pairwise aligns PacBio CCS reads to 16S reference sequences using minscore 750, penalty -9, masklevel 0; its alignments are parsed to tabulate substitution/indel errors and per-position substitution frequencies)
- **PacBio** (Sequencing platform producing the CCS full-length 16S reads and circular-consensus pass numbers analyzed by this skill)
- **ruffus** (CGAT ruffus pipeline framework used to organize the paper's per-figure analysis code in the weinstock_full_length_16s repository) — https://github.com/cgat-developers/ruffus

## Examples

```
cross_match ccs_reads.fasta mock_16s_refs.fasta -minscore 750 -penalty -9 -masklevel 0 > ccs_vs_refs.aln
```

## Evaluation signals

- The combined substitution/indel error frequency at ten CCS passes falls below 1.0%, consistent with Supplementary Fig. 5
- Deletion errors show enrichment within homopolymer runs (>=4 identical consecutive bases) in the 16S references, reproducing Supplementary Figs. 6-7
- SNP-calling reproduces the reported summary statistics: 58 putative species, 61 OTUs at 99% identity, 349/381 isolates (54/61 OTUs) with >=1 SNP, and 205 unique SNP profiles
- Isolates with fewer than 200 aligned reads to their most-abundant sequence are excluded per the paper's coverage filter
- No SNP calls rely on insertion/deletion positions; only per-position substitution frequencies are used for polymorphism detection

## Limitations

- Systematic PacBio CCS deletion errors coincide with homopolymer runs, limiting the ability to resolve highly similar sequences; such systematic errors may occur at a fixed frequency and may not be improved by greater sequencing effort
- Ignoring insertions/deletions is an imperfect approach, demonstrated by a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes
- The study stopped short of determining how well the 99% identity threshold separates intragenomic vs. inter-genomic variation for all bacterial species
- Repository code underlying Figures 1-3 is Python 2.7 and Figure 4 code is Python 3.6 (pipeline_pb_strains) or 2.7 (pipeline_blast_16s), so environments may need legacy interpreters

## Evidence

- [discussion] we highlighted the prevalence of sequencing errors in PacBio CCS reads as a factor that limits the ability to resolve highly similar sequences. A particular problem was deletion errors coincident: "we highlighted the prevalence of sequencing errors in PacBio CCS reads as a factor that limits the ability to resolve highly similar sequences. A particular problem was deletion errors coincident"
- [discussion] we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [readme] Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures.: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
- [abstract] full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach.: "The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach."
