---
name: representative-vs-member-taxonomy-comparison
description: Use when OTU members were annotated only via a single representative
  sequence and you need to measure how often member sequences disagree taxonomically,
  especially for OTUs whose members exceed 3% MSD from the representative.
when_to_use_negative:
- Sequences already carry per-sequence taxonomic assignments (no representative-inherited
  annotation is in use, so there is no disagreement to measure).
- The analysis goal is OTU quality by sequence divergence alone (MSD/BLD compactness)
  rather than annotation consistency.
- Input is de novo unassembled reads not yet clustered into OTUs; representative-vs-member
  comparison requires pre-existing OTU clusters.
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0637
tools:
- name: QIIME
  role: Runs the RDP Classifier via its assign_taxonomy step with the Greengenes reference
    training set to annotate both representative and member sequences under identical
    settings
- name: phylo
  role: Repository associated with the article's dissimilarity-metric computations
    (MSD) used to stratify OTUs into high-divergence (MSD >= 0.03) groups
  repo: github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:ec4139de6ffde5db922ff95634265021b82943d2734ebd87511b925ccecc3986
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - QIIME
  - phylo
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# Representative-vs-member taxonomy comparison

## Summary

A QC skill for 16S rRNA OTU analyses that reclassifies every member sequence of each OTU with the RDP Classifier (Greengenes) via QIIME and compares those assignments against the representative sequence's annotation, quantifying how often a single representative's label misrepresents the OTU. It also stratifies the disagreement rate by sequence divergence (MSD >= 0.03) between members and representatives.

## When to use

Use when OTU members were annotated only via a single representative sequence and you need to measure how often member sequences disagree taxonomically, especially for OTUs whose members exceed 3% MSD from the representative.

## When NOT to use

- Sequences already carry per-sequence taxonomic assignments (no representative-inherited annotation is in use, so there is no disagreement to measure).
- The analysis goal is OTU quality by sequence divergence alone (MSD/BLD compactness) rather than annotation consistency.
- Input is de novo unassembled reads not yet clustered into OTUs; representative-vs-member comparison requires pre-existing OTU clusters.

## Inputs

- OTU FASTA files with representative and member sequences per region (e.g., HMP V3V5, V1V5, Lactobacillus V1V3)
- Greengenes reference taxonomy/training set for the RDP Classifier
- Precomputed or computable MSD values between each member and its OTU representative (for high-MSD stratification)

## Outputs

- Per-OTU flag table of member-vs-representative annotation differences (any taxonomic rank)
- annotation_instability.csv with region/stratum/percentage fields (overall vs high-MSD strata per region)

## How to apply

Load OTU representative and member sequences (FASTA) for each 16S region under study (e.g., HMP-derived V3V5, V1V5, and Lactobacillus V1V3 OTUs). Classify representatives with QIIME's assign_taxonomy step using the RDP Classifier and the Greengenes reference training set, then classify every member sequence with the identical configuration. For each OTU, flag it if any member receives an annotation differing from the representative at any rank; compute the overall instability percentage per region. Restrict to high-divergence OTUs (members with MSD >= 0.03 from the representative, where MSD is the article's dissimilarity metric) and recompute; compare against reference rates such as 12.9% (V3V5) and 19.8% (V1V3) overall, rising to 13.5% and 26.2% among high-MSD OTUs. The rationale is that OTU labels are inherited from one representative, so intra-OTU taxonomy disagreement directly biases downstream abundance-by-taxon analyses.

## Related tools

- **QIIME** (Runs the RDP Classifier via its assign_taxonomy step with the Greengenes reference training set to annotate both representative and member sequences under identical settings)
- **phylo** (Repository associated with the article's dissimilarity-metric computations (MSD) used to stratify OTUs into high-divergence (MSD >= 0.03) groups) — github.com/namphuon/phylo

## Examples

```
assign_taxonomy.py -m rdp -i otu_members.fna -r greengenes_rep_set.fna -t greengenes_tax.txt
```

## Evaluation signals

- annotation_instability.csv contains region/stratum/percentage fields for all strata (V3V5, V1V5, Lactobacillus V1V3; overall and high-MSD)
- Computed overall instability matches reported values within rounding tolerance: 12.9% (V3V5) and 19.8% (V1V5)
- High-MSD stratum (MSD >= 0.03) rates match 13.5% (V3V5) and 26.2% (V1V5), and Lactobacillus V1V3 instability is 2.0%
- Representative and member classifications used the identical RDP Classifier / Greengenes configuration, so disagreements reflect sequence differences, not pipeline settings
- Per-OTU flags reconcile with the aggregate percentages (flagged OTUs / total OTUs per stratum equals the reported rate)

## Limitations

- Percent sequence similarity (the basis for OTU clustering, e.g., 97% PSD threshold) can overestimate evolutionary similarity, so poorly clustered OTUs are the ones most likely to show annotation instability.
- OTU compactness varies greatly across the taxonomic tree; both MSD and BLD often exceed the 0.03 threshold within nominally single OTUs, so disagreement rates are region- and clade-dependent.
- The RDP Classifier / Greengenes configuration is a reference-based annotation choice; instability percentages are conditional on that classifier and reference.

## Evidence

- [abstract] As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
- [intro] From the OTU cluster, a single sequence is selected as a representative sequence. The representative sequence is annotated using a 16S classification method: "From the OTU cluster, a single sequence is selected as a representative sequence. The representative sequence is annotated using a 16S classification method"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
