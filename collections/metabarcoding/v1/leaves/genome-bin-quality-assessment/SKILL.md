---
name: genome-bin-quality-assessment
description: Use when shotgun metagenomic contigs (filtered to minimum 1000 bp length)
  need to be grouped into genome bins and their completeness and contamination assessed
  from single-copy marker genes.
when_to_use_negative:
- Input is raw reads rather than assembled contigs — binning requires an assembly
  step (MEGAHIT/SPAdes/IDBA) with contigs filtered to >=1000 bp first
- The analysis is 16S rRNA amplicon profiling (mothur/dada2 workflow), which has no
  assembly or binning stage
- CheckM's marker-gene database has not been installed and configured (checkm data
  setRoot), in which case quality estimates cannot be computed
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: BBMap
  role: Backmaps contigs to their reads to generate sorted BAM and depth files for
    coverage-based binning
- name: MetaBAT
  role: Coverage-based binning of contigs using depth profiles; must be installed
    separately and visible in the system PATH
  repo: https://bitbucket.org/berkeleylab/metabat/src/master/README.md
- name: MaxBin
  role: Second coverage-based binner run alongside MetaBAT before bin refinement
- name: Binning_refiner
  role: Refines/consolidates bins produced by MetaBAT and MaxBin
- name: CheckM
  role: Refines bins and estimates genome completeness and contamination from single-copy
    marker genes
  repo: https://github.com/Ecogenomics/CheckM/wiki/Installation
- name: kraken2
  role: Assigns taxonomy to the refined bins
  repo: https://github.com/DerrickWood/kraken2
- name: Prokka
  role: 'Functional annotation of bins: gene prediction via Prodigal and homology
    search against CDD, PFAM and TIGRAM'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:fbf4042fdd14b09fd97a4159bf72234e3e302991b13ddf42137e5575d8e44069
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBMap
  - Binning_refiner
  - CheckM
  - DIAMOND
  - FastQC
  - HUMAnN2
  - IDBA
  - MEGAHIT
  - MEGAN
  - MaxBin
  - MetaBAT
  - MetaPhlAn2
  - Prokka
  - SPAdes
  - Sickle
  - Trimmomatic
  - kraken2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# genome-bin-quality-assessment

## Summary

Coverage-based reconstruction and quality assessment of metagenome-assembled genomes: contigs (>=1000 bp) from MEGAHIT/SPAdes/IDBA assemblies are backmapped to reads with BBMap, binned with MetaBAT and MaxBin, refined with Binning_refiner and CheckM, and evaluated for completeness/contamination using CheckM's single-copy marker genes.

## When to use

Use when shotgun metagenomic contigs (filtered to minimum 1000 bp length) need to be grouped into genome bins and their completeness and contamination assessed from single-copy marker genes.

## When NOT to use

- Input is raw reads rather than assembled contigs — binning requires an assembly step (MEGAHIT/SPAdes/IDBA) with contigs filtered to >=1000 bp first
- The analysis is 16S rRNA amplicon profiling (mothur/dada2 workflow), which has no assembly or binning stage
- CheckM's marker-gene database has not been installed and configured (checkm data setRoot), in which case quality estimates cannot be computed

## Inputs

