---
name: its-region-extraction-with-itsx
description: Use when processing fungal ITS amplicon data (e.g. ITS2 reads amplified
  with fITS7/R-ITS4 primers) and you need to excise the ITS subregion from inferred
  ASVs and filter for fungal sequences before taxonomy assignment.
when_to_use_negative:
- Input is 16S bacterial or archaeal amplicon data — dadasnake routes those to SILVA-based
  classification, not ITS region extraction.
- Input is already a feature/ASV table without nucleotide sequences — ITSx needs the
  ASV FASTA to detect and extract subregions.
- Sequences are full-length genomes or long contigs rather than short ITS amplicons
  within the expected length range.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3316
tools:
- name: ITSx
  role: Detects ITS subregions in ASV sequences with e-value 1.00E-05, extracts the
    ITS2 region, and filters for fungal sequences (target taxon F, minimum 1 detected
    region).
- name: dadasnake
  role: Snakemake workflow that orchestrates the ITSx step between DADA2 denoising
    and UNITE/mothur classification within a single conda environment.
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: Upstream producer of the exact sequence variants (ASVs) that ITSx extracts
    ITS2 regions from.
- name: BASTA
  role: Summarizes BLAST hits for the ITS2-extracted, classified ASVs.
  repo: https://github.com/timkahlke/BASTA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:02da67de5740b0628f57640af003efa736e77248caa9e9779e65b7eb1f9a2b69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - BASTA
  - BLAST
  - DADA2
  - ITSx
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# its-region-extraction-with-itsx

## Summary

Extracts the ITS2 subregion from ASV sequences and retains only fungal sequences using ITSx, as implemented in the dadasnake pipeline between DADA2 denoising and taxonomic classification. This subregion extraction improves downstream UNITE-based assignment by removing flanking SSU/LSU sequence and non-fungal ASVs.

## When to use

Use when processing fungal ITS amplicon data (e.g. ITS2 reads amplified with fITS7/R-ITS4 primers) and you need to excise the ITS subregion from inferred ASVs and filter for fungal sequences before taxonomy assignment.

## When NOT to use

- Input is 16S bacterial or archaeal amplicon data — dadasnake routes those to SILVA-based classification, not ITS region extraction.
- Input is already a feature/ASV table without nucleotide sequences — ITSx needs the ASV FASTA to detect and extract subregions.
- Sequences are full-length genomes or long contigs rather than short ITS amplicons within the expected length range.

## Inputs

- FASTA file of DADA2-inferred ASV sequences from ITS amplicon runs (post chimera removal)
- Raw paired-end ITS2 fastq files processed upstream via dadasnake (YAML config + tab-separated sample table with library, r1_file, r2_file columns)

## Outputs

- ITS2-extracted ASV sequences restricted to fungal sequences (>=1 detected ITS region, target taxon F)
- Filtered ASV table feeding into UNITE v8 / mothur classification and final biom/FASTA/tab-separated hand-off formats

## How to apply

After DADA2 quality filtering, error estimation/denoising and chimera removal produce the ASV set, run ITSx on the ASV sequences with an e-value of 1.00E-05, restricting the target taxon to fungi (T=F) and requiring a minimum of 1 detected region. Extract the ITS2 region specifically from each ASV that passes detection. Feed the filtered, ITS2-extracted sequences into taxonomic classification (in dadasnake: mothur Bayesian classifier against UNITE v8 with cutoff 60, summarized with BASTA). Validate the whole workflow on the even 19-species fungal mock community (PRJNA626434) by confirming exactly 19 fungal ASVs with expected intra-genus strain diversity (2 Aspergillus, 3 Fusarium).

## Related tools

- **ITSx** (Detects ITS subregions in ASV sequences with e-value 1.00E-05, extracts the ITS2 region, and filters for fungal sequences (target taxon F, minimum 1 detected region).)
- **dadasnake** (Snakemake workflow that orchestrates the ITSx step between DADA2 denoising and UNITE/mothur classification within a single conda environment.) — https://github.com/a-h-b/dadasnake
- **DADA2** (Upstream producer of the exact sequence variants (ASVs) that ITSx extracts ITS2 regions from.)
- **BASTA** (Summarizes BLAST hits for the ITS2-extracted, classified ASVs.) — https://github.com/timkahlke/BASTA

## Examples

```
ITSx -i asvs.fasta -o itsx_out -E 1e-05 -T F --min_regions 1 --save_regions ITS2
```

## Evaluation signals

- On the even 19-species fungal mock community (PRJNA626434), the pipeline recovers exactly 19 fungal ASVs after ITS2 extraction and fungal filtering.
- Intra-genus strain diversity is preserved: 2 Aspergillus strains and 3 Fusarium strains are resolved (micro-diversity correctly identified).
- Only ASVs with at least 1 ITS region detected and target taxon F survive the filter; non-fungal and non-ITS ASVs are absent from the final table.
- Extracted sequences correspond to the ITS2 subregion only, suitable for classification against UNITE v8 with the mothur Bayesian classifier (cutoff 60).
- Configuration validated by a dadasnake dryrun (-d) before the real run completes without errors.

## Limitations

- One of the three Fusarium strains was misclassified in the fungal mock community despite correct micro-diversity resolution.
- The fungal mock community showed a known overestimation of Rhizophagus irregularis diversity.
- ITSx settings shown (e-value 1.00E-05, target taxon F, minimum 1 region) are tuned for fungal ITS; other taxa would require different target-taxa settings.

## Evidence

- [other] Run ITSx on the ASVs with an e-value of 1.00E-05, extracting the ITS2 region and filtering for fungal sequences (target taxon F, minimum 1 detected region).: [evidence span withheld — anchor status: not_found]
- [other] Classify the filtered ASVs against the UNITE v8 database using the mothur Bayesian classifier (cutoff 60) and summarize BLAST hits with BASTA.: [evidence span withheld — anchor status: not_found]
- [other] Micro-diversity was correctly identified for 2 strains of Aspergillus and the 3 Fusarium strains (although 1 was misclassified) in the fungal mock community dataset.: [evidence span withheld — anchor status: not_found]
- [intro] process amplicon sequencing data, from raw fastq-files to taxonomically assigned OTU tables: "process amplicon sequencing data, from raw fastq-files to taxonomically assigned "OTU" tables"
- [methods] chimera removal - using DADA2: "chimera removal - using DADA2"
- [abstract] The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi.: "The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi."
