---
name: coi-metabarcoding-primer-alignment
description: 'Use when benchmarking a CO1 classifier with metabarcoding-like queries:
  you need primer binding regions located on the CO1 barcoding region, or full-length
  500 bp+ training sequences converted to primer-anchored 200 bp query fragments.'
when_to_use_negative:
- Query data are already real primer-trimmed amplicon reads — classify them directly
  rather than re-simulating fragments from full-length training sequences.
- 'Non-CO1 markers such as 16S rDNA (SILVA/Greengenes) or fungal ITS (UNITE): the
  Table S14 CO1 primer set and the X03240 D. yakuba reference do not apply.'
- Input sequences are shorter than 500 bp or contain nucleotide ambiguities — outside
  the screened training-set definition this simulation assumes.
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Mesquite v3.10
  role: Aligns the compiled CO1 primers (Table S14) against the Drosophila yakuba
    CO1 region (GenBank accession X03240) to determine primer binding regions
- name: CUTADAPT v1.10
  role: Primer-trims the 500 bp+ CO1 training set FASTA with default settings and
    up to a 10% primer mismatch, retrieving primer-anchored sequences the way real
    raw metabarcoding reads would be processed
- name: RDP classifier v2.12
  role: Downstream consumer of the primer-anchored 200 bp fragments for leave-one-out
    and cross-validation benchmarking against the CO1 Eukaryote v1 training set
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:8872668094108e4b3ba459fa66cc699af8f7930cb4afa17f51cf556079ad3756
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CUTADAPT
  - CUTADAPT v1.10
  - Mesquite
  - Mesquite v3.10
  - RDP classifier v2.12
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# coi-metabarcoding-primer-alignment

## Summary

Maps CO1 metabarcoding primers (Table S14) onto the CO1 barcoding region via Mesquite alignment against the Drosophila yakuba CO1 region (GenBank X03240), then simulates real amplicon reads by CUTADAPT primer-trimming of the 500 bp+ CO1 training set and truncating each hit to a 200 bp primer-anchored fragment for RDP classifier benchmarking.

## When to use

Use when benchmarking a CO1 classifier with metabarcoding-like queries: you need primer binding regions located on the CO1 barcoding region, or full-length 500 bp+ training sequences converted to primer-anchored 200 bp query fragments.

## When NOT to use

- Query data are already real primer-trimmed amplicon reads — classify them directly rather than re-simulating fragments from full-length training sequences.
- Non-CO1 markers such as 16S rDNA (SILVA/Greengenes) or fungal ITS (UNITE): the Table S14 CO1 primer set and the X03240 D. yakuba reference do not apply.
- Input sequences are shorter than 500 bp or contain nucleotide ambiguities — outside the screened training-set definition this simulation assumes.

## Inputs

- CO1 primer list compiled from the literature (Table S14), with primer sequences and references
- Drosophila yakuba CO1 region reference sequence (GenBank accession X03240); Bos taurus CO1 secondary structure (UniProt P00396) as reference context
- CO1 training set FASTA: full-length 500 bp+ screened sequences from the CO1Classifier repository training files (CO1 Eukaryote v1)

## Outputs

- Map of primer binding regions along the CO1 barcoding region for every compiled primer
- primer_trimmed_200bp.fasta: 200 bp primer-anchored query fragments retrieved with no more than 10% primer-sequence mismatch
- Per-primer retrieval counts showing coverage of each anchoring position in the training set

## How to apply

Compile the CO1 primers from the literature (Table S14) and align each against the Drosophila yakuba CO1 region (GenBank accession X03240) in Mesquite v3.10, using the Bos taurus CO1 secondary structure (UniProt P00396) as reference context, to determine each primer's binding region. Load the CO1 training set FASTA of full-length 500 bp+ screened sequences (the CO1 Eukaryote v1 training files from the CO1Classifier repository). Run CUTADAPT v1.10 with default settings, allowing up to a 10% mismatch in the primer sequence, so training sequences are retrieved exactly as real raw metabarcoding reads would be processed. Truncate each retrieved primer-anchored sequence to a 200 bp fragment — the average length of an Illumina read after primer trimming — and write the set to primer_trimmed_200bp.fasta. Expect position-dependent coverage: primers designed near the 5′ and 3′ ends of the barcoding region, including the Folmer barcoding primers, retrieve especially few fragments because primers are often trimmed before sequences are submitted to GenBank. Use the fragments for leave-one-out and cross-validation testing with the RDP classifier v2.12, applying the 60% bootstrap support cutoff defined for primer-anchored 200 bp sequences at the genus rank.

## Related tools

- **Mesquite v3.10** (Aligns the compiled CO1 primers (Table S14) against the Drosophila yakuba CO1 region (GenBank accession X03240) to determine primer binding regions)
- **CUTADAPT v1.10** (Primer-trims the 500 bp+ CO1 training set FASTA with default settings and up to a 10% primer mismatch, retrieving primer-anchored sequences the way real raw metabarcoding reads would be processed)
- **RDP classifier v2.12** (Downstream consumer of the primer-anchored 200 bp fragments for leave-one-out and cross-validation benchmarking against the CO1 Eukaryote v1 training set)

## Examples

```
cutadapt -g <PRIMER_FROM_TABLE_S14> -e 0.1 --length 200 -o primer_trimmed_200bp.fasta co1_training.fasta
```

## Evaluation signals

- All records in primer_trimmed_200bp.fasta are exactly 200 bp and were retrieved with no more than 10% primer-sequence mismatch.
- Primers binding near the 5′ and 3′ ends of the barcoding region (e.g., the Folmer primers) return markedly fewer fragments than mid-region primers, matching the expected GenBank primer-trimming artifact.
- Retrieved fragments derive only from screened training sequences of at least 500 bp without nucleotide ambiguities, consistent with the CO1 Eukaryote v1 training set definition.
- Downstream RDP classifier v2.12 benchmarking of the fragments reproduces the 60% bootstrap support cutoff behavior reported for primer-anchored 200 bp sequences at the genus rank.

## Limitations

- Fragments anchored at the 5′ and 3′ ends of the barcoding region are under-represented because primers are often trimmed before sequences are submitted to GenBank, biasing classifier benchmarking at those anchoring positions.
- The 200 bp truncation simulates only the average length of an Illumina read after primer trimming; other read lengths require separately trimmed and truncated fragment sets.
- Species-rank assignment is not achievable for these fragments: the training-set cutoff tables list NA at the species rank for 99% correct assignments, so summarize assignments to the genus or more inclusive ranks.

## Evidence

- [methods] Primers are aligned to the D. yakuba CO1 region with Mesquite to locate binding regions: [evidence span withheld — anchor status: not_found]
- [methods] CUTADAPT primer-trimming parameters for the training set: [evidence span withheld — anchor status: not_found]
- [methods] 200 bp truncation mimics average Illumina read length: [evidence span withheld — anchor status: not_found]
- [results] Low training-set coverage of primers near the 5′/3′ ends of the barcoding region: [evidence span withheld — anchor status: not_found]
- [results] Primer-anchored 200 bp fragments benchmarked with a 60% bootstrap cutoff: [evidence span withheld — anchor status: not_found]
- [readme] Training-set sequences are 500 bp+ and contaminant-screened: "All sequences here are at least 500bp long and have been screened to remove human and bacterial contaminant sequences."
