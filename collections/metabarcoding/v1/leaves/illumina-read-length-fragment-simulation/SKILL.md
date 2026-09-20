---
name: illumina-read-length-fragment-simulation
description: 'Use when benchmarking CO1 taxonomic classifiers on short metabarcoding
  reads: a full-length 500 bp+ training FASTA is available and primer-anchored 200
  bp query fragments mimicking Illumina reads after primer trimming are needed.'
when_to_use_negative:
- Query data are already short, primer-trimmed metabarcoding reads — classify them
  directly instead of simulating fragments from reference sequences.
- The benchmark targets full-length (500 bp+) classification, where truncating references
  to 200 bp would test the wrong length regime.
- Species-rank accuracy is the goal — no 99% bootstrap cutoff exists for 200 bp species-rank
  assignments (species row is NA), so genus-rank or above must be used.
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CUTADAPT
  role: Retrieves primer-trimmed sequences from the full-length 500 bp+ CO1 training
    FASTA using literature CO1 primers with up to 10% primer-sequence mismatch, mimicking
    real raw metabarcoding read processing
- name: Mesquite
  role: Aligns compiled CO1 primers (Table S14) against the Drosophila yakuba CO1
    region (GenBank X03240) to determine primer binding regions before trimming
- name: RDP classifier
  role: Downstream classifier benchmarked on the simulated primer-anchored 200 bp
    fragments at the genus rank with the 60% bootstrap support cutoff
  repo: https://sourceforge.net/projects/rdp-classifier/files/rdp-classifier/rdp_classifier_2.13.zip
- name: CO1Classifier training sets
  role: Source of the full-length CO1 reference FASTA (CO1 Eukaryote v1) from which
    the primer-anchored fragments are simulated
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:7454b78fb9bd8de565fd62d4732f3bda6467db2cde420999b32333b589c2350a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CO1Classifier training sets
  - CUTADAPT
  - Mesquite
  - RDP classifier
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# illumina-read-length-fragment-simulation

## Summary

Simulates Illumina metabarcoding queries by primer-trimming full-length CO1 training-set sequences with CUTADAPT (up to 10% primer mismatch) and truncating each primer-anchored retrieval to 200 bp, the average Illumina read length after primer trimming, for RDP classifier benchmarking.

## When to use

Use when benchmarking CO1 taxonomic classifiers on short metabarcoding reads: a full-length 500 bp+ training FASTA is available and primer-anchored 200 bp query fragments mimicking Illumina reads after primer trimming are needed.

## When NOT to use

- Query data are already short, primer-trimmed metabarcoding reads — classify them directly instead of simulating fragments from reference sequences.
- The benchmark targets full-length (500 bp+) classification, where truncating references to 200 bp would test the wrong length regime.
- Species-rank accuracy is the goal — no 99% bootstrap cutoff exists for 200 bp species-rank assignments (species row is NA), so genus-rank or above must be used.

## Inputs

- CO1 training-set FASTA of full-length (500 bp+, ambiguity-free) reference sequences from the CO1Classifier repository (e.g., CO1 Eukaryote v1)
- Compiled CO1 primer list from the literature (Table S14)
- Drosophila yakuba CO1 region (GenBank accession X03240) as the alignment reference for primer binding regions

## Outputs

- primer_trimmed_200bp.fasta — primer-anchored 200 bp query fragments simulating the average Illumina read length after primer trimming

## How to apply

Compile the CO1 primers from the literature (Table S14) and align them in Mesquite v3.10 against the Drosophila yakuba CO1 region (GenBank X03240) to map each primer's binding region. Load the full-length CO1 training-set FASTA (500 bp+ screened, ambiguity-free records from the CO1Classifier repository) and run CUTADAPT v1.10 with default settings, allowing up to a 10% mismatch in the primer sequence, to retrieve primer-trimmed sequences the same way real raw metabarcoding reads would be processed. Truncate every primer-anchored retrieval to a 200 bp fragment — the average length of an Illumina read after primer trimming — and write the fragments to primer_trimmed_200bp.fasta. Classify the fragments with the RDP classifier (v2.12 with the CO1 Eukaryote v1 training set) under leave-one-out and cross-validation testing, applying the genus-rank 60% bootstrap support cutoff defined for 200 bp sequences. Anticipate position-dependent coverage: fragments anchored at the Folmer primers and other 5′/3′-end primers will be under-represented because primers are typically trimmed before GenBank submission.