- Filtered contigs (minimum 1000 bp) from MEGAHIT/SPAdes/IDBA single-sample or co-assembly
- Paired-end, demultiplexed, gzip-compressed reads (*_1.fastq.gz / *_2.fastq.gz) used to backmap and compute coverage
- CheckM marker gene database (from https://data.ace.uq.edu.au/public/CheckM_databases/)

## Outputs

- Sorted BAM and depth files from BBMap backmapping
- Genome bins from MetaBAT and MaxBin, refined by Binning_refiner
- CheckM completeness and contamination estimates per bin
- Taxonomic assignment of bins (kraken2) and functional annotations (Prokka with CDD, PFAM, TIGRAM)

## How to apply

After assembly, filter contigs to a minimum length of 1000 bp since short contigs cannot be binned reliably. Backmap contigs to their reads with BBMap to produce sorted BAM files and depth/coverage profiles, which are the input for coverage-based binners MetaBAT and MaxBin (MetaBAT must be installed separately and visible in the system PATH). Refine the resulting bins with Binning_refiner and CheckM, then assign bin taxonomy using kraken2 and functionally annotate bins with Prokka (gene prediction via Prodigal; homology search against CDD, PFAM and TIGRAM databases). Install CheckM via checkm-genome and build its database from the downloaded CheckM_databases directory with 'checkm data setRoot', placing checkM under /usr/local/bin. Judge bin quality via CheckM's estimates of completeness and contamination, which are derived from the presence or absence of typically single-copy marker genes.

## Related tools

- **BBMap** (Backmaps contigs to their reads to generate sorted BAM and depth files for coverage-based binning)
- **MetaBAT** (Coverage-based binning of contigs using depth profiles; must be installed separately and visible in the system PATH) — https://bitbucket.org/berkeleylab/metabat/src/master/README.md
- **MaxBin** (Second coverage-based binner run alongside MetaBAT before bin refinement)
- **Binning_refiner** (Refines/consolidates bins produced by MetaBAT and MaxBin)
- **CheckM** (Refines bins and estimates genome completeness and contamination from single-copy marker genes) — https://github.com/Ecogenomics/CheckM/wiki/Installation
- **kraken2** (Assigns taxonomy to the refined bins) — https://github.com/DerrickWood/kraken2
- **Prokka** (Functional annotation of bins: gene prediction via Prodigal and homology search against CDD, PFAM and TIGRAM)

## Examples

```
checkm data setRoot PATH_TO_DOWNLOADED_DATABASE && sh metagenomics_analysis.sh  # run_coverage_bining.sh stage bins MetaBAT/MaxBin output and refines with CheckM
```

## Evaluation signals

- CheckM reports plausible completeness/contamination values for each refined bin, with no bins flagged for excessive contamination
- Binning_refiner consolidates MetaBAT and MaxBin output without losing high-quality bins from either binner
- Every bin receives a kraken2 taxonomy assignment and Prokka gene annotations, confirming the downstream annotation chain ran
- The run_coverage_bining.sh stage of 'sh metagenomics_analysis.sh' completes without failure and stdout status messages confirm each check_and_install-guarded tool (including checkM in /usr/local/bin) was resolved or installed
- Sorted BAM and depth files exist for each contig set, confirming BBMap backmapping succeeded before binning

## Limitations

- CheckM v1 is explicitly unmaintained; CheckM v2 uses machine learning models and is generally more accurate, so running both can be informative since they are independent methodologies for estimating genome quality
- MetaBAT and CheckM cannot be auto-installed by the workflow's check_and_install function — the user must install them beforehand (checkM database via 'checkm data setRoot'; MetaBAT in the system PATH)
- Contigs shorter than 1000 bp are filtered out and therefore cannot be binned; binning quality depends on the upstream assembly and coverage depth, and assemblies are among the most time-consuming, RAM-heavy steps (100 GB RAM recommended)

## Evidence

- [methods] After the stats, assembly filter is performed to filter contigs with minimum 1000bp length: "After the stats, assembly filter is performed to filter contigs with minimum 1000bp length"
- [readme] CheckM marker-gene completeness/contamination methodology: [evidence span withheld — anchor status: external_doc]
- [readme] CheckM v1 vs v2 independence caveat: [evidence span withheld — anchor status: external_doc]
- [readme] CheckM database setup requirement: "After installation the checkM database needs to be built using https://data.ace.uq.edu.au/public/CheckM_databases/ and building by using `checkm data setRoot PATH_TO_DOWNLOADED_DATABASE`"
- [abstract] Binning within the overall best-practice protocol scope: "best-practice protocols on experimental design, sample processing, sequencing, assembly, binning, annotation and visualization"
