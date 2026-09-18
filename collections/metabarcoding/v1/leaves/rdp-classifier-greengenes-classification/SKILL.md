---
name: rdp-classifier-greengenes-classification
description: Use when OTUs are annotated via a single representative sequence and
  you need to measure how often member sequences (HMP V3V5, V1V5, Lactobacillus V1V3)
  would receive a different RDP/Greengenes assignment.
when_to_use_negative:
- The goal is routine taxonomic profiling of a new microbiome sample rather than auditing
  the internal consistency of already-built OTUs
- OTUs were built phylogenetically (e.g., tree-based methods) rather than by percent
  sequence similarity at the 97% threshold, or member-level annotations are already
  available
- Input is a feature/BIOM table without the underlying member sequences, so per-sequence
  classification is impossible
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0084
tools:
- name: QIIME
  role: Runs the RDP Classifier via the assign_taxonomy step with the Greengenes reference
    training set to classify both representative and member OTU sequences
- name: phylo
  role: Article repository associated with computing MSD/BLD dissimilarities between
    representative and member sequences used to define the high-divergence stratum
  repo: github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:2af444ca6888d8587eec60dc47b6fa764d7bd7596b68127aac9e04c943dfe9a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - QIIME
  - phylo
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# rdp-classifier-greengenes-classification

## Summary

Quantifies annotation instability of 16S rRNA OTUs by independently classifying every member sequence with the RDP Classifier (Greengenes training set, run through QIIME's assign_taxonomy) and flagging OTUs where any member disagrees with the representative's taxonomic assignment, overall and stratified by MSD >= 0.03.

## When to use

Use when OTUs are annotated via a single representative sequence and you need to measure how often member sequences (HMP V3V5, V1V5, Lactobacillus V1V3) would receive a different RDP/Greengenes assignment.

## When NOT to use

- The goal is routine taxonomic profiling of a new microbiome sample rather than auditing the internal consistency of already-built OTUs
- OTUs were built phylogenetically (e.g., tree-based methods) rather than by percent sequence similarity at the 97% threshold, or member-level annotations are already available
- Input is a feature/BIOM table without the underlying member sequences, so per-sequence classification is impossible

## Inputs

- OTU FASTA files with representative and member sequences (HMP V3V5, V1V5, Lactobacillus V1V3 regions)
- Greengenes reference training set (reference sequences + taxonomy) for the RDP Classifier
- Per-member MSD values between member and representative sequences (for the MSD >= 0.03 stratum)

## Outputs

- annotation_instability.csv with region / stratum (overall vs high-MSD) / percentage fields
- Per-OTU flags indicating whether any member sequence disagrees with the representative's taxonomic annotation

## How to apply

Load the HMP-derived OTU representative and member FASTA files for the V3V5, V1V5, and Lactobacillus V1V3 regions. Classify each representative sequence with QIIME's assign_taxonomy using the RDP Classifier and the Greengenes reference training set, then classify every member sequence with the identical RDP/Greengenes configuration so assignments are directly comparable. Flag each OTU in which at least one member's taxonomic assignment differs from the representative's at any rank. Compute the overall instability percentage per region, then restrict to highly divergent OTUs — those with average MSD >= 0.03 between member and representative — and recompute; the article's framing is that annotation is inherited from one representative sequence, so divergence can invalidate the shared label.

## Related tools

- **QIIME** (Runs the RDP Classifier via the assign_taxonomy step with the Greengenes reference training set to classify both representative and member OTU sequences)
- **phylo** (Article repository associated with computing MSD/BLD dissimilarities between representative and member sequences used to define the high-divergence stratum) — github.com/namphuon/phylo

## Examples

```
assign_taxonomy.py -i otu_rep_and_members.fasta -r gg_97_otus.fasta -t gg_97_otus_taxonomy.txt -m rdp -o rdp_assignments/
```

## Evaluation signals

- annotation_instability.csv contains region/stratum/percentage fields and each percentage matches the reported targets (12.9%, 19.8%, 13.5%, 26.2%, 2.0%) within rounding tolerance
- All representative and member sequences were classified with the identical RDP Classifier / Greengenes configuration, so disagreement is attributable to sequence content, not pipeline settings
- The high-MSD stratum is restricted to OTUs with average MSD >= 0.03 between member and representative, matching the article's 3% dissimilarity threshold
- Per-OTU disagreement flags reproduce the definition: at least one member differs from the representative's annotation at any taxonomic rank

## Limitations

- The 97% sequence similarity threshold is only a rough approximation for delineating species, so OTU membership alone does not guarantee taxonomic homogeneity
- Percent sequence similarity (PSD) can overestimate evolutionary similarity; MSD and BLD often exceed the 0.03 threshold even for PSD-defined OTUs, which is precisely what drives annotation disagreement
- Results are stratified by hypervariable region (V3V5 vs V1V3) and dataset (HMP vs Lactobacillus), so instability rates do not transfer between regions

## Evidence

- [abstract] As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
- [intro] From the OTU cluster, a single sequence is selected as a representative sequence. The representative sequence is annotated using a 16S classification method: "From the OTU cluster, a single sequence is selected as a representative sequence. The representative sequence is annotated using a 16S classification method"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