## Related tools

- **CUTADAPT** (Retrieves primer-trimmed sequences from the full-length 500 bp+ CO1 training FASTA using literature CO1 primers with up to 10% primer-sequence mismatch, mimicking real raw metabarcoding read processing)
- **Mesquite** (Aligns compiled CO1 primers (Table S14) against the Drosophila yakuba CO1 region (GenBank X03240) to determine primer binding regions before trimming)
- **RDP classifier** (Downstream classifier benchmarked on the simulated primer-anchored 200 bp fragments at the genus rank with the 60% bootstrap support cutoff) — https://sourceforge.net/projects/rdp-classifier/files/rdp-classifier/rdp_classifier_2.13.zip
- **CO1Classifier training sets** (Source of the full-length CO1 reference FASTA (CO1 Eukaryote v1) from which the primer-anchored fragments are simulated) — https://github.com/terrimporter/CO1Classifier

## Examples

```
cutadapt -g $PRIMER -e 0.1 -o primer_trimmed.fasta CO1_training.fasta  # -e 0.1 = 10% primer mismatch; then truncate each record to 200 bp into primer_trimmed_200bp.fasta
```

## Evaluation signals

- All records in primer_trimmed_200bp.fasta are exactly 200 bp and were retrieved with no more than 10% mismatch to a CO1 primer sequence.
- Untrimmed full-length training records lacking a primer hit are absent from the output — only primer-anchored retrievals are kept.
- Fragment coverage is position-dependent: Folmer-anchored and other 5′/3′-end-primer-anchored fragments are markedly under-represented, mirroring the paper's training-set coverage finding.
- Input records satisfy the training-set filters: ≥500 bp, no nucleotide ambiguities, human and bacterial contaminants removed.
- Genus-rank classification of the fragments with the 60% bootstrap cutoff reproduces the paper's leave-one-out / cross-validation pattern for 200 bp sequences.

## Limitations

- Fragments anchored at the Folmer barcoding primers and other primers designed near the 5′ and 3′ ends of the barcoding region are under-represented because primers are often trimmed before GenBank submission, biasing coverage in the CO1 Eukaryote v1 training set.
- No species-rank bootstrap cutoff achieves 99% correct assignments for 200 bp fragments (species row is NA in the README cutoff tables); genus-rank or more inclusive ranks must be used.
- Truncation simulates read length only, not sequencing errors, and assumed correctness holds only when query sequences are present in the reference set (false negatives occur when congeneric sequences are missing).

## Evidence

- [methods] Run CUTADAPT v1.10 with default settings, allowing up to a 10% mismatch in the primer sequence: [evidence span withheld — anchor status: not_found]
- [methods] Truncate each retrieved primer-anchored sequence to a 200 bp fragment to simulate the average length of an Illumina read after primer trimming.: [evidence span withheld — anchor status: not_found]
- [results] The proportion of correct assignments of primer-anchored 200 bp sequences with and without 60% bootstrap support (Table 2) is also shown.: [evidence span withheld — anchor status: not_found]
- [readme] GenBank sequences were filtered to only include sequences 500bp+, containing no nucleotide ambiguities, and preferrably with a Linnean binomial species name and/or a BOLD BIN (new).: "GenBank sequences were filtered to only include sequences 500bp+, containing no nucleotide ambiguities, and preferrably with a Linnean binomial species name and/or a BOLD BIN (new)."
- [readme] Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments: "Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments"
- [other] Because primers are often trimmed before sequences are submitted to GenBank, the Folmer barcoding primers and other primers designed near the 5′ and 3′ ends of the barcoding region: [evidence span withheld — anchor status: not_found]
