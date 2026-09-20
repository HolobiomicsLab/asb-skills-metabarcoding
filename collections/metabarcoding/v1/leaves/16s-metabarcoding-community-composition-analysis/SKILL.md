---
name: 16s-metabarcoding-community-composition-analysis
description: Use when paired-end 16S rRNA amplicon FASTQ data (e.g. Illumina MiSeq
  V3–V4) from environmental water samples must be summarized as OTUs and phylum-level
  relative abundances to answer 'what is the prokaryotic composition?'
when_to_use_negative:
- Input is already a taxonomic/feature table rather than raw 16S FASTQ reads — the
  SILVAngs pipeline would be redundant
- The goal is eukaryotic/fungal diversity from ITS2 amplicons — those require consensus
  assembly and the 'Classify Sequences' tool with the Warcup V2 training set, not
  the 16S/SILVAngs workflow
- Strain-level resolution or functional profiling is needed — OTU clustering at 0.98/1.00
  identity against SILVA SSU ref does not provide whole-genome resolution (the authors
  propose WGS-metagenomics for that)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SILVAngs
  role: Online automatic pipeline that performs quality filtering, dereplication,
    OTU clustering and taxonomic classification of the compressed 16S FASTQ files
    based on the SILVA rDNA database
- name: SINA v1.2.10
  role: SILVA Incremental Aligner used inside SILVAngs to align each read against
    the SILVA SSU rRNA alignment
- name: cd-hit-est v3.1.2
  role: Clusters reads and assigns them as OTUs at identity criteria of 1.00 and 0.98
    ignoring overhangs
- name: BLASTn
  role: Local search for taxonomic assignment of OTUs against the non-redundant SILVA
    SSU ref dataset (release no. 132); hits with (% identity + % coverage)/2 ≤ 93
    are left unclassified
- name: BBmerge
  role: Pairs and merges the paired-end FASTQ files at a normal merge rate prior to
    SILVAngs
- name: BBduk
  role: Trims reads for low quality (Q 30) and discards short reads (below 50 bp)
    in Geneious Prime
- name: Scythe v0.994
  role: Bayesian 3'-end adapter trimmer run before quality-based trimming; sequences
    below the length threshold are discarded
  repo: http://github.com/vsbuffalo/scythe
- name: Sickle
  role: Windowed adaptive quality trimming of FASTQ reads after adapter removal
  repo: https://github.com/najoshi/sickle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:02019946860ae306048ba219f56268d7897fc88db4b399728f50373c8cb74480
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBduk
  - BBmerge
  - BLASTn
  - SILVAngs
  - SINA
  - SINA v1.2.10
  - Scythe v0.994
  - Sickle
  - cd-hit-est
  - cd-hit-est v3.1.2
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# 16s-metabarcoding-community-composition-analysis

## Summary

Turns raw 16S rRNA amplicon FASTQ reads (here, V3–V4 region from trout farm inflow- and pond-water) into a phylum-level prokaryotic composition profile via the SILVAngs pipeline (SINA alignment, cd-hit-est OTU clustering, BLASTn against SILVA SSU ref). Used to quantify which bacterial taxa and relative abundances dominate a metaviromic/metabarcoding water sample.

## When to use

Use when paired-end 16S rRNA amplicon FASTQ data (e.g. Illumina MiSeq V3–V4) from environmental water samples must be summarized as OTUs and phylum-level relative abundances to answer 'what is the prokaryotic composition?'

## When NOT to use

- Input is already a taxonomic/feature table rather than raw 16S FASTQ reads — the SILVAngs pipeline would be redundant
- The goal is eukaryotic/fungal diversity from ITS2 amplicons — those require consensus assembly and the 'Classify Sequences' tool with the Warcup V2 training set, not the 16S/SILVAngs workflow
- Strain-level resolution or functional profiling is needed — OTU clustering at 0.98/1.00 identity against SILVA SSU ref does not provide whole-genome resolution (the authors propose WGS-metagenomics for that)

## Inputs

- Paired-end 16S rRNA V3–V4 amplicon FASTQ files (compressed) from water samples
- Illumina adapter sequences for Scythe trimming
- SILVA SSU ref non-redundant reference dataset (release no. 132) used by SILVAngs/BLASTn

## Outputs

- Phylum-level relative abundance table (e.g. prokaryotic_composition.csv with columns phylum, relative_abundance_percent)
- Summary statistics: total bacterial species (1318), total phyla (17), fraction of prokaryotic OTUs assigned to Bacteria (>99%)
- SILVAngs OTU-level classification report per sample

## How to apply

