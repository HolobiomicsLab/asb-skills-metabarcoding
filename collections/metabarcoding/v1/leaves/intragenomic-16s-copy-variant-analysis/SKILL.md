---
name: intragenomic-16s-copy-variant-analysis
description: Use when full-length (~1500 bp) 16S amplicon or WGS reads from a mock
  or in-vivo sample must be checked for strain-level intragenomic polymorphisms, i.e.,
  when divergent 16S copies within one genome must be resolved and their substitution
  frequencies estimated.
when_to_use_negative:
- The goal is to resolve insertions/deletions between highly similar 16S copies —
  systematic PacBio CCS deletion errors coincident with homopolymer runs make indel
  calls unreliable for this purpose
- Input is short-read sequencing of a single 16S variable region (e.g., V4), which
  cannot capture full-length intragenomic copy variation
- The organism is represented by only a single reference 16S copy, so no intragenomic
  polymorphism stoichiometry can be predicted for validation
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: PacBio
  role: CCS platform generating full-length 16S reads whose per-position substitution
    frequencies form the observed profile
  repo: github.com/TheJacksonLaboratory/weinstock_full_length_16s
- name: Cross_match
  role: Global pairwise aligner used to align B. vulgatus 99% OTU amplicons to the
    reference 16S gene before substitution tabulation
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the figure-specific analysis
    pipelines in the article's code repository
  repo: https://github.com/cgat-developers/ruffus
- name: minimap2
  role: Alternative global pairwise aligner mentioned for aligning PacBio CCS reads
    to the single 16S reference
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:ef118893b8fec8b1802438ba827b6d6b1ac406acd5582eb856a2495cb368f25c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cross_match
  - PacBio
  - minimap2
  - ruffus
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# intragenomic-16s-copy-variant-analysis

## Summary

Detects and validates intragenomic 16S rRNA gene copy variation by tallying per-position substitution frequencies from full-length PacBio CCS reads aligned to a single 16S reference, and validating the observed stoichiometry against the substitution profile predicted from all 16S copies in a RefSeq genome (e.g., the seven copies of E. coli K-12 MG1655).

## When to use

Use when full-length (~1500 bp) 16S amplicon or WGS reads from a mock or in-vivo sample must be checked for strain-level intragenomic polymorphisms, i.e., when divergent 16S copies within one genome must be resolved and their substitution frequencies estimated.

## When NOT to use

- The goal is to resolve insertions/deletions between highly similar 16S copies — systematic PacBio CCS deletion errors coincident with homopolymer runs make indel calls unreliable for this purpose
- Input is short-read sequencing of a single 16S variable region (e.g., V4), which cannot capture full-length intragenomic copy variation
- The organism is represented by only a single reference 16S copy, so no intragenomic polymorphism stoichiometry can be predicted for validation

## Inputs

- Full-length PacBio CCS 16S amplicon reads (FASTQ/FASTA), e.g., mock community or Scott/IronHorse stool samples, optionally filtered to a 99% identity OTU
- Illumina WGS reads for the same organism (validation layer)
- Single reference 16S gene sequence (FASTA) for the target taxon
- RefSeq genome(s) containing multiple annotated 16S copies (e.g., E. coli K-12 MG1655; B. vulgatus ATCC 8482 and mpk40) to derive predicted copy-stoichiometric profiles

## Outputs

- Per-position substitution frequency table written to substitution_profiles.csv (organized by sample and reference strain)
- Matplotlib figures of PacBio, Illumina, and RefSeq-predicted per-position substitution frequency profiles (reproducing Fig. 2a–c and Fig. 3c–d)
- Estimated copy-number stoichiometry of polymorphic sites (e.g., fraction of reads carrying the V5 polymorphism)

## How to apply

Align full-length PacBio CCS 16S reads (e.g., from the weinstock_full_length_16s repository, filtered to a 99%-identity OTU) to a single reference 16S gene sequence using a global pairwise aligner (minimap2, parasail/Smith-Waterman in Python, or Cross_match). Tally substitutions at each aligned position, deliberately ignoring insertions and deletions because systematic PacBio CCS deletion errors coincide with homopolymer runs and make indel calls unreliable. Optionally repeat the tally with Illumina WGS reads against the same 16S reference as an orthogonal validation of the substitution stoichiometry. Extract all 16S copies from the RefSeq genome (e.g., seven copies of E. coli K-12 MG1655, or B. vulgatus ATCC 8482 and mpk40), align them to the single reference, and derive the expected per-position substitution frequency profile from the copy stoichiometry. Plot the PacBio, Illumina, and predicted profiles side-by-side (reproducing Fig. 2a–c and Fig. 3c–d), report the fraction of reads carrying each polymorphism (e.g., the V5 polymorphism), and write per-position profiles and polymorphism frequencies to substitution_profiles.csv organized by sample and reference strain.

## Related tools

- **PacBio** (CCS platform generating full-length 16S reads whose per-position substitution frequencies form the observed profile) — github.com/TheJacksonLaboratory/weinstock_full_length_16s
- **Cross_match** (Global pairwise aligner used to align B. vulgatus 99% OTU amplicons to the reference 16S gene before substitution tabulation)
- **ruffus** (CGAT ruffus pipeline framework used to organize the figure-specific analysis pipelines in the article's code repository) — https://github.com/cgat-developers/ruffus
- **minimap2** (Alternative global pairwise aligner mentioned for aligning PacBio CCS reads to the single 16S reference)

## Examples

```
minimap2 -a ecoli_16s_reference.fasta pacbio_ccs_16s.fastq > pacbio_16s_aln.sam  # then tally per-position substitutions, excluding indels
```

## Evaluation signals

- The observed PacBio substitution profile for E. coli K-12 MG1655 mirrors exactly the profile predicted by aligning all seven RefSeq 16S copies to the single reference
- The Illumina WGS substitution profile agrees in stoichiometry with both the PacBio profile and the seven-copy prediction
- B. vulgatus V5 polymorphism frequencies match reported values (84% in Scott, 69% in IronHorse, consistent with the polymorphism being present in six and five of seven copies)
- substitution_profiles.csv contains per-position frequencies organized by sample and reference strain, and plots reproduce the patterns of Fig. 2a–c and Fig. 3c–d
- Insertion and deletion events are excluded from the tallies, confirming substitutions-only accounting

## Limitations

- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences, which is why indels are ignored
- Ignoring indels is imperfect: a single deletion exists in one of the seven E. coli K-12 MG1655 16S genes
- Systematic errors may occur at a fixed frequency and may not be reduced by greater sequencing effort
- The 99% identity clustering threshold used to separate intragenomic from inter-genomic variation was not validated for all bacterial species

## Evidence

- [abstract] full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] substitutions-only filter rationale: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [discussion] phasing produces substitution profiles: "Phasing of 16S gene SNPs produced highly similar substitution profiles for closely related taxa"
- [discussion] imperfect approach due to K-12 deletion: "The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach."
- [readme] repository code organization: [evidence span withheld — anchor status: not_found]
