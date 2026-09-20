---
name: taxonomic-composition-data-extraction
description: Use when a publication reports 16S rRNA taxonomic profiling results (relative
  abundances, OTU/species/phylum counts) and these figures must be converted into
  a structured composition table with provenance.
when_to_use_negative:
- Input is already a computed feature/OTU table with taxonomy columns — extraction
  from prose is unnecessary
- The target community is eukaryotic ITS-based classification (Warcup V2 'Classify
  Sequences' at 90% similarity, 75 bp minimum overlap), not 16S/SILVA-based prokaryotic
  composition
- Absolute abundance or strain-level resolution is required; the source only reports
  phylum-level relative percentages and OTU counts with some OTUs left unclassified
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: SILVAngs
  role: Online automatic pipeline performing the 16S read classification whose composition
    results are extracted
- name: SINA v1.2.10
  role: Aligns each read against the SILVA SSU rRNA within the SILVAngs classification
    step
- name: cd-hit-est v3.1.2
  role: Clusters dereplicated reads into OTUs at identity criteria 1.00 and 0.98 ignoring
    overhangs
- name: BLASTn
  role: Local search assigning OTUs against the non-redundant SILVA SSU ref dataset
    (release no. 132)
- name: Sickle
  role: Sliding-window quality trimming of FASTQ reads prior to classification, when
    re-deriving the composition from raw reads
  repo: https://github.com/najoshi/sickle
- name: Scythe
  role: Bayesian 3'-end adapter trimming applied before quality-based trimming in
    the read-preprocessing chain
  repo: http://github.com/vsbuffalo/scythe
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:d458b90bdf78d615f9c4e6afd77f82819dca439f433f00ea56628319c6b47742
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/najoshi/sickle
  tools:
  - BLASTn
  - SILVAngs
  - SINA
  - SINA v1.2.10
  - Scythe
  - Sickle
  - cd-hit-est
  - cd-hit-est v3.1.2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# taxonomic-composition-data-extraction

## Summary

Extracts and tabulates domain- and phylum-level prokaryotic composition from 16S metabarcoding results, as demonstrated by SILVAngs classification of trout farm water reads yielding >99% bacterial OTUs dominated by Proteobacteria (75.57%) and Bacteroidetes (14.40%).

## When to use

Use when a publication reports 16S rRNA taxonomic profiling results (relative abundances, OTU/species/phylum counts) and these figures must be converted into a structured composition table with provenance.

## When NOT to use

- Input is already a computed feature/OTU table with taxonomy columns — extraction from prose is unnecessary
- The target community is eukaryotic ITS-based classification (Warcup V2 'Classify Sequences' at 90% similarity, 75 bp minimum overlap), not 16S/SILVA-based prokaryotic composition
- Absolute abundance or strain-level resolution is required; the source only reports phylum-level relative percentages and OTU counts with some OTUs left unclassified

## Inputs

- Published article text (e.g. DOI 10.1038/s41598-020-80236-x) reporting 16S phylum-level relative abundances
- SILVAngs classification parameters (SINA v1.2.10, cd-hit-est v3.1.2, BLASTn vs SILVA SSU ref release 132)
- Raw demultiplexed FASTQ reads of the 16S V3-V4 amplicon (if re-running classification rather than extracting reported figures)

## Outputs

- prokaryotic_composition.csv with columns phylum and relative_abundance_percent
- Summary fields: total bacterial species (1318), total phyla (17), fraction of prokaryotic OTUs assigned to Bacteria (>99%)

## How to apply

Locate the taxonomic assignment section of the source article and record the full classification chain: SILVAngs pipeline with SINA v1.2.10 alignment against SILVA SSU rRNA, cd-hit-est v3.1.2 OTU clustering at identity criteria 1.00 and 0.98 ignoring overhangs, and BLASTn assignment against the non-redundant SILVA SSU ref dataset (release no. 132). Extract the reported phylum-level relative abundances (Proteobacteria 75.57%, Bacteroidetes 14.40%, Actinobacteria 0.94%, Verrucomicrobia 0.62%, Cyanobacteria 0.25%) and summary statistics (1318 bacterial species across 17 phyla; >99% of prokaryotic OTUs assigned to Bacteria). Write a structured table (e.g. prokaryotic_composition.csv) with columns phylum and relative_abundance_percent, plus summary fields for total species, total phyla, and bacterial OTU fraction. Note that reads whose '(% sequence identity + % alignment coverage)/2' did not exceed 93 remained unclassified or unassigned, so tabulated abundances cover classified OTUs only.

## Related tools

- **SILVAngs** (Online automatic pipeline performing the 16S read classification whose composition results are extracted)
- **SINA v1.2.10** (Aligns each read against the SILVA SSU rRNA within the SILVAngs classification step)
- **cd-hit-est v3.1.2** (Clusters dereplicated reads into OTUs at identity criteria 1.00 and 0.98 ignoring overhangs)
- **BLASTn** (Local search assigning OTUs against the non-redundant SILVA SSU ref dataset (release no. 132))
- **Sickle** (Sliding-window quality trimming of FASTQ reads prior to classification, when re-deriving the composition from raw reads) — https://github.com/najoshi/sickle
- **Scythe** (Bayesian 3'-end adapter trimming applied before quality-based trimming in the read-preprocessing chain) — http://github.com/vsbuffalo/scythe

## Examples

```
python scripts/build_composition.py --doi 10.1038/s41598-020-80236-x --phyla Proteobacteria=75.57,Bacteroidetes=14.40,Actinobacteria=0.94,Verrucomicrobia=0.62,Cyanobacteria=0.25 --species 1318 --phyla-count 17 --bacterial-otu-frac 0.99 --out prokaryotic_composition.csv
```

## Evaluation signals

- CSV parses with all required fields present (phylum, relative_abundance_percent, total species, total phyla, bacterial OTU fraction)
- Proteobacteria value equals ~75.57% and Bacteroidetes ~14.4% as reported in the abstract
- Total bacterial species count equals 1318 and total phyla count equals 17
- More than 99% of prokaryotic OTUs are classified as Bacteria in the summary fields
- Every tabulated relative-abundance value matches a figure verbatim from the source article (no rounding drift)

## Limitations

- Reads with no or weak BLAST hits where '(% sequence identity + % alignment coverage)/2' did not exceed 93 remained unclassified or unassigned, so composition covers only classified OTUs
- Exact farm locations were withheld at farmers' request, limiting geographic reproducibility of the composition profile
- Extracted figures are pipeline-version dependent (SILVA SSU ref release no. 132, cd-hit-est v3.1.2, SINA v1.2.10); reclassification with newer releases may shift phylum abundances

## Evidence

- [abstract] Proteobacteria dominance and Bacteroidetes figures with bacterial OTU fraction: "more than 99% of the detected prokaryotic OTUs (Operational Taxonomic Unit identification) were bacteria as of ~ 75.57% were Proteobacteria, and ~ 14.4% were Bacteroidetes"
- [discussion] SILVAngs classification chain with SINA and SILVA database: "Each read was aligned against the SILVA SSU rRNA using the SILVA Incremental Aligner (SINA v1.2.10;51)"
- [discussion] OTU clustering parameters: "clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2;52)"
- [discussion] BLASTn assignment database and release: "based on local BLASTn search against the non-redundant version of SILVA SSU ref dataset (release no. 132)"
- [discussion] Unclassified OTU threshold: "did not exceed the value of 93 remained unclassified or unassigned"
- [readme] Scythe role in the read-preprocessing chain upstream of classification: "it is recommended you run Scythe *before* quality-based trimming, as part of a read quality control pipeline."
