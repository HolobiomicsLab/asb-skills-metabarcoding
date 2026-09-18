---
name: reverse-complement-sequence-manipulation
description: Use when constructing barcoded sequencing templates or adapters where
  one primer/barcode arm must appear reverse-complemented at the opposite end of the
  insert (e.g. PacBio dual-barcoded amplicon simulation).
when_to_use_negative:
- Input sequences are already full-length barcoded templates — reverse-complementing
  again would corrupt the layout
- The downstream protocol uses single (non-dual) barcoding with no reverse-complemented
  3' adapter arm, so the A+barcode+primer+ASV+RC(primer)+RC(barcode)+A layout does
  not apply
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3174
tools:
- name: MHASS
  role: Simulator whose template-generation step concatenates forward barcode/primer
    at the 5' end and reverse-complemented reverse barcode/primer at the 3' end of
    each ASV
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language; Biopython reverse-complement utilities compute the
    reverse complements used in template concatenation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:bc110c942a5989a7b0e35940cfeb9d60331d03dca40e32a579cd023678d78a61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 12
---

# Reverse-Complement Sequence Manipulation

## Summary

Applies reverse-complement transformation to oligonucleotide sequences (barcodes, primers) so that dual-barcode PacBio HiFi amplicon templates faithfully reproduce the SMRTbell library layout, as implemented in MHASS's template-generation step.

## When to use

Use when constructing barcoded sequencing templates or adapters where one primer/barcode arm must appear reverse-complemented at the opposite end of the insert (e.g. PacBio dual-barcoded amplicon simulation).

## When NOT to use

- Input sequences are already full-length barcoded templates — reverse-complementing again would corrupt the layout
- The downstream protocol uses single (non-dual) barcoding with no reverse-complemented 3' adapter arm, so the A+barcode+primer+ASV+RC(primer)+RC(barcode)+A layout does not apply

## Inputs

- Reference 16S amplicon sequences (ASVs) in FASTA from the ATCC (PacBio DevNet Sequel II 2.0 release) and Phylotag datasets
- Barcode TSV with columns id, forward, reverse (e.g. BarcodeID\tForwardBarcode\tReverseBarcode)
- Forward and reverse primer sequences for the amplicon locus

## Outputs

- barcoded_templates.fasta containing per-sample dual-barcoded template sequences in the layout A+ForwardBarcode+ForwardPrimer+ASV+RC(ReversePrimer)+RC(ReverseBarcode)+A
- sequence_file_mapping.tsv with detailed template mappings (downstream MHASS output)

## How to apply

In MHASS's template-generation step, each sample's full-length template is assembled by string concatenation as: A + forward barcode + forward primer + ASV insert + reverse-complement(reverse primer) + reverse-complement(reverse barcode) + terminal A, so a full-length HiFi read spans both barcodes and both primers. Reverse complements are computed with Biopython reverse-complement utilities applied to the reverse primer and reverse barcode sequences drawn from a barcode TSV (columns: BarcodeID, ForwardBarcode, ReverseBarcode, e.g. BC01 CACATATCAGAGTGCG TGGCGTGCATGATTCGA). Each sample receives a unique forward/reverse barcode pair so every template has a distinct signature. The assembled per-sample sequences are written to barcoded_templates.fasta and validated by checking for the expected substring pattern (forward primer + forward barcode at the start, reverse-complemented reverse primer at the end).

## Related tools

- **MHASS** (Simulator whose template-generation step concatenates forward barcode/primer at the 5' end and reverse-complemented reverse barcode/primer at the 3' end of each ASV) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language; Biopython reverse-complement utilities compute the reverse complements used in template concatenation)

## Examples

```
mhass --amplicon-fasta Test-data_ATCC_16S_MSA-1003/amplicons.fa --amplicon-genome-labels Test-data_ATCC_16S_MSA-1003/amplicon_labels.txt --barcode-file Test-data_ATCC_16S_MSA-1003/barcodes.tsv --np-distribution-type lognormal --output-dir testout
```

## Evaluation signals

- barcoded_templates.fasta exists after the script runs end-to-end
- Each FASTA record begins with the forward primer + forward barcode substring
- Each FASTA record ends with the reverse-complemented reverse primer substring (followed by the reverse-complemented reverse barcode and terminal A in the full layout)
- Reverse-complementing the 3' arm of a template recovers the original reverse barcode and reverse primer sequences from the barcode TSV
- Each sample's template carries a unique forward/reverse barcode pair (distinct template signature per sample)

## Limitations

- The layout is specific to PacBio HiFi dual-barcoded amplicon libraries; error and pass-number characteristics (PBSIM3 subread accuracy 0.65, difference ratios per platform) are simulated separately and are not part of this manipulation
- Requires a well-formed barcode TSV (id, forward, reverse) and matching primer sequences; mismatched primer/barcode sequences will silently produce templates that fail the substring validation

## Evidence

- [methods] MHASS concatenates each sample's template as: A, forward barcode, forward primer, ASV, reverse-complemented reverse primer, reverse-complemented reverse barcode, and a terminal A.: [evidence span withheld — anchor status: not_found]
- [methods] assemble the full template sequence by string concatenation of primer, barcode, insert, reverse-complemented reverse barcode, and reverse-complemented reverse primer, using Biopython reverse-complement utilities: [evidence span withheld — anchor status: not_found]
- [abstract] integrate genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [readme] BarcodeID	ForwardBarcode	ReverseBarcode
BC01	CACATATCAGAGTGCG	TGGCGTGCATGATTCGA: [evidence span withheld — anchor status: not_found]
