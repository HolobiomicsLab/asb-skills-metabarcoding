---
name: qiime2-amplicon-pipeline-execution
description: Use when raw 16S FASTQ data must be converted into an ASV feature table
  in QIIME2, especially when downstream alpha diversity metrics (e.g. Faith's PD,
  ACE) require singletons that DEBLUR preserves but DADA2 discards.
when_to_use_negative:
- Input is already a denoised ASV feature table — re-denoising is unnecessary; proceed
  directly to QIIME2's diversity module.
- The analysis goal does not require singletons and singleton removal is acceptable
  (e.g. DADA2-based workflows), since the DEBLUR-vs-DADA2 choice matters only for
  singleton-dependent metrics.
- The project lacks associated metadata — the authors excluded such projects from
  downstream analyses.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: sra-tools
  role: Downloads each dataset's raw 16S sequencing files from NCBI SRA using accession
    numbers from each dataset's folder
  repo: https://github.com/ncbi/sra-tools
- name: QIIME2 2022.2.0
  role: Analysis environment for import, per-dataset QC, DEBLUR denoising, and alpha
    diversity calculation via the diversity module
  repo: https://qiime2.org/
- name: DEBLUR
  role: Denoiser that performs noise removal, dereplication, chimera removal, and
    ASV selection while keeping singletons; chosen over DADA2 for the feature table
- name: anaconda
  role: Virtual environment platform on which the QIIME2 suite was installed and run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:6ae6179ccd6a380a9cc3ea92a7817a155ccbc57c726f9b6bb6db01dc796a8a18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncbi/sra-tools
  tools:
  - DEBLUR
  - QIIME2
  - QIIME2 2022.2.0
  - anaconda
  - sra-tools
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2222222222222222
n_claims: 9
---

# qiime2-amplicon-pipeline-execution

## Summary

Execute the paper's QIIME2-based 16S amplicon processing pipeline: download SRA runs with sra-tools, import reads into QIIME2 2022.2.0, run per-dataset QC, and denoise with DEBLUR (keeping singletons) to produce a non-rarefied feature table suitable for alpha diversity calculation.

## When to use

Use when raw 16S FASTQ data must be converted into an ASV feature table in QIIME2, especially when downstream alpha diversity metrics (e.g. Faith's PD, ACE) require singletons that DEBLUR preserves but DADA2 discards.

## When NOT to use

- Input is already a denoised ASV feature table — re-denoising is unnecessary; proceed directly to QIIME2's diversity module.
- The analysis goal does not require singletons and singleton removal is acceptable (e.g. DADA2-based workflows), since the DEBLUR-vs-DADA2 choice matters only for singleton-dependent metrics.
- The project lacks associated metadata — the authors excluded such projects from downstream analyses.

## Inputs

- 16S raw sequencing files (FASTQ) downloaded from NCBI SRA via sra-tools, with accession numbers from each dataset folder's SraRunTable.txt
- Sample metadata for each project (required for downstream aggregation of alpha diversity metrics)
- QIIME2-imported demultiplexed reads (qza artifact)

## Outputs

- DEBLUR-denoised feature table exported as feature-table.deblur.qza (non-rarefied ASV table with singletons retained)
- Per-dataset alpha diversity metrics aggregated into alpha-out_dataset-name.tsv, later combined with metadata into alpha_aggregated.tsv

## How to apply

Retrieve the raw 16S sequencing files for each project from NCBI SRA using sra-tools, taking accession numbers from each dataset's folder in the repository (./datasets/xx_dataset_alias/SraRunTable.txt or similar). Install QIIME2 2022.2.0 on an anaconda virtual environment and follow QIIME2 best practices; the template commands ran by the authors are in pipeline_scripts/workflow.txt. Perform per-dataset QC independently (read quality assessment, adapter trimming) and remove paired-end samples that only had a single read before denoising. Apply DEBLUR to the QC'd reads to perform noise removal, dereplication, chimera removal, and ASV selection, deliberately choosing DEBLUR over DADA2 because DEBLUR keeps singletons, which some alpha diversity metrics require. Export the denoised feature table (feature-table.deblur.qza) and use it non-rarefied as input to QIIME2's diversity module for alpha diversity calculations.

## Related tools

- **sra-tools** (Downloads each dataset's raw 16S sequencing files from NCBI SRA using accession numbers from each dataset's folder) — https://github.com/ncbi/sra-tools
- **QIIME2 2022.2.0** (Analysis environment for import, per-dataset QC, DEBLUR denoising, and alpha diversity calculation via the diversity module) — https://qiime2.org/
- **DEBLUR** (Denoiser that performs noise removal, dereplication, chimera removal, and ASV selection while keeping singletons; chosen over DADA2 for the feature table)
- **anaconda** (Virtual environment platform on which the QIIME2 suite was installed and run)

## Evaluation signals

- feature-table.deblur.qza exists and is a valid QIIME2 artifact (importable via qiime tools peek) containing ASVs with singletons retained
- Denoised feature table is carried forward non-rarefied into QIIME2's diversity module, matching the paper's 'alpha diversity metrics were calculated with non-rarefied data' approach
- Per-dataset QC was done independently (read quality, adapter trimming) and paired-end samples with only a single read were removed before denoising
- Aggregated per-dataset output file alpha-out_dataset-name.tsv is produced and combined with metadata into alpha_aggregated.tsv
- The choice of DEBLUR over DADA2 is documented, since DADA2 removes singletons and DEBLUR keeps them — a reported source of variability in estimated richness if the denoise tool is not reported

## Limitations

- Raw sample data is not shared in the repository due to size; only accession numbers are provided within each dataset's folder
- Workflows were run on QIIME 2022.2.0 and should be compatible with newer versions, but newer-version behavior is not validated by the paper
- The denoise tool is a source of variability for estimated richness; DADA2 vs DEBLUR differences (singleton treatment) must be reported, and results differ between them

## Evidence

- [readme] All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment.: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [readme] Raw sample data not shared; accession numbers per dataset folder: "Raw sample data is not shared through this repository due to their size, however the accession numbers are available within each dataset's folder"
- [readme] Workflows ran on QIIME 2022.2.0 with template commands in pipeline_scripts: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
- [results] All samples processed with both DADA2 and DEBLUR: "all samples were processed using DADA2 and DEBLUR"
- [discussion] Singleton treatment difference between DADA2 and DEBLUR: "DADA2 removes singletons and DEBLUR keeps them."
- [discussion] Per-dataset QC before denoising: "After performing QC for each dataset independently (read quality, adapter trimming)"
- [discussion] Paired-end samples with a single read removed: "Samples identified as paired-end that only had a single read were removed."
