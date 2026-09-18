---
name: mock-community-reference-alignment-scoring
description: Use when a metabarcoding run includes a biological mock community and
  you need to count expected, partial, and unexpected (miss) sequences to benchmark
  DADA2/Deblur/VSEARCH processing pipelines.
when_to_use_negative:
- Input is an already taxonomically classified feature table with no representative
  sequences available for alignment
- The dataset contains no biological mock community, so there is no expected reference
  set to score against
- The goal is taxonomic assignment to GenBank/BOLD databases rather than benchmarking
  against a known mock composition
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: VSEARCH
  role: Global aligner used to align representative ASV/OTU sequences to the expected
    mock reference sequences with a 97% coverage requirement and identity thresholds
    for exact/partial/miss labeling
  repo: https://github.com/torognes/vsearch
- name: DADA2
  role: Produces one set of representative ASV sequences whose exact/partial/miss
    recovery is scored against the mock references
- name: Deblur
  role: Produces a second set of representative ASV sequences (with --p-min-reads
    2 and --p-min-size 1) scored against the mock references
- name: QIIME 2
  role: Environment (v2018.11) in which reads are imported, primers trimmed with Cutadapt,
    and DADA2/Deblur denoising run before reference scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:799fa3e58d52061e350b1c6c3f10b43e36c763ea61eb5f1a2454592477aaeb10
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# mock-community-reference-alignment-scoring

## Summary

Scores each representative ASV/OTU sequence from a denoised or clustered mock-community dataset against an expected mock reference FASTA by aligning with a minimum 97% query coverage and labeling hits as 'exact' (100% identity), 'partial' (97-99% identity), or 'miss' (<97% identity).

## When to use

Use when a metabarcoding run includes a biological mock community and you need to count expected, partial, and unexpected (miss) sequences to benchmark DADA2/Deblur/VSEARCH processing pipelines.

## When NOT to use

- Input is an already taxonomically classified feature table with no representative sequences available for alignment
- The dataset contains no biological mock community, so there is no expected reference set to score against
- The goal is taxonomic assignment to GenBank/BOLD databases rather than benchmarking against a known mock composition

## Inputs

- Representative sequence FASTA from DADA2 or Deblur denoising (ASV sequences)
- Representative sequence FASTA from VSEARCH OTU clustering
- Expected mock community reference FASTA (e.g., from the tidybug GitHub repo)
- Per-library mock community reads from SRA PRJNA518082 (Libraries A-D) after primer trimming

## Outputs

- Per-library table of exact/partial/miss counts for each processing method (DADA2, Deblur, VSEARCH)
- Labeled alignment results classifying each representative sequence as exact, partial, or miss relative to the mock references

## How to apply

After denoising with DADA2 or Deblur (retaining singleton ASVs via --p-min-reads 2 and --p-min-size 1) or OTU clustering with VSEARCH (98% then 97% identity after de novo chimera filtering), extract each method's representative sequences. Align every representative sequence to the expected mock reference sequences (e.g., the tidybug mock reference FASTA) requiring at least 97% coverage, using a global aligner such as VSEARCH. Classify 100% identity matches as 'exact', 97%-99% identity matches as 'partial', and matches below 97% as 'miss'. Assemble a per-library table of exact/partial/miss counts per processing method; for the tidybug mock libraries, DADA2 and Deblur should recover all 24 expected mock sequences, while VSEARCH clustering misses between 1 and 4, and DADA2 yields fewer partial ASVs (9) than Deblur (146) or VSEARCH (295).

## Related tools

- **VSEARCH** (Global aligner used to align representative ASV/OTU sequences to the expected mock reference sequences with a 97% coverage requirement and identity thresholds for exact/partial/miss labeling) — https://github.com/torognes/vsearch
- **DADA2** (Produces one set of representative ASV sequences whose exact/partial/miss recovery is scored against the mock references)
- **Deblur** (Produces a second set of representative ASV sequences (with --p-min-reads 2 and --p-min-size 1) scored against the mock references)
- **QIIME 2** (Environment (v2018.11) in which reads are imported, primers trimmed with Cutadapt, and DADA2/Deblur denoising run before reference scoring)

## Examples

```
./vsearch --usearch_global reps.fsa --db expected_mock_refs.fsa --id 0.97 --alnout mock_alignments.txt
```

## Evaluation signals

- DADA2 and Deblur runs recover all 24 expected mock sequences as exact matches
- VSEARCH OTU clustering misses between 1 and 4 of the 24 expected mock sequences
- Basic-filtered DADA2 data yields fewer partial ASVs (9) than Deblur (146) or VSEARCH (295) and fewer miss ASVs (40) than Deblur (576) or VSEARCH (753)
- Every representative sequence receives exactly one label (exact, partial, or miss) based on the 97%-99% identity and <97% identity thresholds
- All alignments pass the minimum 97% query coverage requirement before identity labeling is applied

## Limitations

- The number of unexpected mock ASVs depends on the denoising/clustering method and on filtering: differences between denoising and clustering are largely mitigated by removing low-abundance sequence variants
- The study's aim is not to present a single best pathway but to illustrate how processes are affected by program or parameter choices, so counts are comparative rather than prescriptive
- Results are specific to the 181 bp COI fragment and the Libraries A-D mock community composition from PRJNA518082

## Evidence

- [methods] labeling 100% identity matches 'exact', 97%-99% matches 'partial', and <97% matches 'miss': [evidence span withheld — anchor status: not_found]
- [results] DADA2 contained fewer partial ASVs (9) than Deblur (146) or VSEARCH (295): [evidence span withheld — anchor status: not_found]
- [abstract] denoising methods have lower error rates compared to traditional clustering methods: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
- [readme] ./vsearch --usearch_global queries.fsa --db database.fsa --id 0.9 --alnout alnout.txt: [evidence span withheld — anchor status: external_doc]
- [abstract] biological mock communities offer a useful standard to evaluate the myriad computational decisions impacting animal metabarcoding accuracy: "biological mock communities offer a useful standard to evaluate the myriad computational decisions impacting animal metabarcoding accuracy"
