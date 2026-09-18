---
name: 16s-substitution-error-modeling
description: 'Use when calling intragenomic 16S SNPs from PacBio CCS read alignments:
  you must distinguish real intra-genome substitutions from systematic CCS errors
  (e.g., deletions at homopolymers) before counting SNP profiles.'
when_to_use_negative:
- Input reads are short-read V4-region 16S data, which cannot resolve full-gene-length
  intragenomic copy variants
- The goal involves indel-based variant calling — the paper shows PacBio CCS deletion
  errors at homopolymers make indels unreliable (e.g., a single genuine deletion in
  one of seven E. coli K-12 MG1655 16S copies is missed by this approach)
- Isolates fail the >=200 aligned-read coverage filter; SNP calls at low coverage
  are not trustworthy
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: Cross_match
  role: Aligns per-isolate PacBio CCS reads to the most-abundant full-length 16S sequence
    to derive per-position substitution frequencies
  repo: github.com/TheJacksonLaboratory/weinstock_full_length_16s
- name: PacBio
  role: Long-read CCS platform producing the full-length 16S reads whose substitution-level
    accuracy enables intragenomic SNP detection
  repo: github.com/TheJacksonLaboratory/weinstock_full_length_16s
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the figure-wise analyses,
    including the Figure 4 strain/SNP pipeline (pipeline_pb_strains, Python 3.6)
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:be9ae53985b654cbdcb16bc146589fd1ba66c12c393c7376583e522bab01d067
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TheJacksonLaboratory/weinstock_full_length_16s
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# 16S substitution error modeling

## Summary

A method for separating genuine intragenomic 16S rRNA gene copy polymorphisms from systematic sequencing artifacts in PacBio CCS full-length 16S data, using substitution-only analysis, coverage filters, and outlier-based error models.

## When to use

Use when calling intragenomic 16S SNPs from PacBio CCS read alignments: you must distinguish real intra-genome substitutions from systematic CCS errors (e.g., deletions at homopolymers) before counting SNP profiles.

## When NOT to use

- Input reads are short-read V4-region 16S data, which cannot resolve full-gene-length intragenomic copy variants
- The goal involves indel-based variant calling — the paper shows PacBio CCS deletion errors at homopolymers make indels unreliable (e.g., a single genuine deletion in one of seven E. coli K-12 MG1655 16S copies is missed by this approach)
- Isolates fail the >=200 aligned-read coverage filter; SNP calls at low coverage are not trustworthy

## Inputs

- PacBio CCS full-length 16S read sets per cultured isolate (from github.com/TheJacksonLaboratory/weinstock_full_length_16s)
- Most-abundant representative full-length 16S sequence per isolate (consensus/reference for alignment)

## Outputs

- Per-position substitution frequency tables aligned to the isolate consensus sequence
- Filtered SNP calls per isolate and per OTU (99% identity), after 9-IQR outlier removal and 3% impurity filtering
- Unique SNP profiles suitable for phasing into substitution profiles (e.g., 205 profiles across 349/381 isolates in the reference study)

## How to apply

For each of the 381 gut isolates, identify the most abundant unique full-length 16S sequence from the deposited PacBio data, then align all CCS reads to that representative with Cross_match, retaining only isolates with at least 200 aligned reads per the paper's coverage filter. Compute per-position substitution frequencies only from substitutions, deliberately ignoring insertions/deletions because PacBio CCS indel errors are systematic (notably deletion errors coincident with homopolymer runs). Flag SNP positions whose substitution frequency behaves as a 9-IQR outlier as systematic sequencing-error artifacts and remove them from the SNP calls. At the OTU level (99% identity clustering), discard OTUs whose SNP positions show greater than 3% impurity across member isolates. The surviving SNP calls per isolate and OTU yield per-isolate polymorphism calls and unique SNP profiles, phaseable into substitution profiles for closely related taxa.

## Related tools

- **Cross_match** (Aligns per-isolate PacBio CCS reads to the most-abundant full-length 16S sequence to derive per-position substitution frequencies) — github.com/TheJacksonLaboratory/weinstock_full_length_16s
- **PacBio** (Long-read CCS platform producing the full-length 16S reads whose substitution-level accuracy enables intragenomic SNP detection) — github.com/TheJacksonLaboratory/weinstock_full_length_16s
- **ruffus** (CGAT ruffus pipeline framework used to organize the figure-wise analyses, including the Figure 4 strain/SNP pipeline (pipeline_pb_strains, Python 3.6)) — https://github.com/cgat-developers/ruffus

## Examples

```
python pipeline_pb_strains.py --isolate-isos align all CCS reads to each isolate consensus with cross_match; flag SNP positions whose substitution frequency is a 9-IQR outlier; drop OTUs with >3% SNP impurity at 99% identity
```

## Evaluation signals

- Recomputed summary counts must match reported values: 58 putative species, 61 OTUs at 99% identity, 349 of 381 isolates (54 of 61 OTUs) with >=1 SNP, and 205 unique SNP profiles (Fig. 4a)
- Every retained isolate has >=200 CCS reads aligned to its consensus sequence via Cross_match
- No SNP call is retained at a position flagged as a 9-IQR substitution-frequency outlier
- No OTU survives whose SNP positions exceed 3% impurity across member isolates
- Insertions and deletions contribute zero SNP calls (substitution-only analysis); verify homopolymer-associated deletion artifacts were excluded

## Limitations

- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences, so indel-based variation is ignored entirely
- Ignoring insertions/deletions is an imperfect approach, demonstrated by a single genuine deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes
- Systematic errors may occur at a given frequency and may not be improved by greater sequencing effort
- The 99% identity threshold's ability to separate intragenomic vs. inter-genomic variation was not established for all bacterial species

## Evidence

- [discussion] we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [discussion] The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach.: "The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach."
- [readme] Code is organised into a set of ruffus pipelines pertaining to different figures.: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
- [abstract] full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
