---
name: sequence-abundance-annotation-relabeling
description: Use when filtered amplicon FASTA (e.g. temp/filtered.fa) must be de-replicated
  and reduced to non-redundant representative ASV/OTU sequences with abundance annotations
  for downstream OTU-table construction.
when_to_use_negative:
- temp/otus.fa (representative sequences) already exists and the next step is OTU-table
  building or taxonomy assignment
- Input is still raw paired-end FASTQ that has not passed primer removal and length/quality
  filtering (vsearch --fastx_filter)
- Long-read PacBio/Nanopore amplicon data requiring the EasyAmplicon 2 long-read pipeline
  rather than the Illumina short-read OTU/ASV route
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0102
tools:
- name: USEARCH
  role: Performs ASV denoising (-unoise3) and 97% OTU clustering (-cluster_otus) with
    -minsize 10 and prefix relabeling to produce temp/otus.fa
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: VSEARCH
  role: De-replicates filtered reads (--derep_fulllength with --minuniquesize 10 --sizeout
    --relabel Uni_) and provides the open-source alternative clustering/chimera route
    (--cluster_size --id 0.97, --uchime_denovo)
  repo: https://github.com/torognes/vsearch
- name: EasyAmplicon
  role: Pipeline that integrates USEARCH/VSEARCH and packages the dereplication–denoising/clustering–relabeling
    workflow ending in temp/otus.fa
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:2c0ecf808385e676789a4e9a5bb3dbda8ec206f3ca20c751b4dca4fee5877893
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sequence-abundance-annotation-relabeling

## Summary

De-replicates filtered amplicon reads, then selects representative sequences either as ASVs (usearch -unoise3) or 97% OTUs (usearch -cluster_otus), relabeling headers (Zotu→ASV_, or -relabel OTU_/Uni_) to produce the representative-sequence file temp/otus.fa.

## When to use

Use when filtered amplicon FASTA (e.g. temp/filtered.fa) must be de-replicated and reduced to non-redundant representative ASV/OTU sequences with abundance annotations for downstream OTU-table construction.

## When NOT to use

- temp/otus.fa (representative sequences) already exists and the next step is OTU-table building or taxonomy assignment
- Input is still raw paired-end FASTQ that has not passed primer removal and length/quality filtering (vsearch --fastx_filter)
- Long-read PacBio/Nanopore amplicon data requiring the EasyAmplicon 2 long-read pipeline rather than the Illumina short-read OTU/ASV route

## Inputs

- temp/filtered.fa — quality-filtered amplicon sequences in FASTA format
- temp/uniques.fa — de-replicated unique sequences with size= abundance annotations
- Reference chimera database (e.g. usearch/rdp_16s_v18.fa) for uchime_ref route

## Outputs

- temp/uniques.fa — unique sequences relabeled Uni_ with --sizeout abundance annotations
- temp/otus.fa — representative ASV (ASV_ prefix) or OTU (OTU_ prefix) sequences in FASTA format
- Chimera-removal statistics reported by clustering/denoising commands

## How to apply

First de-replicate temp/filtered.fa with vsearch --derep_fulllength using --minuniquesize 10, --sizeout, and --relabel Uni_ to write abundance-annotated unique sequences to temp/uniques.fa. Then choose ASV denoising: run usearch -unoise3 on temp/uniques.fa with -minsize 10, and relabel the 'Zotu' prefixes to 'ASV_' via sed to obtain temp/otus.fa. Alternatively, choose 97% OTU clustering: usearch -cluster_otus temp/uniques.fa -minsize 10 -relabel OTU_ produces temp/otus.fa with de novo chimera filtering included by default. For very large datasets, substitute the VSEARCH route: vsearch --cluster_size --id 0.97 yields temp/otus_raw.fa centroids, and vsearch --uchime_denovo (or reference-based uchime_ref against e.g. rdp_16s_v18) keeps non-chimeras as temp/otus.fa; --minuniquesize 10 filters singleton/doubleton uniques before denoising.

## Related tools

- **USEARCH** (Performs ASV denoising (-unoise3) and 97% OTU clustering (-cluster_otus) with -minsize 10 and prefix relabeling to produce temp/otus.fa) — https://github.com/YongxinLiu/EasyAmplicon
- **VSEARCH** (De-replicates filtered reads (--derep_fulllength with --minuniquesize 10 --sizeout --relabel Uni_) and provides the open-source alternative clustering/chimera route (--cluster_size --id 0.97, --uchime_denovo)) — https://github.com/torognes/vsearch
- **EasyAmplicon** (Pipeline that integrates USEARCH/VSEARCH and packages the dereplication–denoising/clustering–relabeling workflow ending in temp/otus.fa) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
vsearch --derep_fulllength temp/filtered.fa --minuniquesize 10 --sizeout --relabel Uni_ -output temp/uniques.fa; usearch -unoise3 temp/uniques.fa -minsize 10 -zotus zotus.fa; sed 's/Zotu/ASV_/' zotus.fa > temp/otus.fa
```

## Evaluation signals

- temp/otus.fa exists and parses as valid FASTA
- Sequence headers carry the correct prefix: ASV_ for the unoise3 route (after sed relabel of Zotu), OTU_ for cluster_otus, Uni_ with size= abundance annotations in uniques.fa
- The clustering/denoising command reports the number of chimeras removed; non-chimera count should be consistent with otus.fa sequence count
- No representative sequence has abundance below the -minsize 10 threshold (uniques smaller than 10 were excluded)
- For the VSEARCH fallback, otus.fa should equal the non-chimera subset of otus_raw.fa centroids clustered at --id 0.97

## Limitations

- ASV denoising (unoise3) and OTU clustering (cluster_otus) are alternative branches — only one route should be chosen; results differ between the two
- --minuniquesize/-minsize 10 discards low-abundance uniques, so rare singletons/doubletons never become OTUs/ASVs
- De novo chimera filtering (cluster_otus) is the default; reference-based dechimeria (uchime_ref against rdp_16s_v18) is a separate step that may also be needed
- USEARCH is free but not open source; for very large datasets the VSEARCH route (--cluster_size at --id 0.97 plus --uchime_denovo) substitutes, with potentially different clustering results

## Evidence

- [methods] Use tools like **VSEARCH** to merge identical sequences into a single one to reduce redundant calculations: "Use tools like **VSEARCH** to merge identical sequences into a single one to reduce redundant calculations"
- [methods] By default, de novo dechimerization is used (as described above with `cluster_otus`). Reference-based dechimerization can also be performed: "By default, de novo dechimerization is used (as described above with `cluster_otus`). Reference-based dechimerization can also be performed"
- [intro] USEARCH is a versatile tool for high-throughput sequence analysis, offering capabilities such as clustering, chimera detection, and OTU picking.: "USEARCH is a versatile tool for high‐throughput sequence analysis, offering capabilities such as clustering, chimera detection, and OTU picking."
- [intro] VSEARCH v2.7.1+52 or later: VSEARCH is an open-source tool for performing tasks such as sequence clustering, chimera detection, and dereplication.: "VSEARCH v2.7.1+52 or later: VSEARCH is an open‐source tool for performing tasks such as sequence clustering, chimera detection, and dereplication."
