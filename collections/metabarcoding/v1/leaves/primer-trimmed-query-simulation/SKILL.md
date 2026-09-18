---
name: primer-trimmed-query-simulation
description: Use when you need realistic primer-anchored short CO1 query fragments
  (e.g., 200 bp) from full-length reference sequences to benchmark taxonomic classifiers
  such as the RDP classifier or top BLAST hit methods on simulated Illumina-like metabarcoding
  data.
when_to_use_negative:
- When the input queries are already short amplicon reads from real sequencing data
  rather than full-length reference sequences needing simulation
- When benchmarking species-rank assignment, since the paper notes no bootstrap cutoff
  can guarantee 99% correct species-rank assignments (Species = NA for 99% tables)
  and 200 bp fragments are only scored at the genus rank in this workflow
- When the target locus is not CO1 and primer sequences/barcode region differ substantially
  from those in the CO1 Eukaryote training sets
tools:
- name: CUTADAPT
  role: Retrieves primer-trimmed sequences from the CO1 training set using default
    settings with up to a 10% mismatch in the primer sequence
- name: RDP classifier v2.12
  role: Consumes the simulated 200 bp query fragments to make genus-rank taxonomic
    assignments against the CO1 Eukaryote v1 training set, with 'good' defined as
    bootstrap proportion >= 0.60
  repo: https://github.com/terrimporter/CO1Classifier/releases
- name: BLAST
  role: Searches the same 200 bp query fragments against a locally installed copy
    of the NCBI nucleotide database for the top-hit speed/accuracy comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:28289641cb43162670e4b2fba41d4719a53bd3f1b2889400cd4fbcf24448605d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - CUTADAPT
  - RDP classifier v2.12
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# primer-trimmed-query-simulation

## Summary

Simulates realistic metabarcoding query reads by using CUTADAPT to retrieve primer-trimmed CO1 sequences from the training/reference set and truncating them to 200 bp fragments, mimicking Illumina reads after primer trimming. These simulated queries are then used to benchmark taxonomic assignment speed and accuracy (e.g., RDP classifier vs top BLAST hit).

## When to use

Use when you need realistic primer-anchored short CO1 query fragments (e.g., 200 bp) from full-length reference sequences to benchmark taxonomic classifiers such as the RDP classifier or top BLAST hit methods on simulated Illumina-like metabarcoding data.

## When NOT to use

- When the input queries are already short amplicon reads from real sequencing data rather than full-length reference sequences needing simulation
- When benchmarking species-rank assignment, since the paper notes no bootstrap cutoff can guarantee 99% correct species-rank assignments (Species = NA for 99% tables) and 200 bp fragments are only scored at the genus rank in this workflow
- When the target locus is not CO1 and primer sequences/barcode region differ substantially from those in the CO1 Eukaryote training sets

## Inputs

- CO1 reference/training sequences (e.g., CO1 Eukaryote v1 training set FASTA)
- Metabarcode primer sequences (with allowed 10% mismatch tolerance)
- Desired simulated read length (200 bp to mimic primer-trimmed Illumina reads)

## Outputs

- Primer-trimmed CO1 sequences (CUTADAPT output)
- 200 bp query fragment FASTA simulating primer-trimmed Illumina reads
- Downstream benchmark outputs: taxonomic assignments with bootstrap support (RDP) and top-hit BLAST results with throughput in sequences per minute (e.g., speed_benchmark.csv)

## How to apply

Start from the CO1 reference/training sequences (e.g., the CO1 Eukaryote v1 training set from github.com/terrimporter/CO1Classifier). Run CUTADAPT v1.10 with default settings, allowing up to a 10% mismatch in the primer sequence, to extract primer-trimmed sequences. Truncate the primer-trimmed sequences to 200 bp fragments anchored at the primer to simulate Illumina reads after primer trimming. Use these fragments as queries for both the RDP classifier v2.12 (scoring genus-rank assignments as 'good' when bootstrap proportion >= 0.60) and the top BLAST hit method (scoring hits as 'good' when percent identity >= 95% and alignment spans >= 85% of query length). The rationale is that classification performance is similar across the length of the CO1 barcoding region, and the paper calculated cutoffs specifically for primer-anchored 200 bp sequences.

## Related tools

- **CUTADAPT** (Retrieves primer-trimmed sequences from the CO1 training set using default settings with up to a 10% mismatch in the primer sequence)
- **RDP classifier v2.12** (Consumes the simulated 200 bp query fragments to make genus-rank taxonomic assignments against the CO1 Eukaryote v1 training set, with 'good' defined as bootstrap proportion >= 0.60) — https://github.com/terrimporter/CO1Classifier/releases
- **BLAST** (Searches the same 200 bp query fragments against a locally installed copy of the NCBI nucleotide database for the top-hit speed/accuracy comparison)

## Evaluation signals

- All simulated query fragments are exactly 200 bp and anchored at the primer (primer-trimmed, no primer sequence retained)
- CUTADAPT recovery rate: the number of primer-trimmed sequences retrieved is consistent with the number of reference sequences containing the primer (allowing <=10% primer mismatch)
- Genus-rank RDP assignments on the 200 bp fragments use the 0.60 bootstrap cutoff defined in Table 2 for primer-anchored 200 bp sequences
- The measured sequences-per-minute throughput ratio (RDP/BLAST) should be approximately 19x, matching Figure 4 of the paper
- BLAST top hits scored as 'good' must satisfy percent identity >= 95% and alignment span >= 85% of query length

## Limitations

- Simulation assumes queries are present in the reference set; the paper notes false negatives occur when congeneric sequences are missing from the database, a particular concern in quarantine situations
- The 200 bp fragment simulation is specific to the primer-anchored scenario; the paper's 99% cutoff matrix is length-dependent (e.g., genus cutoff 70 for 500 bp+ vs 60 for 200 bp in the v1 training set), so results do not transfer to other read lengths without recalibration
- CO1 reference database coverage is incomplete (only 22% of extant Insecta genera have full-length species-rank CO1 barcodes in GenBank), so simulated queries may not represent real diversity

## Evidence

- [abstract] we show that classification performance is similar across the length of the CO1 barcoding region: "we show that classification performance is similar across the length of the CO1 barcoding region"
- [results] The proportion of correct assignments of primer-anchored 200 bp sequences with and without 60% bootstrap support (Table 2) is also shown.: [evidence span withheld — anchor status: not_found]
- [readme] using these minimum bootstrap support cutoffs should result in at least 99% correct assignments: "Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments"
