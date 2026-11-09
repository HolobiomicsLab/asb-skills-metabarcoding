---
name: species-level-classification-failure-rate-analysis
description: Use when evaluating whether a short-read 16S variable region (e.g., V4)
  can achieve species-level taxonomic resolution, by classifying in-silico amplicons
  against the reference database their source genomes came from.
when_to_use_negative:
- Amplicons were generated from a reference database other than the one used for classification
  — failure to match the source species would conflate database bias with regional
  resolution loss
- The goal is strain-level or intragenomic copy-variant analysis (requires full-length
  long-read data and SNP phasing, not RDP species-level classification of sub-region
  amplicons)
- Input is experimental sequencing reads rather than in-silico amplicons with known
  source species, since the ground-truth comparison to source species cannot be made
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: mothur
  role: Runs classify.seqs with the Wang Bayesian classifier to assign taxonomic lineages
    (and species-level labels) to in-silico amplicons against the Greengenes reference
    database across confidence cutoffs 30-98
- name: ruffus
  role: CGAT ruffus pipeline framework used to orchestrate the figure-level analysis
    pipelines in the source repository
  repo: github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:2f89402d27a38e2642c4dbb498054ef8528f82cbfe938a1140c4da53018b25d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - mothur
  - ruffus
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# species-level-classification-failure-rate-analysis

## Summary

Quantifies how often in-silico 16S amplicons from a given sub-region (e.g., V4) fail to be confidently classified to the species level by the RDP Wang Bayesian classifier against the Greengenes reference database, benchmarking sub-regions against the full-length V1-V9 gene.

## When to use

Use when evaluating whether a short-read 16S variable region (e.g., V4) can achieve species-level taxonomic resolution, by classifying in-silico amplicons against the reference database their source genomes came from.

## When NOT to use

- Amplicons were generated from a reference database other than the one used for classification — failure to match the source species would conflate database bias with regional resolution loss
- The goal is strain-level or intragenomic copy-variant analysis (requires full-length long-read data and SNP phasing, not RDP species-level classification of sub-region amplicons)
- Input is experimental sequencing reads rather than in-silico amplicons with known source species, since the ground-truth comparison to source species cannot be made

## Inputs

- In-silico amplicon FASTA sequences per 16S sub-region and full-length V1-V9 (from Greengenes reference genomes)
- Greengenes reference sequence alignment (FASTA)
- Greengenes reference taxonomy file
- Mapping of each amplicon to its source reference genome and known species

## Outputs

- Per-region species-level failure rates table (fig1b_species_failure_rates.csv)
- Taxonomy assignments per amplicon at each confidence cutoff (30-98) from mothur classify.seqs

## How to apply

Load the in-silico amplicon FASTA sequences (produced by primer-based extraction from full-length Greengenes reference genomes) together with the Greengenes reference alignment and taxonomy files. Run mothur's classify.seqs using the Wang Bayesian classifier against the Greengenes database, sweeping the confidence cutoff parameter from 30 to 98 to generate assignments at each threshold. For each amplicon, extract the species-level label and compare it to the known source species of the reference genome from which the amplicon was derived. Compute the per-region species-level failure rate (fraction of amplicons whose species assignment differs from, or fails to reach, the source species) for each 16S sub-region and for the full-length V1-V9 amplicon. The rationale, per Johnson et al., is that short-read targeting of variable regions cannot match the taxonomic resolution of the ~1500 bp full gene, and the V4 region performed worst with 56% of amplicons failing confident species-level assignment.

## Related tools

- **mothur** (Runs classify.seqs with the Wang Bayesian classifier to assign taxonomic lineages (and species-level labels) to in-silico amplicons against the Greengenes reference database across confidence cutoffs 30-98)
- **ruffus** (CGAT ruffus pipeline framework used to orchestrate the figure-level analysis pipelines in the source repository) — github.com/cgat-developers/ruffus

## Examples

```
mothur "#classify.seqs(fasta=in_silico_amplicons.fasta, reference=greengenes_alignment.fasta, taxonomy=greengenes_taxonomy.txt, cutoff=80)"
```

## Evaluation signals

- The V4 sub-region should show the worst species-level performance, with approximately 56% of in-silico amplicons failing to confidently match the sequence from which they were derived, matching the reported Fig. 1b value
- The full-length V1-V9 amplicon should show near-complete recovery of species-level assignment relative to all partial sub-regions
- No variable region should recapture the full diversity represented by the ~1500 bp gene — every sub-region's failure rate must exceed the full-length rate
- Failure rates must be computed per region as the fraction of amplicons whose species assignment differs from or fails to reach the source species, at each confidence cutoff swept from 30 to 98
- Output CSV (fig1b_species_failure_rates.csv) must contain a failure-rate row/column for each 16S sub-region plus the full-length V1-V9 amplicon

## Limitations

- Short-read sequencing of 16S variable regions cannot achieve the taxonomic resolution of sequencing the entire (~1500) bp gene, so sub-region failure rates are expected to be high regardless of classifier tuning
- None of the variable regions covered by partial 16S sequencing recaptured the diversity represented by the full ~1500 bp gene, so no sub-region choice fully mitigates the limitation
- Results are specific to the Greengenes reference database and the Wang classifier's confidence cutoff behavior; cutoff values between 30 and 98 materially change assignments

## Evidence

- [abstract] Short-read sequencing of 16S variable regions cannot achieve the taxonomic resolution of sequencing the entire (~1500 bp) gene: "We demonstrate that targeting of 16S variable regions with short-read sequencing platforms cannot achieve the taxonomic resolution afforded by sequencing the entire (~1500 bp) gene."
- [intro] Sub-regions such as V4 underperform full-length classification: "commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the"
- [discussion] No variable region recaptured full-length diversity: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
- [readme] Pipelines organized by figure in the source repo: [evidence span withheld — anchor status: not_found]
