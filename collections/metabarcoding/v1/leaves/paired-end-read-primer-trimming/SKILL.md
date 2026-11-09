---
name: paired-end-read-primer-trimming
description: Use when raw paired-end amplicon FASTQs (e.g., dual-indexed 181 bp COI
  reads from SRA PRJNA518082 Libraries A-D) still carry primer sequences and must
  be trimmed before denoising or OTU clustering.
when_to_use_negative:
- Input reads have already been primer-trimmed during demultiplexing or a prior preprocessing
  run — re-trimming risks removing biological bases from the 181 bp COI fragment
- Data are shotgun metagenomic or otherwise non-amplicon reads with no expected primer
  motifs at read ends
- A feature table or denoised ASV artifact is already available — trimming applies
  only to raw read data upstream of DADA2/Deblur/VSEARCH
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: Cutadapt
  role: performs the primer-sequence trimming on demultiplexed paired-end reads inside
    the QIIME 2 environment
- name: QIIME 2
  role: workflow environment (v2018.11) in which reads are imported and Cutadapt trim-paired
    is executed prior to DADA2/Deblur denoising
- name: VSEARCH
  role: downstream consumer of the trimmed reads for merging, dereplication, and clustering
    in the OTU pipeline
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:3c8a779c79f7fe727dbfcc554814b28ccdbc9e8c48390735375e9e04817f0d56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - Cutadapt
  - DADA2
  - Deblur
  - QIIME 2
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# paired-end-read-primer-trimming

## Summary

Removes primer sequences from raw demultiplexed paired-end amplicon reads (e.g., dual-indexed 181 bp COI fragments) with Cutadapt inside a QIIME 2 workflow, so that downstream denoising (DADA2, Deblur) or VSEARCH clustering operates only on biological sequence.

## When to use

Use when raw paired-end amplicon FASTQs (e.g., dual-indexed 181 bp COI reads from SRA PRJNA518082 Libraries A-D) still carry primer sequences and must be trimmed before denoising or OTU clustering.

## When NOT to use

- Input reads have already been primer-trimmed during demultiplexing or a prior preprocessing run — re-trimming risks removing biological bases from the 181 bp COI fragment
- Data are shotgun metagenomic or otherwise non-amplicon reads with no expected primer motifs at read ends
- A feature table or denoised ASV artifact is already available — trimming applies only to raw read data upstream of DADA2/Deblur/VSEARCH

## Inputs

- raw demultiplexed paired-end FASTQ reads (QIIME 2 demux artifact or FASTQ manifest) from mock-community Libraries A-D (SRA PRJNA518082)
- forward and reverse primer sequence motifs used in the dual-indexed 181 bp COI amplification

## Outputs

- primer-trimmed paired-end reads (QIIME 2 artifact, e.g. trimmed-seqs.qza) with unmerged pairs of minimum length 175 bp retained
- per-library read counts before and after primer trimming for downstream filtering decisions

## How to apply

Retrieve the raw paired-end reads (mock community Libraries A-D from NCBI SRA PRJNA518082) and import them into a QIIME 2 v2018.11 environment. Run Cutadapt on the demultiplexed paired-end reads to trim the dual-indexed primer sequences (the wet-lab protocol used a Kozich et al. 2013 dual-indexed primer design to amplify a 181 bp COI fragment, so the forward and reverse primer motifs are known and can be passed as adapter parameters). Retain all unmerged paired-end reads with a minimum length of 175 bp after trimming, discarding shorter fragments. Because trimming is one of the 'common yet often overlooked and undocumented bioinformatics steps' in metabarcoding, record the exact Cutadapt parameters and the per-library read-retention counts in a documented sequence-filtering workflow (as tidybug does in its docs folder) so the effect on downstream ASV/OTU counts can be traced.

## Related tools

- **Cutadapt** (performs the primer-sequence trimming on demultiplexed paired-end reads inside the QIIME 2 environment)
- **QIIME 2** (workflow environment (v2018.11) in which reads are imported and Cutadapt trim-paired is executed prior to DADA2/Deblur denoising)
- **VSEARCH** (downstream consumer of the trimmed reads for merging, dereplication, and clustering in the OTU pipeline) — https://github.com/torognes/vsearch

## Examples

```
qiime cutadapt trim-paired --i-demultiplexed-sequences demux.qza --p-front-f <COI_fwd_primer> --p-front-r <COI_rev_primer> --o-trimmed-sequences trimmed-seqs.qza
```

## Evaluation signals

- No reads in the trimmed output retain the expected primer motifs at their termini (verify by grepping trimmed FASTQs or checking Cutadapt trimming summary counts)
- Only unmerged paired-end reads with length >= 175 bp are retained; per-library retention counts match the minimum-length rule
- The validation target for the downstream mock-community analysis is reachable: DADA2 and Deblur recover all 24 expected mock sequences while the VSEARCH route misses between 1 and 4 of them
- Trimming parameters and read counts are documented in the project's sequence-filtering workflow file (tidybug docs/sequence_filtering.md)
- Trimmed reads are importable into QIIME 2 v2018.11 and denoise without 'expected errors' or length-related failures in DADA2/Deblur

## Limitations

- Trimming choices are among the 'common yet often overlooked and undocumented bioinformatics steps' — failing to record parameters makes the ASV/OTU results irreproducible
- Denoising program and filtering choices downstream strongly influence the number of mock ASVs observed, so trimming quality cannot be judged in isolation from the denoising step
- The minimum-length threshold of 175 bp is specific to the 181 bp COI fragment; other amplicons need their own length cutoffs

## Evidence

- [other] Import reads into a QIIME 2 v2018.11 environment, trim primers with Cutadapt, and retain all unmerged paired-end reads with a minimum length of 175 bp.: [evidence span withheld — anchor status: not_found]
- [methods] Dual-indexed PCR amplification of 181 bp COI fragment: [evidence span withheld — anchor status: not_found]
- [abstract] decisions made in core bioinformatic processes, including sequence filtering, database design, and classification: "decisions made in core bioinformatic processes, including sequence filtering, database design, and classification"
- [readme] tidybug docs list sequence filtering documentation: "[sequence filtering](https://github.com/devonorourke/tidybug/blob/master/docs/sequence_filtering.md)"