Sequence paired-end FASTQ with MiSeq 3000 (MiSeq Control Software v2.2, bcl2fastq v1.8.4), then merge pairs with BBmerge and trim to Q30 with BBduk in Geneious Prime, discarding reads below 50 bp and removing adapters with Scythe (run before quality trimming, as it is quality-aware for 3'-end adapters) and Sickle. Submit the compressed FASTQ to the SILVAngs online pipeline: reads with more than 2% ambiguities or homopolymers are discarded, identical reads are dereplicated, each read is aligned against SILVA SSU rRNA with SINA v1.2.10, and reads are clustered and assigned as OTUs with cd-hit-est v3.1.2 applying identity criteria of 1.00 and 0.98 ignoring overhangs. Taxonomy is assigned by local BLASTn against the non-redundant SILVA SSU ref dataset (release no. 132); reads whose (% sequence identity + % alignment coverage)/2 does not exceed 93 remain unclassified. The output is a taxonomic profile (e.g. Proteobacteria 75.57%, Bacteroidetes 14.40%, Actinobacteria 0.94%, Verrucomicrobia 0.62%, Cyanobacteria 0.25% across 1318 bacterial species in 17 phyla) tabulated as a structured CSV with phylum and relative_abundance_percent columns plus summary fields.

## Related tools

- **SILVAngs** (Online automatic pipeline that performs quality filtering, dereplication, OTU clustering and taxonomic classification of the compressed 16S FASTQ files based on the SILVA rDNA database)
- **SINA v1.2.10** (SILVA Incremental Aligner used inside SILVAngs to align each read against the SILVA SSU rRNA alignment)
- **cd-hit-est v3.1.2** (Clusters reads and assigns them as OTUs at identity criteria of 1.00 and 0.98 ignoring overhangs)
- **BLASTn** (Local search for taxonomic assignment of OTUs against the non-redundant SILVA SSU ref dataset (release no. 132); hits with (% identity + % coverage)/2 ≤ 93 are left unclassified)
- **BBmerge** (Pairs and merges the paired-end FASTQ files at a normal merge rate prior to SILVAngs)
- **BBduk** (Trims reads for low quality (Q 30) and discards short reads (below 50 bp) in Geneious Prime)
- **Scythe v0.994** (Bayesian 3'-end adapter trimmer run before quality-based trimming; sequences below the length threshold are discarded) — http://github.com/vsbuffalo/scythe
- **Sickle** (Windowed adaptive quality trimming of FASTQ reads after adapter removal) — https://github.com/najoshi/sickle

## Examples

```
scythe -a adapter_file.fasta -o trimmed_sequences.fastq sequences.fastq
```

## Evaluation signals

- CSV parses with all required fields present and every tabulated value matches the reported figures: >99% of prokaryotic OTUs classified as Bacteria, ~75.57% Proteobacteria, ~14.4% Bacteroidetes
- Phylum table reconciles with summary statistics: 1318 detected bacterial species across 17 phyla
- QC tractability check: reads passing filters should satisfy the SILVAngs filter (<2% ambiguities or homopolymers) and length minimums (≥50 bp after BBduk)
- Classification completeness sanity check: only reads with (% sequence identity + % alignment coverage)/2 ≤ 93 should appear as unclassified/unassigned
- Merged-read rate from BBmerge should be at a 'normal merge rate' before proceeding to SILVAngs

## Limitations

- Some OTUs remain unclassified or unassigned due to no or weak BLAST hits (score threshold 93), so phylum fractions are conditional on classified reads
- Exact farm locations were withheld at farmers' request, limiting geographic reproducibility of the community profile
- Scythe only trims 3'-end contaminants and should not be used for 5'-trimming; it must run before quality-based trimming so low-quality bases that identify adapters are not removed first
- The correlation between detected bacteria, feed type, and volatile compounds cannot currently be stated from this composition profile alone

## Evidence

- [abstract] more than 99% of the detected prokaryotic OTUs (Operational Taxonomic Unit identification) were bacteria as of ~ 75.57% were Proteobacteria, and ~ 14.4% were Bacteroidetes: "more than 99% of the detected prokaryotic OTUs (Operational Taxonomic Unit identification) were bacteria as of ~ 75.57% were Proteobacteria, and ~ 14.4% were Bacteroidetes"
- [discussion] SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline based on the SILVA rDNA database: "SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline based on the SILVA rDNA database"
- [discussion] Each read was aligned against the SILVA SSU rRNA using the SILVA Incremental Aligner (SINA v1.2.10: "Each read was aligned against the SILVA SSU rRNA using the SILVA Incremental Aligner (SINA v1.2.10"
- [discussion] clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2: "clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2"
- [discussion] Short reads and/or with more than 2% ambiguities or homopolymers were discarded: "Short reads and/or with more than 2% ambiguities or homopolymers were discarded"
- [readme] it is recommended you run Scythe *before* quality-based trimming, as part of a read quality control pipeline.: "it is recommended you run Scythe *before* quality-based trimming, as part of a read quality control pipeline."
