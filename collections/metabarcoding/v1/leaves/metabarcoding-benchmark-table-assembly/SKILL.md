---
name: metabarcoding-benchmark-table-assembly
description: Use when raw paired-end amplicon reads from a biological mock community
  (with a known expected reference fasta) are available and you need to compare how
  denoising vs clustering pipelines recover expected, partial, and missed sequences.
when_to_use_negative:
- Input is already a feature table with no representative sequences or expected mock
  reference fasta available for alignment
- The study has no biological mock community, so exact/partial/miss recovery cannot
  be evaluated against known expected sequences
- The goal is taxonomic classification accuracy (classifier benchmarking) rather than
  sequence-recovery comparison between denoising and clustering
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: DADA2
  role: Denoises trimmed paired-end reads in QIIME 2 to generate error-corrected ASVs
    for benchmarking
- name: Deblur
  role: Denoises trimmed reads with --p-min-reads 2 and --p-min-size 1 to retain all
    non-singleton ASVs for benchmarking
- name: VSEARCH
  role: Merges, dereplicates, chimer-filters, and clusters the same input at 98% then
    97% identity as the OTU-clustering comparator
  repo: https://github.com/torognes/vsearch
- name: QIIME 2
  role: Environment (v2018.11) hosting Cutadapt trimming, DADA2 and Deblur denoising
    of the mock libraries
- name: Cutadapt
  role: Trims primers from paired-end reads before denoising/clustering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:2975ca4328cc493ddd492a92147006bf9ab2affb17140b400547901c75e6d9e1
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabarcoding-benchmark-table-assembly

## Summary

Benchmark amplicon denoising (DADA2, Deblur) against VSEARCH clustering by recovering a mock community's expected sequences, aligning every representative sequence back to the expected mock references, and assembling a per-library table of exact/partial/miss counts to quantify ASV recovery differences.

## When to use

Use when raw paired-end amplicon reads from a biological mock community (with a known expected reference fasta) are available and you need to compare how denoising vs clustering pipelines recover expected, partial, and missed sequences.

## When NOT to use

- Input is already a feature table with no representative sequences or expected mock reference fasta available for alignment
- The study has no biological mock community, so exact/partial/miss recovery cannot be evaluated against known expected sequences
- The goal is taxonomic classification accuracy (classifier benchmarking) rather than sequence-recovery comparison between denoising and clustering

## Inputs

- Raw paired-end mock community reads (Libraries A-D) from NCBI SRA PRJNA518082
- Expected mock community reference sequences (fasta) from the tidybug GitHub repo
- Primer sequences for Cutadapt trimming of the 181 bp COI fragment

## Outputs

- ASV/OTU feature tables and representative sequence fastas for DADA2, Deblur, and VSEARCH runs
- Per-library table of exact/partial/miss counts for each processing method

## How to apply

Retrieve the mock community raw paired-end reads (Libraries A-D) from SRA PRJNA518082 and the expected mock reference fasta from the tidybug GitHub repo. Import reads into a QIIME 2 v2018.11 environment, trim primers with Cutadapt, and retain unmerged paired-end reads with a minimum length of 175 bp. Denoise the trimmed reads with DADA2 and with Deblur (--p-min-reads 2 and --p-min-size 1 so only singleton reads are discarded while all singleton ASVs are retained); separately run the VSEARCH OTU route: merge paired ends, dereplicate, cluster at 98% identity, de novo chimera-filter, then cluster remaining variants at 97% identity. Align each representative sequence from every run to the expected mock references requiring at least 97% coverage, labeling 100% identity matches 'exact', 97%-99% matches 'partial', and <97% matches 'miss'. Rationale: denoising programs generate error models to address sequence errors, and mock communities let you empirically count unexpected sequences that each processing choice introduces.

## Related tools

- **DADA2** (Denoises trimmed paired-end reads in QIIME 2 to generate error-corrected ASVs for benchmarking)
- **Deblur** (Denoises trimmed reads with --p-min-reads 2 and --p-min-size 1 to retain all non-singleton ASVs for benchmarking)
- **VSEARCH** (Merges, dereplicates, chimer-filters, and clusters the same input at 98% then 97% identity as the OTU-clustering comparator) — https://github.com/torognes/vsearch
- **QIIME 2** (Environment (v2018.11) hosting Cutadapt trimming, DADA2 and Deblur denoising of the mock libraries)
- **Cutadapt** (Trims primers from paired-end reads before denoising/clustering)

## Examples

```
./vsearch --usearch_global queries.fsa --db database.fsa --id 0.9 --alnout alnout.txt
```

## Evaluation signals

- Validation: DADA2 and Deblur recover all 24 expected mock sequences, while the VSEARCH clustering route misses between 1 and 4 of them
- DADA2 on basic-filtered data contains fewer partial ASVs (9) than Deblur (146) or VSEARCH (295)
- DADA2 has fewer miss ASVs (40) than Deblur (576) or VSEARCH (753)
- Every representative sequence from each run is labeled exact (100% identity), partial (97%-99%), or miss (<97%) against references at ≥97% coverage
- A complete per-library table exists with exact/partial/miss counts for each of the three processing methods

## Limitations

- Denoising methods' lower error rates compared to traditional clustering are largely mitigated by removing low-abundance sequence variants
- Software versions for the denoising programs are not reported in the results, hindering exact reproduction
- The clustering-based VSEARCH route sacrifices expected mock sequences (misses 1-4 of the 24 expected), so OTU outputs are not directly comparable in recovery completeness

## Evidence

- [intro] Denoising programs generate error models: "Denoising programs like DADA2 (Callahan et al., 2016) or Deblur (Amir et al., 2017) generate error models to address potential sequence errors"
- [methods] VSEARCH pipeline parameters mirrored from wiki: [evidence span withheld — anchor status: not_found]
- [methods] Clustering at multiple identity thresholds produces per-cluster fasta outputs: "We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id"
- [intro] Mock communities assess unexpected sequences and filtering strategies: [evidence span withheld — anchor status: not_found]
- [readme] SRA data location for raw mock reads: "SRA data available at: https://www.ncbi.nlm.nih.gov/sra/PRJNA518082"
