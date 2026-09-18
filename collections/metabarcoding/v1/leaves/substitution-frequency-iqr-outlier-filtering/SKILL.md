---
name: substitution-frequency-iqr-outlier-filtering
description: Use when calling intragenomic 16S SNPs from per-isolate PacBio CCS full-length
  16S alignments where systematic platform errors could inflate SNP counts, and error-artifact
  positions must be removed before tabulating unique SNP profiles.
when_to_use_negative:
- Variant discovery targets insertions/deletions rather than substitutions — the paper
  notes a single genuine deletion in one of the seven E. coli strain K-12 substr.
  MG1655 16S genes, which this substitution-only, IQR-filtered approach cannot recover.
- Input is short-read sequencing of a 16S variable region (e.g., V4); the paper shows
  partial sub-regions cannot match the taxonomic resolution of the full ~1500 bp gene,
  so substitution frequencies lack the accuracy needed for intragenomic copy variant
  calling.
- An isolate has fewer than 200 reads aligned by Cross_match to its most-abundant
  sequence, leaving too sparse a substitution-frequency distribution for a meaningful
  9-IQR outlier baseline.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3301
- http://edamontology.org/topic_3168
tools:
- name: Cross_match
  role: Aligns each isolate's PacBio CCS reads to the most-abundant full-length 16S
    sequence to produce the per-position substitution frequencies that the 9-IQR outlier
    filter screens; also enforces the ≥200-aligned-reads coverage filter
- name: PacBio
  role: Long-read platform producing the full-length 16S CCS reads whose systematic
    substitution and deletion errors (coincident with homopolymer runs) motivate substitution-only,
    outlier-filtered SNP calling
- name: ruffus
  role: CGAT ruffus framework structuring the Fig. 4 pipeline (pipeline_pb_strains,
    Python 3.6) that contains the substitution-frequency IQR filtering stage
  repo: https://github.com/cgat-developers/ruffus
- name: weinstock_full_length_16s
  role: Repository holding the isolate PacBio data and the Figure 4 code for the SNP-profile
    analysis in which this filtering step runs
  repo: github.com/TheJacksonLaboratory/weinstock_full_length_16s
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:53760945f2e7a70a9d87789a5cd9c024cca222839a47e824a1ef899941c64870
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cross_match
  - PacBio
  - ruffus
  - weinstock_full_length_16s
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3301
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# substitution-frequency-iqr-outlier-filtering

## Summary

A statistical filter for intragenomic 16S SNP calling that flags candidate SNP positions whose PacBio CCS substitution frequency is a 9-IQR outlier as systematic sequencing-error artifacts and removes them, so that only true 16S gene copy polymorphisms are counted. It underpins the Johnson et al. Figure 4 SNP-profile analysis of cultured human gut isolates.

## When to use

Use when calling intragenomic 16S SNPs from per-isolate PacBio CCS full-length 16S alignments where systematic platform errors could inflate SNP counts, and error-artifact positions must be removed before tabulating unique SNP profiles.

## When NOT to use

- Variant discovery targets insertions/deletions rather than substitutions — the paper notes a single genuine deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes, which this substitution-only, IQR-filtered approach cannot recover.
- Input is short-read sequencing of a 16S variable region (e.g., V4); the paper shows partial sub-regions cannot match the taxonomic resolution of the full ~1500 bp gene, so substitution frequencies lack the accuracy needed for intragenomic copy variant calling.
- An isolate has fewer than 200 reads aligned by Cross_match to its most-abundant sequence, leaving too sparse a substitution-frequency distribution for a meaningful 9-IQR outlier baseline.

## Inputs

- Per-isolate PacBio CCS full-length 16S read sets from the deposited isolate data (github.com/TheJacksonLaboratory/weinstock_full_length_16s)
- Most-abundant unique full-length 16S consensus sequence per isolate (Cross_match alignment reference)
- Cross_match alignments of reads vs. the per-isolate reference, restricted to isolates with at least 200 aligned reads

