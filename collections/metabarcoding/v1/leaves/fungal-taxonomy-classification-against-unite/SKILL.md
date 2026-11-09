---
name: fungal-taxonomy-classification-against-unite
description: Use when ASVs derived from fungal ITS2 amplicon sequencing (e.g., after
  DADA2 denoising and ITSx filtering) need genus/species-level taxonomic assignment,
  as in mock community validation or microbial ecology studies of fungi.
when_to_use_negative:
- Input is bacterial or archaeal 16S amplicon data — dadasnake suggests the SILVA
  database for 16S rather than UNITE, which targets ITS
- Input is raw FASTQ reads rather than denoised ASVs — classification happens after
  DADA2 error estimation, denoising, chimera removal, and ITSx extraction of the ITS2
  region
- Non-fungal targets are of interest — the ITSx filtering step restricts to fungal
  sequences (target taxon F)
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: UNITE
  role: Reference database for taxonomic classification of fungal ITS ASVs
  repo: https://doi.org//10.15156/BIO/786336
- name: mothur
  role: Bayesian classifier (cutoff 60) assigning taxonomy to the ITSx-filtered fungal
    ASVs against UNITE v8
- name: BLAST
  role: Aligns ASVs against reference sequences to produce hits summarized for taxonomy
- name: BASTA
  role: Summarizes BLAST hits, inferring taxonomy via Last Common Ancestor of best
    hits (e-value, percent identity, alignment length)
  repo: https://github.com/timkahlke/BASTA
- name: ITSx
  role: Extracts the ITS2 region from ASVs and filters for fungal sequences prior
    to classification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:44f90d8411beeb498e36bdbb9cd07357d3e95ba653469aad09caac5c8fe15997
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BASTA
  - BLAST
  - ITSx
  - UNITE
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# fungal-taxonomy-classification-against-unite

## Summary

Assigns taxonomy to fungal ITS amplicon ASVs by classifying the ITSx-filtered fungal sequences against the UNITE reference database with the mothur Bayesian classifier, then summarizing BLAST hits with BASTA. This is the fungal branch of dadasnake's taxonomic classification step, demonstrated on an even 19-species fungal mock community.

## When to use

Use when ASVs derived from fungal ITS2 amplicon sequencing (e.g., after DADA2 denoising and ITSx filtering) need genus/species-level taxonomic assignment, as in mock community validation or microbial ecology studies of fungi.

## When NOT to use

- Input is bacterial or archaeal 16S amplicon data — dadasnake suggests the SILVA database for 16S rather than UNITE, which targets ITS
- Input is raw FASTQ reads rather than denoised ASVs — classification happens after DADA2 error estimation, denoising, chimera removal, and ITSx extraction of the ITS2 region
- Non-fungal targets are of interest — the ITSx filtering step restricts to fungal sequences (target taxon F)

## Inputs

- Fungal ITS2 ASV sequences (FASTA) output by ITSx filtering (target taxon F, min 1 region, e-value 1.00E-05)
- UNITE v8 reference database for ITS taxonomy (dadasnake supports UNITE DOI 10.15156/BIO/786336)
- NCBI taxonomy-backed tabular BLAST output (blast -outfmt 6 style) for BASTA LCA summarization

## Outputs

- Taxonomy table assigning genus/species-level labels to each fungal ASV (mothur Bayesian classification, cutoff 60)
- BASTA summary of BLAST hits (7-level NCBI taxonomy per sequence)
- Final ASV and taxonomy tables in standard hand-off formats (biom, FASTA, tab-separated tables, phyloseq objects)

## How to apply

After DADA2 inference and chimera removal, run ITSx on the ASV sequences with an e-value of 1.00E-05, extracting the ITS2 region and filtering for fungal sequences (target taxon F, minimum 1 detected region). Classify the filtered fungal ASVs against the UNITE v8 reference database (dadasnake suggests UNITE for ITS data, as opposed to SILVA for 16S) using the mothur Bayesian classifier with a cutoff of 60. In parallel, BLAST the ASVs and summarize the hits with BASTA, which infers taxonomy from the Last Common Ancestor of best hits based on e-value, percent identity, and alignment length. Validate the result against the known community composition: the even 19-species fungal mock community should yield exactly 19 fungal ASVs with the expected intra-genus strain diversity (2 Aspergillus, 3 Fusarium), noting the known overestimation of Rhizophagus irregularis diversity.

## Related tools

- **UNITE** (Reference database for taxonomic classification of fungal ITS ASVs) — https://doi.org//10.15156/BIO/786336
- **mothur** (Bayesian classifier (cutoff 60) assigning taxonomy to the ITSx-filtered fungal ASVs against UNITE v8)
- **BLAST** (Aligns ASVs against reference sequences to produce hits summarized for taxonomy)
- **BASTA** (Summarizes BLAST hits, inferring taxonomy via Last Common Ancestor of best hits (e-value, percent identity, alignment length)) — https://github.com/timkahlke/BASTA
- **ITSx** (Extracts the ITS2 region from ASVs and filters for fungal sequences prior to classification)

## Evaluation signals

- The even 19-species fungal mock community yields exactly 19 fungal ASVs after ITSx filtering and classification
- Intra-genus strain diversity matches expectation: 2 Aspergillus strains and 3 Fusarium strains are resolved
- Known systematic behavior is reproduced: Rhizophagus irregularis diversity is overestimated
- BASTA LCA taxonomy summaries are consistent with the mothur Bayesian classifier assignments against UNITE v8
- One known error case is tolerated: 1 of the 3 Fusarium strains was misclassified in the reference run

## Limitations

- One of the 3 Fusarium strains in the mock community was misclassified, showing that closely related strains may not always resolve correctly
- Diversity of Rhizophagus irregularis is known to be overestimated by this pipeline, a documented caveat for AMF-containing samples
- UNITE is appropriate for ITS data only; 16S data should use SILVA instead

## Evidence

- [full_text] Classify the filtered ASVs against the UNITE v8 database using the mothur Bayesian classifier (cutoff 60) and summarize BLAST hits with BASTA.: [evidence span withheld — anchor status: not_found]
- [methods] and [UNITE](https://doi.org//10.15156/BIO/786336) for ITS: "and [UNITE](https://doi.org//10.15156/BIO/786336) for ITS"
- [full_text] Run ITSx on the ASVs with an e-value of 1.00E-05, extracting the ITS2 region and filtering for fungal sequences (target taxon F, minimum 1 detected region).: [evidence span withheld — anchor status: not_found]
- [readme] BASTA assigns taxonomies to sequences or groups of sequences based on the Last Common Ancestor (LCA) of a number of best hits.: [evidence span withheld — anchor status: not_found]
- [readme] Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts.: "Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts."
