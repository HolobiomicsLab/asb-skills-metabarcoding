---
name: amplicon-template-structure-construction
description: Use when simulating PacBio HiFi amplicon reads that must carry dual barcodes
  and primers flanking each ASV insert, so full-length reads span both barcodes and
  both primers per sample.
when_to_use_negative:
- Input is already a demultiplexed, primer-trimmed reads file (FASTQ) — barcode/primer
  concatenation only applies to template construction before read simulation
- You need single-barcode or unbarcoded amplicon templates rather than PacBio dual-barcode
  library structure
- The target platform is short-read (e.g., Illumina) where this dual-barcode HiFi
  template layout does not apply
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: MHASS
  role: Simulator whose template-generation step performs the dual-barcode template
    concatenation for each sample
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language for the template concatenation script, using Biopython
    reverse-complement utilities
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:4014f8b1bb9321d8d669bd15cb2691452d39f3cc52dcf1af1dd47c6bc65afd7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# amplicon-template-structure-construction

## Summary

Constructs per-sample dual-barcode PacBio HiFi amplicon templates by concatenating A, forward barcode, forward primer, ASV insert, reverse-complemented reverse primer, reverse-complemented reverse barcode, and a terminal A, as implemented in the MHASS simulator's template-generation step.

## When to use

Use when simulating PacBio HiFi amplicon reads that must carry dual barcodes and primers flanking each ASV insert, so full-length reads span both barcodes and both primers per sample.

## When NOT to use

- Input is already a demultiplexed, primer-trimmed reads file (FASTQ) — barcode/primer concatenation only applies to template construction before read simulation
- You need single-barcode or unbarcoded amplicon templates rather than PacBio dual-barcode library structure
- The target platform is short-read (e.g., Illumina) where this dual-barcode HiFi template layout does not apply

## Inputs

- Reference 16S amplicon FASTA sequences (e.g., ATCC Sequel II 2.0 release or Phylotag dataset)
- Per-sample forward/reverse barcode assignments (TSV of barcode id, forward, reverse)
- Forward and reverse primer sequences
- Biopython reverse-complement utilities

## Outputs

- barcoded_templates.fasta containing per-sample dual-barcode template sequences with the layout A+ForwardBarcode+ForwardPrimer+ASV+RC(ReversePrimer)+RC(ReverseBarcode)+A

## How to apply

Load reference 16S amplicon sequences (e.g., from the PacBio DevNet Sequel II 2.0 ATCC release or the Phylotag dataset) and assign each sample a unique forward/reverse barcode pair so every sample has a distinct template signature. For each sample, assemble the full template by string concatenation in the order: A, forward barcode, forward primer, ASV, reverse-complemented reverse primer, reverse-complemented reverse barcode, terminal A, using Biopython reverse-complement utilities for the 3' adapters. Write the per-sample sequences to barcoded_templates.fasta. This layout mirrors real PacBio dual-barcode library structure, so a full-length HiFi read spans both barcodes and both primers, enabling realistic downstream demultiplexing and primer-trimming in simulated workflows.

## Related tools

- **MHASS** (Simulator whose template-generation step performs the dual-barcode template concatenation for each sample) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language for the template concatenation script, using Biopython reverse-complement utilities)

## Examples

```
mhass --amplicon-fasta amplicons.fa --amplicon-genome-labels labels.tsv --barcode-file barcodes.tsv --output-dir testout
```

## Evaluation signals

- The script runs end-to-end without errors and barcoded_templates.fasta exists
- Each FASTA record begins with the expected substring pattern: forward primer followed by forward barcode
- Each record ends with the reverse-complemented reverse primer followed by the reverse-complemented reverse barcode and terminal A
- Each sample's template carries its unique forward/reverse barcode pair, giving a distinct template signature per sample
- Reverse-complement regions verify correctly when reverse-complemented back to primer/barcode sequences

## Limitations

- The template layout assumes dual-barcode PacBio library structure; other library chemistries or single-index designs would need a modified concatenation order
- Barcode file is optional in MHASS (--barcode-file); without it, no per-sample barcode concatenation signature exists
- Only reverse-complementing the 3' adapters is correct for this orientation; mistakenly reverse-complementing the 5' side breaks the expected substring validation

## Evidence

- [other] In the template-generation step, MHASS concatenates each sample's template as: A, forward barcode, forward primer, ASV, reverse-complemented reverse primer, reverse-complemented reverse barcode, and a terminal A.: [evidence span withheld — anchor status: not_found]
- [abstract] MHASS creates realistic synthetic PacBio HiFi amplicon sequencing datasets ... by integrating genome-aware abundance modeling, realistic dual-barcoding strategies: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [other] Assign a unique forward/reverse barcode pair to each sample so every sample has a distinct template signature.: [evidence span withheld — anchor status: not_found]
- [other] Validation: verify the script runs end-to-end, the output FASTA exists, and each record contains the expected substring pattern: [evidence span withheld — anchor status: not_found]
- [readme] --barcode-file: TSV file with barcodes (id, forward, reverse): "--barcode-file BARCODE_FILE
                        TSV file with barcodes (id, forward, reverse) (default: None)"