## Outputs

- Per-position substitution frequency table with systematic-error positions flagged as 9-IQR outliers
- Filtered intragenomic 16S SNP calls per isolate and per 99%-identity OTU, yielding unique SNP profiles (205 profiles across 349/381 isolates, 54/61 OTUs)

## How to apply

Per the Fig. 4 workflow, first align each of the 381 gut isolates' PacBio CCS reads to the isolate's most-abundant unique full-length 16S sequence using Cross_match, retaining only isolates with at least 200 aligned reads. Compute per-position substitution frequencies from these alignments, counting substitutions only and ignoring insertions/deletions because PacBio CCS indel errors (notably deletions coincident with homopolymer runs) are systematic rather than stochastic. Treat the distribution of substitution frequencies across positions as the error baseline and flag any candidate SNP position whose substitution frequency is a 9-IQR outlier as a systematic sequencing-error artifact, removing it from the SNP calls; the rationale is that such systematic errors occur at a given frequency and are not improved by greater sequencing effort. Downstream, discard OTUs clustered at 99% identity whose SNP positions show greater than 3% impurity across member isolates, then tabulate surviving SNP calls per isolate and OTU. Correct application reproduces the reported summary statistics: 58 putative species, 61 OTUs, 349/381 isolates (54/61 OTUs) with at least one SNP, and 205 unique SNP profiles as in Fig. 4a.

## Related tools

- **Cross_match** (Aligns each isolate's PacBio CCS reads to the most-abundant full-length 16S sequence to produce the per-position substitution frequencies that the 9-IQR outlier filter screens; also enforces the ≥200-aligned-reads coverage filter)
- **PacBio** (Long-read platform producing the full-length 16S CCS reads whose systematic substitution and deletion errors (coincident with homopolymer runs) motivate substitution-only, outlier-filtered SNP calling)
- **ruffus** (CGAT ruffus framework structuring the Fig. 4 pipeline (pipeline_pb_strains, Python 3.6) that contains the substitution-frequency IQR filtering stage) — https://github.com/cgat-developers/ruffus
- **weinstock_full_length_16s** (Repository holding the isolate PacBio data and the Figure 4 code for the SNP-profile analysis in which this filtering step runs) — github.com/TheJacksonLaboratory/weinstock_full_length_16s

## Examples

```
python pipeline_pb_strains.py --help
```

## Evaluation signals

- Recomputed summary counts match the paper: 58 putative species, 61 OTUs at 99% identity, 349/381 isolates (54/61 OTUs) with at least one SNP, and 205 unique SNP profiles as in Fig. 4a
- Every removed SNP position is a 9-IQR outlier of the per-position substitution-frequency distribution; no non-outlier SNP calls are dropped
- Final SNP calls contain no indel-derived variants — only substitutions, consistent with ignoring PacBio CCS systematic indel errors
- All isolates entering the filter have ≥200 Cross_match-aligned reads against their most-abundant 16S sequence
- OTUs whose SNP positions exceed 3% impurity across member isolates are discarded downstream of the filter

## Limitations

- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences and may occur at a given frequency not improved by greater sequencing effort
- Focusing on substitutions and ignoring indels is imperfect — a single real deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes would be missed
- Results depend on the surrounding thresholds (≥200 aligned reads, 99% OTU identity, >3% impurity discard), and the study stopped short of determining how well 99% identity separates intragenomic from inter-genomic variation for all bacterial species

## Evidence

- [abstract] full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [discussion] such systematic errors may occur at a given frequency and hence may not be improved by greater sequencing effort: "such systematic errors may occur at a given frequency and hence may not be improved by greater sequencing effort"
- [readme] Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures.: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
- [readme] The code underlying Figure 4 is written in python 3.6 (pipeline_pb_strains) or 2.7 (pipeline_blast_16s).: "The code underlying Figure 4 is written in python 3.6 (pipeline_pb_strains) or 2.7 (pipeline_blast_16s)."
