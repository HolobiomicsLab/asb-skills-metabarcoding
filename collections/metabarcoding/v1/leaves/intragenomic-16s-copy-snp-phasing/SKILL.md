---
name: intragenomic-16s-copy-snp-phasing
description: Use when analyzing full-length (~1500 bp) PacBio CCS 16S reads from bacterial
  isolates and the goal is to detect intragenomic 16S copy polymorphisms and build
  phased SNP profiles for strain-level discrimination.
when_to_use_negative:
- Input reads are short reads of 16S variable regions (e.g., V4) — short-read sequencing
  cannot achieve the taxonomic resolution of the full ~1500 bp gene needed to resolve
  copy variants
- The analysis goal requires resolving indel differences between intragenomic 16S
  copies — PacBio CCS indel errors are systematic, which is why the method ignores
  insertions/deletions
- Isolates have fewer than 200 reads aligned to their reference sequence — the paper's
  coverage filter excludes them from SNP calling
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: Cross_match
  role: Aligns each isolate's PacBio CCS reads to its most-abundant full-length 16S
    sequence to compute per-position substitution frequencies
- name: PacBio
  role: Long-read platform generating the full-length 16S CCS reads in which substitutions
    between intragenomic copies are resolvable
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the Figure 4 analysis code
    (pipeline_pb_strains) in the weinstock_full_length_16s repository
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:06ef7cceb73fd2f12a860954a387036cbc4ae2ef58bfa32f87c7836004233527
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# intragenomic-16s-copy-snp-phasing

## Summary

Detect and phase intragenomic 16S rRNA gene copy SNPs from full-length PacBio CCS isolate reads, yielding SNP-based substitution profiles that discriminate strains within a species. This exploits the finding that PacBio long reads resolve substitutions (but not indels) between intragenomic 16S copies.

## When to use

Use when analyzing full-length (~1500 bp) PacBio CCS 16S reads from bacterial isolates and the goal is to detect intragenomic 16S copy polymorphisms and build phased SNP profiles for strain-level discrimination.

## When NOT to use

- Input reads are short reads of 16S variable regions (e.g., V4) — short-read sequencing cannot achieve the taxonomic resolution of the full ~1500 bp gene needed to resolve copy variants
- The analysis goal requires resolving indel differences between intragenomic 16S copies — PacBio CCS indel errors are systematic, which is why the method ignores insertions/deletions
- Isolates have fewer than 200 reads aligned to their reference sequence — the paper's coverage filter excludes them from SNP calling

## Inputs

- Full-length PacBio CCS 16S rRNA gene reads per isolate (from github.com/TheJacksonLaboratory/weinstock_full_length_16s deposited isolate data)
- Most-abundant unique full-length 16S reference sequence per isolate
- 99%-identity OTU clustering of full-length 16S sequences

## Outputs

- Per-isolate table of SNP positions and substitution frequencies across the 16S gene
- Phased SNP substitution profiles (unique SNP profiles, e.g. 205 for 381 gut isolates)
- Per-OTU SNP prevalence summary (e.g. 349/381 isolates, 54/61 OTUs with ≥1 SNP)

## How to apply

Identify the most abundant unique full-length 16S sequence for each isolate from the PacBio CCS data, then align all of that isolate's CCS reads to it with Cross_match, retaining only isolates with at least 200 aligned reads. Compute per-position substitution frequencies while deliberately ignoring insertions and deletions, because PacBio CCS deletion errors are systematic (coincident with homopolymer runs). Flag SNP positions whose substitution frequency is a 9-IQR outlier as systematic sequencing-error artifacts and remove them. Cluster full-length sequences at 99% identity into OTUs and discard OTUs whose member isolates show greater than 3% SNP impurity. Finally, phase the surviving SNPs into substitution profiles per isolate/OTU; for the 381 gut isolates in Johnson et al., this yields 349/381 isolates (54/61 OTUs) with at least one SNP and 205 unique SNP profiles.

## Related tools

- **Cross_match** (Aligns each isolate's PacBio CCS reads to its most-abundant full-length 16S sequence to compute per-position substitution frequencies)
- **PacBio** (Long-read platform generating the full-length 16S CCS reads in which substitutions between intragenomic copies are resolvable)
- **ruffus** (CGAT ruffus pipeline framework used to organize the Figure 4 analysis code (pipeline_pb_strains) in the weinstock_full_length_16s repository) — https://github.com/cgat-developers/ruffus

## Examples

```
cd weinstock_full_length_16s && python pipeline_pb_strains.py
```

## Evaluation signals

- Recomputed isolate/OTU SNP prevalence matches the reported 349 of 381 isolates and 54 of 61 OTUs with at least one SNP
- Number of unique SNP profiles matches the reported 205 (Fig. 4a) after the 9-IQR sequencing-error filter
- OTU count at 99% identity matches the reported 61 OTUs (58 putative species)
- Every called isolate passed the coverage filter of at least 200 reads aligned to its most-abundant sequence
- No SNP calls derive from insertion/deletion positions; only substitution frequencies were used, per the platform-error rationale

## Limitations

- Ignoring insertions/deletions is an imperfect approach, demonstrated by a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes
- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences, and may occur at a fixed frequency not improved by greater sequencing effort
- The study stopped short of determining how well the 99% identity threshold separates intragenomic vs. inter-genomic variation for all bacterial species

## Evidence

- [abstract] full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [discussion] Phasing of 16S gene SNPs produced highly similar substitution profiles for closely related taxa: "Phasing of 16S gene SNPs produced highly similar substitution profiles for closely related taxa"
- [discussion] The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach.: "The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach."
- [readme] Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures.: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
