---
name: silva-taxonomic-classification-interpretation
description: Use when 16S amplicon reads need phylum/domain-level taxonomic assignment
  via the SILVA rRNA database, or when interpreting reported SILVAngs OTU classifications
  (e.g. %Proteobacteria, unclassified fractions).
when_to_use_negative:
- Input is already a taxonomically assigned feature/OTU table — only interpretation
  or re-analysis is needed, not re-classification
- Classifying ITS/eukaryotic reads — this article used a separate 'Classify Sequences'
  tool with the Warcup V2 training data set at 90% similarity and 75 bp minimum overlap,
  not SILVAngs
- Full-length rRNA sequences or isolate genomes where direct SINA/SILVA alignment
  outside an OTU-clustering pipeline is more appropriate
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: SILVAngs
  role: Online automatic pipeline that performs the 16S OTU clustering and SILVA-based
    taxonomic classification
- name: SINA v1.2.10
  role: SILVA Incremental Aligner aligning each read against the SILVA SSU rRNA
- name: cd-hit-est v3.1.2
  role: Clusters dereplicated reads into OTUs at identity criteria 1.00 and 0.98,
    ignoring overhangs
- name: BLASTn
  role: Local search assigning OTU taxonomy against the non-redundant SILVA SSU ref
    dataset (release no. 132)
- name: BBduk
  role: 'Pre-classification trimming: Q30 trim and discard reads below 50 bp in Geneious
    Prime'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:53f8bcac1f2343d4dc947bf071e44f52cabdb488a88c324d5172466463b90ddd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBduk
  - BLASTn
  - SILVAngs
  - SINA
  - SINA v1.2.10
  - cd-hit-est
  - cd-hit-est v3.1.2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# silva-taxonomic-classification-interpretation

## Summary

Interpret and reproduce SILVA-based taxonomic classification of 16S rRNA metabarcoding reads, understanding how SILVAngs assigns OTUs to phyla via SINA alignment, cd-hit-est clustering, and BLASTn against the SILVA SSU ref database, including the thresholds that leave reads unclassified.

## When to use

Use when 16S amplicon reads need phylum/domain-level taxonomic assignment via the SILVA rRNA database, or when interpreting reported SILVAngs OTU classifications (e.g. %Proteobacteria, unclassified fractions).

## When NOT to use

- Input is already a taxonomically assigned feature/OTU table — only interpretation or re-analysis is needed, not re-classification
- Classifying ITS/eukaryotic reads — this article used a separate 'Classify Sequences' tool with the Warcup V2 training data set at 90% similarity and 75 bp minimum overlap, not SILVAngs
- Full-length rRNA sequences or isolate genomes where direct SINA/SILVA alignment outside an OTU-clustering pipeline is more appropriate

## Inputs

- Quality-trimmed, merged FASTQ reads (e.g. 16S V3-V4 amplicons from MiSeq, Q30-trimmed, reads below 50 bp discarded)
- SILVA SSU rRNA reference database (SILVAngs pipeline, default settings)
- Non-redundant SILVA SSU ref dataset (release no. 132) for BLASTn assignment

## Outputs

- Phylum-level relative abundance table (e.g. prokaryotic_composition.csv with columns phylum, relative_abundance_percent)
- OTU table with taxonomic assignments, including counts of unclassified/unassigned reads
- Summary statistics: total bacterial species (1318), total phyla (17), fraction of prokaryotic OTUs assigned to Bacteria (>99%)

## How to apply

Run compressed FASTQ reads through SILVAngs, which aligns each read against the SILVA SSU rRNA using SINA v1.2.10, then clusters reads into OTUs with cd-hit-est v3.1.2 applying identity criteria of 1.00 and 0.98 while ignoring overhangs. Taxonomy is assigned via local BLASTn against the non-redundant SILVA SSU ref dataset (release no. 132); reads where the function '(% sequence identity + % alignment coverage)/2' does not exceed 93 remain unclassified or unassigned, so report them as such rather than forcing an assignment. Discard short reads and reads with more than 2% ambiguities or homopolymers before dereplication of identical reads. Summarize composition at phylum level as relative abundances (in this trout-farm dataset: Proteobacteria 75.57%, Bacteroidetes 14.40%, Actinobacteria 0.94%, Verrucomicrobia 0.62%, Cyanobacteria 0.25%) and track the fraction of prokaryotic OTUs assignable to Bacteria.

## Related tools

- **SILVAngs** (Online automatic pipeline that performs the 16S OTU clustering and SILVA-based taxonomic classification)
- **SINA v1.2.10** (SILVA Incremental Aligner aligning each read against the SILVA SSU rRNA)
- **cd-hit-est v3.1.2** (Clusters dereplicated reads into OTUs at identity criteria 1.00 and 0.98, ignoring overhangs)
- **BLASTn** (Local search assigning OTU taxonomy against the non-redundant SILVA SSU ref dataset (release no. 132))
- **BBduk** (Pre-classification trimming: Q30 trim and discard reads below 50 bp in Geneious Prime)

## Examples

```
blastn -query otu_rep_seqs.fasta -db SILVA_132_SSURefNr99 -outfmt "6 staxids pident qlen slen" -out silva_assignments.tsv  # assign OTUs, then flag hits with (pident + coverage)/2 <= 93 as unclassified
```

## Evaluation signals

- More than 99% of detected prokaryotic OTUs classify as Bacteria (domain-level sanity check)
- Dominant phylum is Proteobacteria at ~75.57% relative abundance, second Bacteroidetes at ~14.4%
- Reads with (% sequence identity + % alignment coverage)/2 ≤ 93 are correctly reported as unclassified/unassigned rather than force-assigned
- Output table parses with all required fields (phylum, relative_abundance_percent) and every value matches the reported figures (1318 species across 17 phyla)
- Classification is reproducible only if SILVA release no. 132 and the stated identity criteria (1.00/0.98) are used

## Limitations

- Some OTUs remained unclassified or unassigned due to no or weak BLAST hits — classification is not exhaustive
- Exact farm locations were withheld at farmers' request, limiting geographic reproducibility of the classification-based diversity conclusions
- Classification depends on the SILVA reference release (no. 132); results are not directly comparable to runs against newer SILVA versions or other classifiers

## Evidence

- [discussion] SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline based on the SILVA rDNA database: "SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline based on the SILVA rDNA database"
- [discussion] Each read was aligned against the SILVA SSU rRNA using the SILVA Incremental Aligner (SINA v1.2.10;51): "Each read was aligned against the SILVA SSU rRNA using the SILVA Incremental Aligner (SINA v1.2.10;51)"
- [discussion] based on local BLASTn search against the non-redundant version of SILVA SSU ref dataset (release no. 132): "based on local BLASTn search against the non-redundant version of SILVA SSU ref dataset (release no. 132)"
- [discussion] Samples with no or weak BLAST hits where the function "(% sequence identity + % alignment coverage)/2" did not exceed the value of 93 remained unclassified: "Samples with no or weak BLAST hits where the function "(% sequence identity + % alignment coverage)/2" did not exceed the value of 93 remained unclassified"
- [abstract] more than 99% of the detected prokaryotic OTUs (Operational Taxonomic Unit identification) were bacteria as of ~ 75.57% were Proteobacteria: "more than 99% of the detected prokaryotic OTUs (Operational Taxonomic Unit identification) were bacteria as of ~ 75.57% were Proteobacteria"
