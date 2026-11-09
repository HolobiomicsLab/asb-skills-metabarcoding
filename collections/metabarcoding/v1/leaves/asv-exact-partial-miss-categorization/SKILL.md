---
name: asv-exact-partial-miss-categorization
description: Use when you have ASVs/OTUs from a mock community amplicon dataset and
  a reference fasta of expected sequences, and need to count how many variants are
  exact matches, near-matches, or unexpected relative to the known mock composition.
when_to_use_negative:
- Input is an environmental/community sample with no known expected taxon list — there
  is no reference set to define 'exact' or 'miss' categories.
- You need taxonomic classification of unknown sequences — this skill only matches
  against a closed mock reference set, not a general reference database or classifier.
- Your goal is to assess classifier accuracy (e.g., BOLD API vs. naive Bayes) rather
  than denoising/clustering recovery of mock sequences.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: DADA2
  role: Denoise trimmed paired-end reads into error-corrected ASVs that are then aligned
    to the expected mock references
  repo: https://github.com/devonorourke/tidybug
- name: Deblur
  role: Alternative denoiser run with --p-min-reads 2 and --p-min-size 1 so only singleton
    reads are discarded; its representative sequences feed the same exact/partial/miss
    alignment
  repo: https://github.com/devonorourke/tidybug
- name: VSEARCH
  role: OTU-clustering route (merge, dereplicate, 98% identity clustering, de novo
    chimera filtering, then 97% clustering) whose representatives are categorized
    against mock references
  repo: https://github.com/torognes/vsearch
- name: QIIME 2
  role: Environment (v2018.11) hosting DADA2, Deblur, and VSEARCH plugins used to
    generate representative sequences
  repo: https://github.com/devonorourke/tidybug
- name: Cutadapt
  role: Trims primers from paired-end reads before denoising/clustering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:9a9ba72fb0074be9e6511cde978ecca686b24a03367a16ff44f77f422aa944d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/devonorourke/tidybug
  tools:
  - Cutadapt
  - DADA2
  - Deblur
  - QIIME 2
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ASV exact/partial/miss categorization

## Summary

Categorize denoised or clustered representative sequences from a biological mock community as 'exact' (100% identity), 'partial' (97%-99% identity), or 'miss' (<97% identity) by aligning each variant to a set of expected mock reference sequences requiring at least 97% coverage. This quantifies how well DADA2, Deblur, or VSEARCH processing recovers known mock taxa versus spurious variants.

## When to use

Use when you have ASVs/OTUs from a mock community amplicon dataset and a reference fasta of expected sequences, and need to count how many variants are exact matches, near-matches, or unexpected relative to the known mock composition.

## When NOT to use

- Input is an environmental/community sample with no known expected taxon list — there is no reference set to define 'exact' or 'miss' categories.
- You need taxonomic classification of unknown sequences — this skill only matches against a closed mock reference set, not a general reference database or classifier.
- Your goal is to assess classifier accuracy (e.g., BOLD API vs. naive Bayes) rather than denoising/clustering recovery of mock sequences.

## Inputs

- Representative sequence fasta from DADA2, Deblur, or VSEARCH processing of mock community amplicon reads (Libraries A-D, SRA PRJNA518082)
- Expected mock community reference fasta (from the tidybug GitHub repo)

## Outputs

- Per-library table of exact, partial, and miss ASV counts for each processing method (DADA2, Deblur, VSEARCH)

## How to apply

After denoising (DADA2, or Deblur with --p-min-reads 2 and --p-min-size 1 to discard only singleton reads) or after the VSEARCH OTU pipeline (merge paired ends, dereplicate, cluster at 98% identity, de novo chimera filtering, then re-cluster at 97% identity), take every representative sequence and align it to the expected mock reference fasta from the tidybug GitHub repo. Require at least 97% query coverage for a match to count; label 100% identity alignments 'exact', 97%-99% identity alignments 'partial', and anything below 97% identity 'miss'. Assemble a per-library table of exact/partial/miss counts for each processing method so pipelines can be compared; in the tidybug benchmark, DADA2 on basic-filtered data yielded 9 partial and 40 miss ASVs versus Deblur's 146 partial/576 miss and VSEARCH's 295 partial/753 miss.

## Related tools

- **DADA2** (Denoise trimmed paired-end reads into error-corrected ASVs that are then aligned to the expected mock references) — https://github.com/devonorourke/tidybug
- **Deblur** (Alternative denoiser run with --p-min-reads 2 and --p-min-size 1 so only singleton reads are discarded; its representative sequences feed the same exact/partial/miss alignment) — https://github.com/devonorourke/tidybug
- **VSEARCH** (OTU-clustering route (merge, dereplicate, 98% identity clustering, de novo chimera filtering, then 97% clustering) whose representatives are categorized against mock references) — https://github.com/torognes/vsearch
- **QIIME 2** (Environment (v2018.11) hosting DADA2, Deblur, and VSEARCH plugins used to generate representative sequences) — https://github.com/devonorourke/tidybug
- **Cutadapt** (Trims primers from paired-end reads before denoising/clustering)

## Examples

```
qiime feature-classifier align --i-query-reps mock_asvs.qza --i-reference-reads expected_mock_refs.qza; then classify alignments as exact (100% id), partial (97-99% id), or miss (<97% id) with ≥97% coverage.
```

## Evaluation signals

- DADA2 and Deblur runs recover all 24 expected mock sequences (exact + partial covering every reference).
- The VSEARCH clustering route misses between 1 and 4 of the 24 expected mock sequences.
- Every representative sequence assigned a label ('exact', 'partial', or 'miss') achieved at least 97% query coverage against an expected reference; 'partial' labels fall in the 97%-99% identity window.
- Counts are reported per library (A-D) and per method, forming a complete exact/partial/miss table with no unclassified high-coverage alignments.
- DADA2 on basic-filtered data produces fewer partial ASVs (9) and fewer miss ASVs (40) than Deblur (146 partial, 576 miss) and VSEARCH (295 partial, 753 miss).

## Limitations

- Removing low-abundance sequence variants largely mitigates the excess of unexpected sequences from clustering, so the exact/partial/miss profile is highly sensitive to filtering settings applied before categorization.
- Results depend on the 97% coverage and identity thresholds chosen for the match definitions; sequences below 97% coverage have no defined category in this scheme.
- The mock community reference fasta reflects a fixed set of expected taxa; contaminants or index-hopped reads from outside that set will all be collapsed into 'miss' without further taxonomic diagnosis.

## Evidence

- [methods] Align each representative sequence from every run to the expected mock references requiring at least 97% coverage, labeling 100% identity matches 'exact', 97%-99% matches 'partial', and <97% matches 'miss'.: [evidence span withheld — anchor status: not_found]
- [results] DADA2 produced fewer unexpected mock ASVs than the alternatives: for basic-filtered data it contained fewer partial ASVs (9) than Deblur (146) or VSEARCH (295), and fewer miss ASVs (40) than Deblur (576) or VSEARCH (753).: [evidence span withheld — anchor status: not_found]
- [abstract] denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
- [methods] Denoise the trimmed reads with DADA2 and with Deblur (setting --p-min-reads 2 and --p-min-size 1 so only singleton reads are discarded and all singleton ASVs are retained).: [evidence span withheld — anchor status: not_found]
- [methods] Separately run the VSEARCH OTU pipeline on the same input: merge paired ends, dereplicate, cluster at 98% identity, apply de novo chimera filtering, and finally cluster remaining variants at 97% identity.: [evidence span withheld — anchor status: not_found]
