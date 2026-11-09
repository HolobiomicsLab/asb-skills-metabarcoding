---
name: vsearch-otu-clustering-execution
description: Use when paired-end amplicon reads must be processed with an OTU/clustering
  pipeline instead of a denoiser (DADA2/Deblur), or when benchmarking how clustering
  inflates unexpected sequences relative to denoising.
when_to_use_negative:
- Input is already a feature table or representative-sequence fasta from another pipeline
  — this skill operates on raw primer-trimmed paired-end reads.
- 'The goal is to preserve all expected mock sequences: the article''s validation
  expects VSEARCH clustering to miss 1-4 of the 24 expected mock sequences, so denoising
  (DADA2/Deblur) is preferable when completeness matters.'
- Data are microbial rRNA amplicons (e.g., UNITE/SILVA/Greengenes targets) — the tidybug
  pipeline is designed for a 181 bp COI animal-metabarcoding fragment.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3941
tools:
- name: VSEARCH
  role: Performs the merge, full-length dereplication, 98%/97% identity de novo clustering,
    and de novo chimera filtering on the mock community reads
  repo: https://github.com/torognes/vsearch
- name: QIIME 2
  role: Hosts the workflow environment (v2018.11) in which reads are imported and
    the VSEARCH OTU route is run
  repo: https://github.com/devonorourke/tidybug
- name: Cutadapt
  role: Trims primers and retains unmerged paired-end reads with a minimum length
    of 175 bp prior to clustering
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:d1782cd139a868c485deb636be4285f11ca741f79493ff5f8309f08c75579df4
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# vsearch-otu-clustering-execution

## Summary

Execute the VSEARCH OTU pipeline (dereplication, de novo clustering at 98% identity, de novo chimera filtering, final clustering at 97% identity) to define representative sequences for metabarcoding data, as the clustering-based comparator to ASV denoising in the tidybug mock-community benchmark.

## When to use

Use when paired-end amplicon reads must be processed with an OTU/clustering pipeline instead of a denoiser (DADA2/Deblur), or when benchmarking how clustering inflates unexpected sequences relative to denoising.

## When NOT to use

- Input is already a feature table or representative-sequence fasta from another pipeline — this skill operates on raw primer-trimmed paired-end reads.
- The goal is to preserve all expected mock sequences: the article's validation expects VSEARCH clustering to miss 1-4 of the 24 expected mock sequences, so denoising (DADA2/Deblur) is preferable when completeness matters.
- Data are microbial rRNA amplicons (e.g., UNITE/SILVA/Greengenes targets) — the tidybug pipeline is designed for a 181 bp COI animal-metabarcoding fragment.

## Inputs

- Primer-trimmed paired-end FASTQ reads (mock community libraries A-D from SRA PRJNA518082)
- Single fasta file of sequences filtered to >100 bp with gap characters removed, formatted for VSEARCH
- Expected mock community reference sequences (tidybug GitHub repo)

## Outputs

- Representative OTU sequences (fasta) clustered at 97% identity
- Per-library table of exact/partial/miss counts for the VSEARCH route (comparable to DADA2 and Deblur outputs)
- Mock error rates summary (mock_error_rates.csv) across basic/standard/extra filtering levels

## How to apply

Import the primer-trimmed paired-end reads (from SRA PRJNA518082 mock community libraries A-D) into QIIME 2 (v2018.11), then run the VSEARCH route: merge paired ends, dereplicate full-length sequences, cluster at 98% identity, apply de novo chimera detection, and finally cluster the remaining variants at 97% identity. Before clustering, format the input fasta by removing gap characters and selecting only sequences longer than 100 bp, and check for non-ATCG ambiguous characters. Parameters should mirror the VSEARCH Wiki GitHub pipeline (github.com/torognes/vsearch/wiki/VSEARCH-pipeline). After obtaining representative sequences, optionally compare error rates against DADA2 and Deblur (run with --p-min-reads 2 and --p-min-size 1 so only singleton reads are discarded) by aligning each sequence to expected mock references at ≥97% coverage and labeling matches as 'exact' (100%), 'partial' (97-99%), or 'miss' (<97%).

## Related tools

- **VSEARCH** (Performs the merge, full-length dereplication, 98%/97% identity de novo clustering, and de novo chimera filtering on the mock community reads) — https://github.com/torognes/vsearch
- **QIIME 2** (Hosts the workflow environment (v2018.11) in which reads are imported and the VSEARCH OTU route is run) — https://github.com/devonorourke/tidybug
- **Cutadapt** (Trims primers and retains unmerged paired-end reads with a minimum length of 175 bp prior to clustering)

## Examples

```
vsearch --derep_fulllength tmp_nolabels.fasta
```

## Evaluation signals

- DADA2 and Deblur recover all 24 expected mock sequences, while the VSEARCH clustering route misses between 1 and 4 of them.
- Under basic filtering, VSEARCH yields far more unexpected mock ASVs than the denoisers: 295 partial and 753 miss versus DADA2's 9 partial and 40 miss.
- VSEARCH error-rate differences versus denoising methods are largely mitigated after applying the standard filter (drop variants in only one sample, retain samples with ≥5,000 filtered reads) and the extra filter (subtract per-library maximum 'miss' variant count from every feature-table element).
- Clustering produces three output fasta files at 99%, 97%, and 95% identity thresholds, one per cluster %id.
- VSEARCH alignments use the optimal global Needleman-Wunsch aligner (full dynamic programming), so check merge/align behavior differs from USEARCH's heuristic seed-and-extend.

## Limitations

- Clustering at 97% after 98% preclustering inflates unexpected (partial and miss) sequences relative to DADA2 and Deblur under the basic filter, so raw VSEARCH output overstates community diversity.
- Software versions for the denoising programs and clustering route are not reported in the results section of the manuscript.
- The reference fasta and mock expectations depend on the tidybug repo contents; results may shift with different mock community composition or read lengths.

## Evidence

- [results] Under basic filtering, DADA2 produced fewer unexpected mock ASVs than the other methods (partial: 9 vs 146 for Deblur and 295 for VSEARCH; miss: 40 vs 576 and 753): [evidence span withheld — anchor status: not_found]
- [methods] We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id: "We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id"
- [methods] We removed any gap character then selected only sequences longer than 100 bp to produce a single fasta file formatted for further processing in Vsearch: "We removed any gap character then selected only sequences longer than 100 bp to produce a single fasta file formatted for further processing in Vsearch"
- [methods] mirrored the parameters outlined at the VSEARCH Wiki GitHub page (https://github.com/torognes/vsearch/wiki/VSEARCH-pipeline): [evidence span withheld — anchor status: not_found]
- [readme] VSEARCH uses an optimal global aligner (full dynamic programming Needleman-Wunsch), in contrast to USEARCH which by default uses a heuristic seed and extend aligner.: [evidence span withheld — anchor status: external_doc]
