---
name: dual-barcode-sequence-design
description: Use when simulating or preparing PacBio HiFi amplicon reads that must
  carry sample-identifying dual barcodes inside a single full-length read, e.g. building
  barcoded_templates.fasta from ASV sequences, primer sequences, and a barcode TSV.
when_to_use_negative:
- Input is already a FASTQ of sequencing reads — barcodes are then demultiplexed,
  not concatenated into templates
- The platform reads only one end of the molecule (e.g. single-indexed short-read
  libraries), where the dual-barcode 5'/3' layout does not apply
- You need to simulate only ASV abundance profiles without sequence-level templates
  (MHASS counts.tsv output), since no template concatenation is required
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: MHASS
  role: Simulator whose template-generation step performs the dual-barcode concatenation
    (A + forward barcode + forward primer + ASV + RC reverse primer + RC reverse barcode
    + A)
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language; Biopython reverse-complement utilities used to build
    the reverse-complemented 3' barcode/primer segments
- name: MHASS_evaluation
  role: Evaluation scripts and data (ATCC and Phylotag test datasets) used to exercise
    the template-generation step
  repo: https://github.com/rhowardstone/MHASS_evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:7cfb315d87eb47e67d136488f8c659eedfe340da5642d3d20254575d81fab5f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - MHASS_evaluation
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dual-barcode-sequence-design

## Summary

Constructs PacBio HiFi amplicon sequencing templates in which each sample is tagged with a unique forward/reverse barcode pair arranged in the dual-barcoded layout A + forward barcode + forward primer + ASV + RC(reverse primer) + RC(reverse barcode) + A, so full-length HiFi reads span both barcodes and both primers. This mirrors the template-concatenation step used by MHASS to create realistic synthetic microbiome HiFi datasets.

## When to use

Use when simulating or preparing PacBio HiFi amplicon reads that must carry sample-identifying dual barcodes inside a single full-length read, e.g. building barcoded_templates.fasta from ASV sequences, primer sequences, and a barcode TSV.

## When NOT to use

- Input is already a FASTQ of sequencing reads — barcodes are then demultiplexed, not concatenated into templates
- The platform reads only one end of the molecule (e.g. single-indexed short-read libraries), where the dual-barcode 5'/3' layout does not apply
- You need to simulate only ASV abundance profiles without sequence-level templates (MHASS counts.tsv output), since no template concatenation is required

## Inputs

- FASTA file of reference ASV/amplicon sequences (amplicons.fa)
- TSV barcode file with columns id, forward barcode, reverse barcode (e.g. BarcodeID\tForwardBarcode\tReverseBarcode)
- Forward and reverse primer sequences
- Sample-to-barcode assignments

## Outputs

- barcoded_templates.fasta containing per-sample dual-barcoded template sequences
- Per-sample mapping of samples to barcodes (sample_barcode_map.tsv / sequence_file_mapping.tsv in MHASS output)

## How to apply

Load reference 16S amplicon (ASV) sequences (e.g. from the ATCC PacBio DevNet Sequel II 2.0 release or Phylotag datasets) and a barcode file assigning a unique forward/reverse barcode pair per sample. Define the template layout so the forward primer and forward barcode sit at the 5' end of the insert and the reverse barcode and reverse primer appear reverse-complemented at the 3' end, since full-length HiFi reads span both barcodes and both primers. Assemble each sample's template by string concatenation: A + forward barcode + forward primer + ASV + reverse-complement(reverse primer) + reverse-complement(reverse barcode) + A, using Biopython reverse-complement utilities. Write the per-sample barcoded templates to barcoded_templates.fasta for downstream number-of-passes and PBSIM3 simulation.

## Related tools

- **MHASS** (Simulator whose template-generation step performs the dual-barcode concatenation (A + forward barcode + forward primer + ASV + RC reverse primer + RC reverse barcode + A)) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language; Biopython reverse-complement utilities used to build the reverse-complemented 3' barcode/primer segments)
- **MHASS_evaluation** (Evaluation scripts and data (ATCC and Phylotag test datasets) used to exercise the template-generation step) — https://github.com/rhowardstone/MHASS_evaluation

## Examples

```
mhass --amplicon-fasta Test-data_ATCC_16S_MSA-1003/amplicons.fa --amplicon-genome-labels Test-data_ATCC_16S_MSA-1003/amplicon_labels.txt --barcode-file Test-data_ATCC_16S_MSA-1003/barcodes.tsv --output-dir testout
```

## Evaluation signals

- Script runs end-to-end and barcoded_templates.fasta exists
- Every FASTA record starts with the expected substring pattern (forward primer + forward barcode)
- Every FASTA record ends with the reverse-complemented reverse primer (then RC reverse barcode, then terminal A)
- Each sample maps to a distinct forward/reverse barcode pair (no duplicate template signatures across samples)
- Reverse-complemented segments match Biopython seq.reverse_complement() of the original primer/barcode sequences

## Limitations

- Barcode file is optional in MHASS (--barcode-file); without it, barcoding must come from elsewhere or templates lack sample tags
- Barcode sequences must be supplied as TSV with id, forward, and reverse columns; malformed TSV rows will break concatenation
- Validation relies on substring checks (forward primer + barcode at start, RC reverse primer at end); internal ordering errors within the template may not be caught

## Evidence

- [abstract] by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [readme] barcode-file optional TSV format: "--barcode-file BARCODE_FILE
                        TSV file with barcodes (id, forward, reverse) (default: None)"
- [readme] barcode file example columns: "BarcodeID	ForwardBarcode	ReverseBarcode
BC01	CACATATCAGAGTGCG	TGGCGTGCATGATTCGA"
- [readme] sequence_file_mapping output: [evidence span withheld — anchor status: external_doc]
- [supplementary] task_006 finding on template layout: [evidence span withheld — anchor status: not_found]
