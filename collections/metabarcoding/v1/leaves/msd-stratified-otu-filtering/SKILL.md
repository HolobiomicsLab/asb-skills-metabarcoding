---
name: msd-stratified-otu-filtering
description: Use when reproducing or extending HMP V1V3/V3V5 OTU analyses that need
  OTUs stratified by member-vs-representative MSD (threshold 0.03) and per-stratum
  annotation disagreement percentages.
when_to_use_negative:
- The analysis goal is OTU clustering itself or choosing a clustering threshold —
  this skill only evaluates annotation stability of already-clustered OTUs.
- Input is a feature table or already-annotated BIOM file rather than raw OTU representative/member
  sequences, since member-level reclassification is required.
- Phylogenetic metrics other than MSD (e.g. BLD) are the stratification variable —
  the targets here (0.03 MSD) are specific to mean sequence dissimilarity.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: QIIME
  role: Runs the RDP Classifier through its assign_taxonomy step with the Greengenes
    reference training set to classify both representative and member sequences identically
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:f3156d210673448af176ed56c3f5d8f9bb70f8512ddc021efb164f5126b3b5ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# msd-stratified-otu-filtering

## Summary

A skill for quantifying annotation instability of 97%-similarity OTUs: each OTU's member sequences are independently classified (RDP Classifier / Greengenes) and compared to the representative's annotation, then OTUs are stratified by member-vs-representative MSD (e.g. >= 0.03) to show how disagreement rates change in highly divergent OTUs.

## When to use

Use when reproducing or extending HMP V1V3/V3V5 OTU analyses that need OTUs stratified by member-vs-representative MSD (threshold 0.03) and per-stratum annotation disagreement percentages.

## When NOT to use

- The analysis goal is OTU clustering itself or choosing a clustering threshold — this skill only evaluates annotation stability of already-clustered OTUs.
- Input is a feature table or already-annotated BIOM file rather than raw OTU representative/member sequences, since member-level reclassification is required.
- Phylogenetic metrics other than MSD (e.g. BLD) are the stratification variable — the targets here (0.03 MSD) are specific to mean sequence dissimilarity.

## Inputs

- OTU FASTA files with representative and member sequences for HMP V3V5, V1V5, and Lactobacillus V1V3 regions
- Greengenes reference training set (reference sequences and taxonomy) for RDP Classifier
- Precomputed MSD values between each member sequence and its OTU representative (threshold 0.03)

## Outputs

- annotation_instability.csv with region / stratum (overall vs high-MSD) / percentage fields
- Per-OTU flags indicating whether any member sequence disagrees with the representative's taxonomic assignment at any rank

## How to apply

Load the OTU representative and member sequences for the V3V5, V1V5, and Lactobacillus V1V3 regions from the OTU FASTA files. Classify each representative and every member sequence with the RDP Classifier via QIIME's assign_taxonomy step using the Greengenes reference training set, keeping the classifier configuration identical for representatives and members. For each OTU, flag it if at least one member receives a taxonomic assignment differing from the representative at any rank, and compute the overall instability percentage per region (targets: V3V5 12.9%, V1V5 19.8%). Then restrict to high-MSD OTUs, i.e. those with at least one member having MSD >= 0.03 from the representative, and recompute instability (targets: 13.5% V3V5, 26.2% V1V5, 2.0% Lactobacillus V1V3). Rationale: because OTUs are annotated solely from one representative sequence, members that are >3% dissimilar (MSD>0.03) can carry a different taxonomy than the annotation inherited by the whole OTU.

## Related tools

- **QIIME** (Runs the RDP Classifier through its assign_taxonomy step with the Greengenes reference training set to classify both representative and member sequences identically)

## Examples

```
assign_taxonomy.py -i otu_seqs/rep_set.fasta -r gg_97_otus.fasta -t gg_97_otu_taxonomy.txt -m rdp -o rdp_taxonomy/
```

## Evaluation signals

- Each computed percentage in annotation_instability.csv matches the reported values (12.9% V3V5 overall, 19.8% V1V5 overall, 13.5% V3V5 high-MSD, 26.2% V1V5 high-MSD, 2.0% Lactobacillus V1V3) within rounding tolerance.
- The CSV contains the region/stratum/percentage fields with all strata present (region x overall-vs-high-MSD, plus Lactobacillus V1V3).
- The high-MSD stratum only includes OTUs with at least one member at MSD >= 0.03 from the representative.
- Representative and member sequences were classified with the identical RDP Classifier / Greengenes configuration, so disagreements reflect the data, not pipeline drift.
- The fraction of high-MSD OTUs is consistent with the article's report that 80.5% of V3V5 OTUs had at least one sequence with MSD>0.03.

## Limitations

- Percent sequence similarity clustering can overestimate evolutionary similarity, so 97% OTUs may contain members with MSD exceeding the 0.03 threshold — the very instability this skill measures.
- Poorly clustered OTUs bias downstream analyses because the single representative's annotation is applied to all remaining sequences in the OTU.
- The namphuon/phylo README provides no usage documentation, so reproducing MSD computation may require the article's methods rather than repo docs.

## Evidence

- [abstract] the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs: "the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs"
- [abstract] As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [intro] both MSD and BLD often exceed the 0.03 threshold for an OTU: "both MSD and BLD often exceed the 0.03 threshold for an OTU"
