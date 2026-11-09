---
name: long-read-error-aware-variant-calling
description: Use when calling within-genome 16S gene copy variants from full-length
  PacBio amplicon reads (e.g., B. vulgatus OTUs at 99% identity) and substitution
  frequencies must be separated from platform-specific indel errors.
when_to_use_negative:
- When the variants of interest are insertions or deletions — PacBio CCS deletion
  errors coincident with homopolymer runs make indel calls unreliable, and the paper
  notes a single real deletion in an E. coli K-12 MG1655 16S gene would be missed
  by this substitution-only approach
- When working with short-read 16S variable-region data (e.g., V4), which cannot resolve
  full-gene intragenomic copy variants
- When reads are not full-length 16S amplicons, since resolving divergent intragenomic
  copies requires sequencing the entire ~1500 bp gene
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: PacBio
  role: Long-read sequencing platform generating the full-length CCS 16S amplicon
    reads used to resolve intragenomic 16S copy variants
- name: Cross_match
  role: Aligner used to align B. vulgatus OTU amplicons from Scott and IronHorse samples
    to the reference 16S gene for substitution tabulation
- name: ruffus
  role: Pipeline framework used to organize the analysis workflows in the weinstock_full_length_16s
    repository
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:0f2c90ae000183ba0f4c24de62272dc2d7429784f97a62a224352150ca539ba5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cross_match
  - PacBio
  - ruffus
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# long-read-error-aware-variant-calling

## Summary

Detects intragenomic 16S rRNA gene copy polymorphisms from full-length PacBio CCS amplicons by aligning reads to a reference with Cross_match and tabulating only nucleotide substitutions, deliberately ignoring insertions/deletions because systematic PacBio CCS deletion errors coincide with homopolymer runs.

## When to use

Use when calling within-genome 16S gene copy variants from full-length PacBio amplicon reads (e.g., B. vulgatus OTUs at 99% identity) and substitution frequencies must be separated from platform-specific indel errors.

## When NOT to use

- When the variants of interest are insertions or deletions — PacBio CCS deletion errors coincident with homopolymer runs make indel calls unreliable, and the paper notes a single real deletion in an E. coli K-12 MG1655 16S gene would be missed by this substitution-only approach
- When working with short-read 16S variable-region data (e.g., V4), which cannot resolve full-gene intragenomic copy variants
- When reads are not full-length 16S amplicons, since resolving divergent intragenomic copies requires sequencing the entire ~1500 bp gene

## Inputs

- Full-length (~1500 bp) PacBio CCS 16S amplicon reads per sample (e.g., Scott and IronHorse stool samples)
- OTU-assigned read sets clustered at 99% identity for the target taxon (e.g., B. vulgatus)
- Reference 16S gene sequence for the species
- RefSeq 16S sequences for comparator strains (e.g., B. vulgatus ATCC 8482, mpk40)

## Outputs

- Per-position nucleotide substitution frequency profiles, organized by sample and reference strain (substitution_profiles.csv)
- Per-sample polymorphism frequencies (e.g., V5 polymorphism at 84% Scott, 69% IronHorse)
- RefSeq-predicted substitution profiles for each comparator strain

## How to apply

Retrieve full-length PacBio 16S amplicon reads (e.g., from github.com/TheJacksonLaboratory/weinstock_full_length_16s) and select reads assigned to the taxon-specific OTU clustered at 99% identity. Align the OTU amplicons to a single reference 16S gene sequence (plus RefSeq 16S sequences for comparator strains such as ATCC 8482 and mpk40) using Cross_match. Tabulate nucleotide substitutions at each aligned position, explicitly discarding insertion and deletion calls because PacBio CCS deletion errors are systematic and coincide with homopolymer runs, so indel-based variant calls are unreliable. Compute the fraction of reads carrying each polymorphism (e.g., the V5 polymorphism) per sample and derive expected RefSeq-predicted substitution profiles for comparison, writing per-position profiles organized by sample and reference strain to substitution_profiles.csv. Validate by checking that computed polymorphism frequencies match expected copy-number fractions (e.g., 84% Scott, 69% IronHorse, consistent with 6 and 5 of seven 16S copies) and that profiles reproduce the paper's Fig. 3c/3d patterns.

## Related tools

- **PacBio** (Long-read sequencing platform generating the full-length CCS 16S amplicon reads used to resolve intragenomic 16S copy variants)
- **Cross_match** (Aligner used to align B. vulgatus OTU amplicons from Scott and IronHorse samples to the reference 16S gene for substitution tabulation)
- **ruffus** (Pipeline framework used to organize the analysis workflows in the weinstock_full_length_16s repository) — https://github.com/cgat-developers/ruffus

## Examples

```
python pipeline_fig3.py --reads weinstock_full_length_16s --otu B_vulgatus_99 --ref b_vulgatus_16s.fasta --strains ATCC_8482,mpk40 --out substitution_profiles.csv
```

## Evaluation signals

- Computed V5 polymorphism frequencies match reported values: 84% in Scott and 69% in IronHorse samples
- Frequencies are consistent with the polymorphism being present in six and five of seven 16S gene copies, respectively, as predicted from RefSeq strain sequences
- Per-position substitution profiles reproduce the patterns shown in Fig. 3c and 3d of the paper
- Output substitution_profiles.csv contains per-position profiles organized by sample and reference strain (ATCC 8482 and mpk40)
- Insertion and deletion calls are excluded from the polymorphism tabulation, per the substitution-only design

## Limitations

- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences and make indel calls unreliable
- Systematic errors may occur at a given fixed frequency and may not be improved by greater sequencing effort
- Ignoring insertions/deletions is an imperfect approach, as demonstrated by a single real deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes
- The 99% identity OTU threshold's ability to separate intragenomic from inter-genomic variation was not determined for all bacterial species
- Future improvements may come from advances in sequencing platforms, chemistries, and computational approaches, which were not determined in this study

## Evidence

- [abstract] full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [discussion] full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
- [discussion] Phasing of 16S gene SNPs produced highly similar substitution profiles for closely related taxa: "Phasing of 16S gene SNPs produced highly similar substitution profiles for closely related taxa"
- [readme] Each pipeline is designed to be run using a [CGAT ruffus framework](https://github.com/cgat-developers/ruffus).: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
