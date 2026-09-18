---
name: fasta-sequence-output-generation
description: Use when simulating PacBio HiFi amplicon data requires constructing full-length
  barcoded template sequences (adapters + barcodes + primers + ASV) and writing them
  to a FASTA file for downstream read simulation.
when_to_use_negative:
- Input is already barcoded amplicon reads or a FASTQ of sequenced data rather than
  raw ASV reference sequences needing template construction
- The target is a short-read or single-barcode (non-dual-barcoded) amplicon protocol
  where the A+ForwardBarcode+ForwardPrimer+ASV+RC(ReversePrimer)+RC(ReverseBarcode)+A
  layout does not apply
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: MHASS
  role: Pipeline whose template-generation step concatenates A, forward barcode, forward
    primer, ASV, reverse-complemented reverse primer, and reverse-complemented reverse
    barcode, writing barcoded_templates.fasta
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language; uses Biopython reverse-complement utilities and string
    concatenation to assemble template sequences
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:3c56513e33d8f2be818f5ac5700755f22e5447096944f44712b6bf2376563677
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# fasta-sequence-output-generation

## Summary

Generates per-sample dual-barcode PacBio template sequences as a FASTA file by concatenating adapters, barcodes, primers, and ASV inserts (with reverse complements), mirroring the template layout MHASS uses to simulate realistic PacBio HiFi amplicon reads.

## When to use

Use when simulating PacBio HiFi amplicon data requires constructing full-length barcoded template sequences (adapters + barcodes + primers + ASV) and writing them to a FASTA file for downstream read simulation.

## When NOT to use

- Input is already barcoded amplicon reads or a FASTQ of sequenced data rather than raw ASV reference sequences needing template construction
- The target is a short-read or single-barcode (non-dual-barcoded) amplicon protocol where the A+ForwardBarcode+ForwardPrimer+ASV+RC(ReversePrimer)+RC(ReverseBarcode)+A layout does not apply

## Inputs

- Amplicon (ASV) reference sequences in FASTA format (e.g. amplicons.fa from ATCC or Phylotag datasets)
- Barcode TSV file with columns: BarcodeID, ForwardBarcode, ReverseBarcode (e.g. BC01	CACATATCAGAGTGCG	TGGCGTGCATGATTCGA)
- Forward and reverse primer sequences
- Sample-to-barcode assignment mapping

## Outputs

- barcoded_templates.fasta containing per-sample dual-barcode template sequences
- Per-sample template mappings (e.g. sequence_file_mapping.tsv) documenting barcode and primer layout per record

## How to apply

Load the reference 16S amplicon (ASV) sequences — e.g. from the ATCC (PacBio DevNet Sequel II 2.0 release) or Phylotag datasets — and assign each sample a unique forward/reverse barcode pair (from a TSV with columns id, forward, reverse). Assemble each sample's template by string concatenation in the MHASS dual-barcode layout: 5' A adapter, forward barcode, forward primer, the ASV insert, then the reverse-complemented reverse primer and reverse-complemented reverse barcode at the 3' end, terminated by a final A, using Biopython reverse-complement utilities so a full-length HiFi read spans both barcodes and both primers. Write the per-sample sequences to barcoded_templates.fasta and feed them into the simulation (e.g. PBSIM3 with subread accuracy 0.65 and an empirical or lognormal number-of-passes distribution). Validate that the FASTA exists and each record starts with the expected forward-primer + forward-barcode pattern and ends with the reverse-complemented reverse primer.

## Related tools

- **MHASS** (Pipeline whose template-generation step concatenates A, forward barcode, forward primer, ASV, reverse-complemented reverse primer, and reverse-complemented reverse barcode, writing barcoded_templates.fasta) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language; uses Biopython reverse-complement utilities and string concatenation to assemble template sequences)

## Examples

```
mhass --amplicon-fasta Test-data_ATCC_16S_MSA-1003/amplicons.fa --amplicon-genome-labels Test-data_ATCC_16S_MSA-1003/amplicon_labels.txt --barcode-file Test-data_ATCC_16S_MSA-1003/barcodes.tsv --np-distribution-type lognormal --output-dir testout
```

## Evaluation signals

- barcoded_templates.fasta exists after the script runs end-to-end
- Each FASTA record begins with the expected forward primer + forward barcode substring
- Each FASTA record ends with the reverse-complemented reverse primer substring (preceded by the reverse-complemented reverse barcode)
- Every sample has a distinct forward/reverse barcode pair, so each template carries a unique sample signature
- Template length equals sum of components: two A adapters + both barcodes + both primers + the ASV insert length

## Limitations

- Requires an optional --barcode-file TSV; without it the default MHASS behavior (no user-supplied barcodes) differs from the dual-barcode layout described here
- Template layout assumes full-length HiFi reads spanning both barcodes and both primers; truncated or partial reads will not contain the complete expected pattern

## Evidence

- [methods] MHASS concatenates each sample's template as: A, forward barcode, forward primer, ASV, reverse-complemented reverse primer, reverse-complemented reverse barcode, and a terminal A.: [evidence span withheld — anchor status: not_found]
- [abstract] realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [readme] TSV file of barcodes with columns: id, forward, reverse.: [evidence span withheld — anchor status: not_found]
- [readme] BarcodeID	ForwardBarcode	ReverseBarcode: "BarcodeID	ForwardBarcode	ReverseBarcode
BC01	CACATATCAGAGTGCG	TGGCGTGCATGATTCGA"
