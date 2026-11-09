---
name: in-silico-amplicon-primer-truncation-evaluation
description: Use when primer-derived in-silico 16S amplicon FASTA sequences exist
  and you need to quantify how badly a sub-region (esp. V4) fails confident species-level
  classification versus full-length V1-V9.
when_to_use_negative:
- Input is real sequencing data (e.g., Nanopore/PacBio full-length reads) rather than
  in-silico amplicons derived from references with known source species — there is
  no ground-truth species to compare against.
- The goal is abundance estimation or OTU clustering of full-length sequences (99%
  identity OTUs), not resolution benchmarking of sub-regions.
- No reference taxonomy database is available or a non-Greengenes reference is required
  — failure rates are defined relative to the Greengenes source database.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: mothur
  role: runs classify.seqs with the Wang Bayesian classifier to assign taxonomic lineages
    to in-silico amplicons against the Greengenes reference, with swept confidence
    cutoffs
- name: ruffus (CGAT framework)
  role: pipeline framework used to organise the Figure 1 analysis code in the paper's
    repository
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:c6494f0e201b44bdeff4002f83c705b736fede2e64be6467a58072c680560b58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - mothur
  - ruffus (CGAT framework)
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# in-silico-amplicon-primer-truncation-evaluation

## Summary

Evaluates whether commonly targeted 16S rRNA sub-regions (e.g., V4) lose species-level taxonomic resolution relative to the full ~1500 bp gene, by classifying in-silico amplicons against their source Greengenes reference with the Wang Bayesian classifier in mothur.

## When to use

Use when primer-derived in-silico 16S amplicon FASTA sequences exist and you need to quantify how badly a sub-region (esp. V4) fails confident species-level classification versus full-length V1-V9.

## When NOT to use

- Input is real sequencing data (e.g., Nanopore/PacBio full-length reads) rather than in-silico amplicons derived from references with known source species — there is no ground-truth species to compare against.
- The goal is abundance estimation or OTU clustering of full-length sequences (99% identity OTUs), not resolution benchmarking of sub-regions.
- No reference taxonomy database is available or a non-Greengenes reference is required — failure rates are defined relative to the Greengenes source database.

## Inputs

- in-silico 16S amplicon FASTA sequences (per sub-region and full-length V1-V9, from task_001 primer truncation)
- Greengenes reference sequence alignment (FASTA)
- Greengenes reference taxonomy file

## Outputs

- fig1b_species_failure_rates.csv: per-amplicon-region species-level failure rates at each confidence cutoff
- per-amplicon taxonomy assignments from classify.seqs at cutoffs 30-98

## How to apply

Load the in-silico amplicon FASTA sequences (produced by primer-based truncation of full-length 16S) together with the Greengenes reference alignment and taxonomy files. Run mothur classify.seqs with the Wang Bayesian classifier, sweeping the confidence cutoff parameter across 30 to 98 so taxonomic assignments exist at every threshold. Extract the species-level label from each assignment and compare it to the known source species of the reference genome from which each amplicon was derived. Compute the per-region species-level failure rate (fraction of amplicons whose species assignment differs from or fails to reach the source species) for each 16S sub-region and for the full-length V1-V9 amplicon. Rationale: the paper shows short-read targeting of variable regions cannot match the taxonomic accuracy of sequencing the entire gene, so failure rates against the ground-truth source species directly measure resolution loss.

## Related tools

- **mothur** (runs classify.seqs with the Wang Bayesian classifier to assign taxonomic lineages to in-silico amplicons against the Greengenes reference, with swept confidence cutoffs)
- **ruffus (CGAT framework)** (pipeline framework used to organise the Figure 1 analysis code in the paper's repository) — https://github.com/cgat-developers/ruffus

## Examples

```
mothur "#classify.seqs(fasta=in_silico_v4_amplicons.fasta, reference=greengenes_ref.fasta, taxonomy=greengenes.tax, cutoff=80)"
```

## Evaluation signals

- Per-region failure rates in fig1b_species_failure_rates.csv reproduce the reported Fig. 1b values: V4 is the worst case at 56% species-level failure.
- Full-length V1-V9 amplicons show near-complete recovery of the correct source species at species-level classification.
- The confidence cutoff sweep spans the full 30-98 range and failure rates are reported at consistent thresholds across sub-regions.
- Every amplicon's extracted species label can be traced to the known source species of the reference genome it was derived from.
- Output CSV has one row per 16S sub-region plus full-length V1-V9, with failure-rate fractions in [0,1].

## Limitations

- Results are reference-database dependent: failure rates are defined against the Greengenes database from which the amplicons were derived, so they do not transfer directly to other taxonomies.
- The paper found none of the variable regions covered by partial 16S sequencing recaptured the diversity of the full ~1500 bp gene — sub-region results should be treated as a lower bound on resolution loss.
- Species-level failure is sensitive to the classify.seqs confidence cutoff; sweeping 30-98 mitigates but does not eliminate threshold choice effects.

## Evidence

- [results] V4 worst case 56% species-level failure: [evidence span withheld — anchor status: not_found]
- [intro] sub-regions cannot match full-gene accuracy: "commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the"
- [discussion] variable regions lose diversity: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
- [readme] repo organises analysis as ruffus pipelines per figure: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures. Each pipeline is designed to be run using a"
