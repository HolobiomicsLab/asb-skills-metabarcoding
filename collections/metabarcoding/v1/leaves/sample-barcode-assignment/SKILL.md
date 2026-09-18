---
name: sample-barcode-assignment
description: Use when simulating multiplexed PacBio HiFi amplicon sequencing data
  and each sample needs a distinct dual-barcode signature embedded in its template
  so downstream demultiplexing and ASV workflows can be benchmarked.
when_to_use_negative:
- Input is already demultiplexed per-sample FASTQ/FASTA with barcodes removed — barcode
  assignment is a pre-sequencing template-construction step, not a demultiplexing
  step
- Simulating short-read (e.g. Illumina) amplicon data with single-index barcoding
  — MHASS implements PacBio HiFi dual-barcoding specifically
- The user only needs read-level error simulation (PBSIM3 subread accuracy/np distribution)
  without template construction
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: MHASS
  role: 'Performs the dual-barcode template generation: assigns unique forward/reverse
    barcode pairs per sample and concatenates A + ForwardBarcode + ForwardPrimer +
    ASV + RC(ReversePrimer) + RC(ReverseBarcode) + A'
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language; Biopython reverse-complement utilities are used to
    build the reverse-complemented reverse primer/barcode segments
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:3b5db9f4f6f187606fed94f8e632dca59f43e1054830dea49810c50124cdb96e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# sample-barcode-assignment

## Summary

Assigns a unique forward/reverse dual-barcode pair to each simulated sample and concatenates them into PacBio HiFi amplicon templates (A + ForwardBarcode + ForwardPrimer + ASV + RC(ReversePrimer) + RC(ReverseBarcode) + A) so every sample carries a distinct template signature, mirroring real PacBio dual-barcoding strategies.

## When to use

Use when simulating multiplexed PacBio HiFi amplicon sequencing data and each sample needs a distinct dual-barcode signature embedded in its template so downstream demultiplexing and ASV workflows can be benchmarked.

## When NOT to use

- Input is already demultiplexed per-sample FASTQ/FASTA with barcodes removed — barcode assignment is a pre-sequencing template-construction step, not a demultiplexing step
- Simulating short-read (e.g. Illumina) amplicon data with single-index barcoding — MHASS implements PacBio HiFi dual-barcoding specifically
- The user only needs read-level error simulation (PBSIM3 subread accuracy/np distribution) without template construction

## Inputs

- Amplicon reference FASTA (e.g. 16S ASV sequences from the ATCC Sequel II 2.0 or Phylotag datasets)
- Primer sequences flanking the ASV insert
- Optional barcode TSV with columns: BarcodeID, ForwardBarcode, ReverseBarcode (e.g. BC01	CACATATCAGAGTGCG	TGGCGTGCATGATTCGA)
- Number of samples to simulate (--num-samples, default 10)

## Outputs

- Per-sample barcoded dual-barcode template sequences (barcoded_templates.fasta / sequence_file_mapping.tsv)
- sample_barcode_map.tsv: mapping of samples to barcode pairs
- counts.tsv: simulated ASV counts per sample keyed by the barcode assignments

## How to apply

Define the dual-barcode template layout used by MHASS: the forward primer and forward barcode are concatenated at the 5' end of the insert and the reverse barcode and reverse primer appear as reverse complements at the 3' end, so a full-length HiFi read spans both barcodes and both primers. Load barcodes from an optional TSV file with columns 'id', 'forward', 'reverse' (e.g. BarcodeID\tForwardBarcode\tReverseBarcode); if none is supplied MHASS assigns barcodes itself. Assign a unique forward/reverse pair per sample, then assemble each template by string concatenation — A, forward barcode, forward primer, ASV, reverse-complemented reverse primer, reverse-complemented reverse barcode, terminal A — using Biopython reverse-complement utilities. Record which sample received which barcode pair (emitted as sample_barcode_map.tsv) so simulated reads can be traced back to their source sample.

## Related tools

- **MHASS** (Performs the dual-barcode template generation: assigns unique forward/reverse barcode pairs per sample and concatenates A + ForwardBarcode + ForwardPrimer + ASV + RC(ReversePrimer) + RC(ReverseBarcode) + A) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language; Biopython reverse-complement utilities are used to build the reverse-complemented reverse primer/barcode segments)

## Examples

```
mhass --amplicon-fasta Test-data_ATCC_16S_MSA-1003/amplicons.fa --amplicon-genome-labels Test-data_ATCC_16S_MSA-1003/amplicon_labels.txt --barcode-file Test-data_ATCC_16S_MSA-1003/barcodes.tsv --output-dir testout
```

## Evaluation signals

- Every sample in sample_barcode_map.tsv maps to a unique forward/reverse barcode pair (no two samples share the same pair)
- Each generated template record contains the expected substring pattern: forward primer + forward barcode at the start and the reverse-complemented reverse primer at the end
- Template length equals the ASV insert plus both primers, both barcodes, and two flanking A's (5' A and terminal A)
- The reverse-complement of the 3' barcode/primer segment, when reversed, recovers the reverse barcode and reverse primer sequences from the barcode TSV
- The script runs end-to-end and the output FASTA exists with one record per sample

## Limitations

- The barcode file is optional; when --barcode-file is not provided, barcode assignment is fully internal to MHASS and cannot be user-specified
- Barcode TSV must use exactly the columns id, forward, reverse; malformed files are not covered in the documentation
- Simulated barcode assignment reflects the simulator's model of dual-barcoding; empirically derived pass-number distributions (np, lognormal mu=3.88, sigma=1.22, np-min 2, np-max 59) affect how often barcodes are observed but are handled downstream

## Evidence

- [methods] In the template-generation step, MHASS concatenates each sample's template as: A, forward barcode, forward primer, ASV, reverse-complemented reverse primer, reverse-complemented reverse barcode, and a terminal A.: [evidence span withheld — anchor status: not_found]
- [methods] Assign a unique forward/reverse barcode pair to each sample so every sample has a distinct template signature.: [evidence span withheld — anchor status: not_found]
- [abstract] realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [readme] --barcode-file BARCODE_FILE TSV file with barcodes (id, forward, reverse) (default: None): "--barcode-file BARCODE_FILE
                        TSV file with barcodes (id, forward, reverse) (default: None)"
- [readme] sample_barcode_map.tsv: Mapping of samples to barcodes: [evidence span withheld — anchor status: external_doc]
