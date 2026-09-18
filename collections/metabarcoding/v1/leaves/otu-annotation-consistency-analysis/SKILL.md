---
name: otu-annotation-consistency-analysis
description: Use when OTUs were built by clustering 16S sequences (e.g. HMP V3V5/V1V3
  FASTAs at 97% similarity) with a single representative sequence, and you need to
  measure how representative-sequence annotation propagates (or fails) across OTU
  members.
when_to_use_negative:
- Input is already a feature table or BIOM file with pre-assigned taxonomy rather
  than raw OTU member/representative FASTA sequences
- The analysis goal is OTU clustering itself (choosing PSD/MSD/BLD thresholds) rather
  than auditing annotation consistency of existing 97%-similarity OTUs
- Sequences come from de novo phylogenetic placement methods (e.g. SEPP/pplacer) instead
  of representative-sequence-based OTU annotation
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: QIIME
  role: Runs the RDP Classifier via the assign_taxonomy step with the Greengenes reference
    training set to classify both representative and member sequences
- name: RDP Classifier
  role: The taxonomic classifier used to annotate representative and member sequences
    for disagreement detection
- name: phylo
  role: Repository companion from the paper's authors for phylogenetic/sequence-similarity
    analyses of HMP OTUs (minimal README provided)
  repo: github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:ed819165b8faa9f82da26102677f3a9869a1eb1eab55e75bccd627ac546237d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - QIIME
  - RDP Classifier
  - phylo
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# otu-annotation-consistency-analysis

## Summary

Quantifies how often member sequences within an HMP-derived OTU receive an RDP Classifier (Greengenes) annotation that differs from the OTU's representative sequence, stratified by 16S region (V3V5, V1V3) and by sequence divergence (average MSD >= 0.03).

## When to use

Use when OTUs were built by clustering 16S sequences (e.g. HMP V3V5/V1V3 FASTAs at 97% similarity) with a single representative sequence, and you need to measure how representative-sequence annotation propagates (or fails) across OTU members.

## When NOT to use

- Input is already a feature table or BIOM file with pre-assigned taxonomy rather than raw OTU member/representative FASTA sequences
- The analysis goal is OTU clustering itself (choosing PSD/MSD/BLD thresholds) rather than auditing annotation consistency of existing 97%-similarity OTUs
- Sequences come from de novo phylogenetic placement methods (e.g. SEPP/pplacer) instead of representative-sequence-based OTU annotation

## Inputs

- OTU FASTA files with representative and member sequences for V3V5, V1V5, and Lactobacillus V1V3 regions (HMP-derived, clustered at 97% similarity)
- Greengenes reference training set (reference sequences + taxonomy) for the RDP Classifier
- MSD (maximum sequence dissimilarity) values between each member and its OTU representative

## Outputs

- annotation_instability.csv with region / stratum (overall vs high-MSD, Lactobacillus V1V3) / percentage fields
- Per-OTU flags indicating whether any member sequence's taxonomic assignment differs from the representative's at any rank

## How to apply

Load the OTU representative and member sequences for each region (V3V5, V1V5/Lactobacillus V1V3) from the OTU FASTA files. Classify representatives and every member with the RDP Classifier via QIIME's assign_taxonomy step against the Greengenes reference training set, using an identical configuration for both so assignments are comparable. Flag an OTU as unstable if any member differs from its representative's taxonomic assignment at any rank. Compute the overall instability percentage per region (expected: 12.9% V3V5, 19.8% V1V5), then restrict to high-divergence OTUs where the member-representative average MSD >= 0.03 and recompute (expected: 13.5% V3V5, 26.2% V1V5; 2.0% for Lactobacillus V1V3). The rationale is that OTUs are annotated from a single representative, so annotation errors concentrate in poorly clustered OTUs — 80.5% of V3V5 OTUs contain at least one sequence >3% dissimilar (MSD>0.03) from the representative.

## Related tools

- **QIIME** (Runs the RDP Classifier via the assign_taxonomy step with the Greengenes reference training set to classify both representative and member sequences)
- **RDP Classifier** (The taxonomic classifier used to annotate representative and member sequences for disagreement detection)
- **phylo** (Repository companion from the paper's authors for phylogenetic/sequence-similarity analyses of HMP OTUs (minimal README provided)) — github.com/namphuon/phylo

## Examples

```
assign_taxonomy.py -i otu_seqs.fasta -m rdp -r gg_97_otus.fasta -t gg_97_otu_taxonomy.txt
```

## Evaluation signals

- Each computed percentage in annotation_instability.csv matches the reported values (12.9% V3V5, 19.8% V1V5, 13.5% V3V5 high-MSD, 26.2% V1V5 high-MSD, 2.0% Lactobacillus V1V3) within rounding tolerance
- The CSV schema contains region, stratum, and percentage fields for all strata (region x overall/high-MSD, plus Lactobacillus V1V3)
- The high-MSD stratum is correctly restricted to OTUs whose member-representative average MSD >= 0.03
- An OTU is flagged only when a member's assignment differs from the representative's at any taxonomic rank, and classifier configuration is identical for representatives and members
- Sanity check against paper results: the majority of V3V5 OTUs (80.5%) contain at least one sequence with MSD > 0.03 from the representative, so the high-MSD subset must be large, not marginal

## Limitations

- The traditional pairwise-percent-similarity (PSD, typically 97%) approach can result in poorly clustered OTUs, so instability percentages are properties of that clustering choice and may not transfer to other clustering methods
- The 97% 16S rRNA similarity threshold used to delineate species is only a rough approximation, and percent sequence similarity can overestimate evolutionary similarity between pairs of sequences
- Instability is measured against a single representative sequence's annotation; the method cannot establish which annotation (representative's or member's) is correct

## Evidence

- [abstract] As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [intro] From the OTU cluster, a single sequence is selected as a representative sequence. The representative sequence is annotated using a 16S classification method: "From the OTU cluster, a single sequence is selected as a representative sequence. The representative sequence is annotated using a 16S classification method"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
