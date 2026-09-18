---
name: nucleotide-substitution-profiling
description: Use when full-length 16S PacBio reads have been assigned to a 99%-identity
  OTU and the goal is to resolve strain-level intragenomic 16S copy variants via per-position
  substitution frequencies rather than indels.
when_to_use_negative:
- The analysis target is insertion/deletion variation between 16S copies — PacBio
  CCS deletion errors coincident with homopolymer runs make indel calls unreliable;
  only substitutions are resolved
- Reads have not been clustered at the 99% identity OTU threshold, so inter-genomic
  (between-strain) variation would be confounded with intragenomic copy variation
- The data are short reads of a single 16S variable region (e.g., V4), which cannot
  recapture the diversity of the full ~1500 bp gene
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3693
- http://edamontology.org/topic_3697
tools:
- name: PacBio
  role: Long-read platform whose full-length CCS reads provide the accuracy needed
    to resolve subtle nucleotide substitutions between intragenomic 16S copies
- name: Cross_match
  role: Aligner used to align full-length 16S OTU amplicons from each sample to the
    reference 16S gene for per-position substitution tabulation
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the analysis code per figure
    in the weinstock_full_length_16s repository
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:68eb6c733f9620702733088944c7b93fcb447457517a4969cab3d32d38d28de7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cross_match
  - PacBio
  - ruffus
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3693
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# nucleotide-substitution-profiling

## Summary

Profiles single-nucleotide substitution frequencies across full-length (~1500 bp) PacBio 16S amplicon reads to detect intragenomic variation between 16S gene copies within a genome, deliberately excluding insertions/deletions because systematic PacBio CCS deletion errors make indel calls unreliable.

## When to use

Use when full-length 16S PacBio reads have been assigned to a 99%-identity OTU and the goal is to resolve strain-level intragenomic 16S copy variants via per-position substitution frequencies rather than indels.

## When NOT to use

- The analysis target is insertion/deletion variation between 16S copies — PacBio CCS deletion errors coincident with homopolymer runs make indel calls unreliable; only substitutions are resolved
- Reads have not been clustered at the 99% identity OTU threshold, so inter-genomic (between-strain) variation would be confounded with intragenomic copy variation
- The data are short reads of a single 16S variable region (e.g., V4), which cannot recapture the diversity of the full ~1500 bp gene

## Inputs

- Full-length PacBio 16S amplicon reads (FASTA/FASTQ) from stool samples, e.g., Scott and IronHorse from github.com/TheJacksonLaboratory/weinstock_full_length_16s
- Reference 16S gene sequence for the target taxon (e.g., single B. vulgatus reference 16S gene)
- RefSeq 16S sequences for comparison strains (e.g., B. vulgatus ATCC 8482 and mpk40)

## Outputs

- Per-position substitution profiles across the ~1500 bp 16S gene, organized by sample and reference strain (substitution_profiles.csv)
- Polymorphism frequency estimates per sample (e.g., V5 polymorphism at 84% Scott, 69% IronHorse), interpretable as copy-number fractions of the 16S gene pool

## How to apply

Retrieve full-length PacBio 16S amplicon reads (e.g., from the weinstock_full_length_16s repository), select reads belonging to the taxon of interest at the 99% identity OTU threshold, and align them to the single reference 16S gene sequence with Cross_match. Tabulate nucleotide substitutions at each aligned position, deliberately ignoring insertions and deletions because systematic PacBio CCS deletion errors coincident with homopolymer runs make indel calls unreliable. Compute the fraction of reads carrying each polymorphism (e.g., the V5 polymorphism) and compare the observed substitution profile to RefSeq-predicted profiles of known strains (e.g., B. vulgatus ATCC 8482 vs. mpk40) to infer how many of the 16S gene copies carry each variant. Pipelines are organized as CGAT ruffus pipelines per figure in the TheJacksonLaboratory/weinstock_full_length_16s repository (Python 2.7 for Figures 1-3).

## Related tools

- **PacBio** (Long-read platform whose full-length CCS reads provide the accuracy needed to resolve subtle nucleotide substitutions between intragenomic 16S copies)
- **Cross_match** (Aligner used to align full-length 16S OTU amplicons from each sample to the reference 16S gene for per-position substitution tabulation)
- **ruffus** (CGAT ruffus pipeline framework used to organize the analysis code per figure in the weinstock_full_length_16s repository) — https://github.com/cgat-developers/ruffus

## Examples

```
cross_match Scott_B_vulgatus_OTU.fasta B_vulgatus_16S_ref.fasta > scott_vs_ref.aln
```

## Evaluation signals

- Computed V5 polymorphism frequencies match the reported values: 84% for the Scott sample and 69% for the IronHorse sample
- Observed frequencies are consistent with copy-fraction expectations (e.g., 84% ≈ six of seven 16S copies; 69% ≈ five of seven, as in B. vulgatus ATCC 8482)
- Per-position substitution profiles reproduce the patterns shown in Fig. 3c and 3d of the paper
- Substitution profiles omit indel columns; no insertion/deletion events are called in the output profiles
- Comparison strains yield distinct RefSeq-predicted substitution profiles (ATCC 8482 vs. mpk40) that discriminate which strain profile best matches the in vivo sample

## Limitations

- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences and force the analysis to ignore indels entirely
- Such systematic errors may occur at a given frequency and may not be improved by greater sequencing effort
- Ignoring insertions/deletions is imperfect — a single deletion in one of the seven E. coli K-12 MG1655 16S genes demonstrates genuine indel variants will be missed
- The 99% identity OTU threshold does not necessarily separate intragenomic from inter-genomic variation for all bacterial species

## Evidence

- [discussion] substitutions not indels rationale: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [abstract] long-read accuracy for substitutions: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] PacBio CCS deletion error caveat: "we highlighted the prevalence of sequencing errors in PacBio CCS reads as a factor that limits the ability to resolve highly similar sequences."
- [discussion] systematic errors not improved by depth: "such systematic errors may occur at a given frequency and hence may not be improved by greater sequencing effort"
- [readme] ruffus pipeline organization: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
