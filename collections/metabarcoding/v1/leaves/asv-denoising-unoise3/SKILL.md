---
name: asv-denoising-unoise3
description: Use when filtered amplicon reads have been dereplicated (temp/uniques.fa
  with size annotations) and exact-inference ASVs, rather than 97% OTU clustering,
  are desired for downstream OTU tables and diversity analysis.
when_to_use_negative:
- Input is already a feature/OTU table (result/otutab_rare.txt or similar) — denoising
  applies to raw/filtered FASTA reads only.
- The analysis goal is 97% OTU clustering with reference-based dechimerization (e.g.
  vsearch --uchime_ref against rdp_16s_v18.fa) rather than exact ASV inference.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: USEARCH
  role: Runs -unoise3 denoising on dereplicated uniques.fa with -minsize 10 and -cluster_otus
    with -relabel OTU_ as the OTU-clustering alternative
- name: VSEARCH
  role: Performs --derep_fulllength with --minuniquesize 10 and --sizeout to build
    uniques.fa; provides --cluster_size --id 0.97 and --uchime_denovo for very large
    datasets
  repo: https://github.com/torognes/vsearch
- name: EasyAmplicon
  role: Pipeline that integrates USEARCH and VSEARCH and chains dereplication, denoising,
    and relabeling into temp/otus.fa
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:aea5df83a7b82aac28cd6d851f8c3528c13a19f686afffa4d05266e712e0794a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ASV denoising with UNOISE3

## Summary

Denoises dereplicated amplicon reads into zero-radius OTUs (ASVs) using USEARCH -unoise3 (or VSEARCH alternatives) within the EasyAmplicon pipeline, then relabels Zotu headers to ASV_ to produce the representative-sequence file temp/otus.fa.

## When to use

Use when filtered amplicon reads have been dereplicated (temp/uniques.fa with size annotations) and exact-inference ASVs, rather than 97% OTU clustering, are desired for downstream OTU tables and diversity analysis.

## When NOT to use

- Input is already a feature/OTU table (result/otutab_rare.txt or similar) — denoising applies to raw/filtered FASTA reads only.
- The analysis goal is 97% OTU clustering with reference-based dechimerization (e.g. vsearch --uchime_ref against rdp_16s_v18.fa) rather than exact ASV inference.

## Inputs

- temp/filtered.fa (quality- and length-filtered merged amplicon sequences in FASTA format)
- temp/uniques.fa (dereplicated unique sequences with abundance/size annotations, Uni_ relabeling)

## Outputs

- zotus.fa (denoised zero-radius OTUs from -unoise3)
- temp/otus.fa (representative-sequence file with ASV_-prefixed headers after sed relabeling)
- reported counts of chimeras removed by the denoising/clustering command

## How to apply

First dereplicate temp/filtered.fa with vsearch --derep_fulllength using --minuniquesize 10, --sizeout, and --relabel Uni_ to write unique sequences with abundance annotations to temp/uniques.fa. Then run usearch -unoise3 temp/uniques.fa with -minsize 10 to denoise sequences into zotus.fa, and relabel the 'Zotu' prefix to 'ASV_' via sed to obtain temp/otus.fa, the representative-sequence file. If 97% OTUs are preferred instead, usearch -cluster_otus temp/uniques.fa -minsize 10 -relabel OTU_ produces temp/otus.fa with de novo chimera filtering included; for very large datasets substitute vsearch --cluster_size --id 0.97 to get temp/otus_raw.fa centroids followed by vsearch --uchime_denovo keeping non-chimeras as temp/otus.fa. The -minsize 10 threshold mirrors --minuniquesize 10, ensuring only sequences observed at least 10 times enter denoising/clustering, which limits error propagation and spurious ASV inference.

## Related tools

- **USEARCH** (Runs -unoise3 denoising on dereplicated uniques.fa with -minsize 10 and -cluster_otus with -relabel OTU_ as the OTU-clustering alternative)
- **VSEARCH** (Performs --derep_fulllength with --minuniquesize 10 and --sizeout to build uniques.fa; provides --cluster_size --id 0.97 and --uchime_denovo for very large datasets) — https://github.com/torognes/vsearch
- **EasyAmplicon** (Pipeline that integrates USEARCH and VSEARCH and chains dereplication, denoising, and relabeling into temp/otus.fa) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
usearch -unoise3 temp/uniques.fa -minsize 10 -zotus zotus.fa && sed 's/Zotu/ASV_/' zotus.fa > temp/otus.fa
```

## Evaluation signals

- temp/otus.fa exists and is valid FASTA with headers carrying the ASV_ (or OTU_) prefix
- Sequence names in the pre-relabel zotus.fa output start with 'Zotu' and are relabeled to 'ASV_' before use as representatives
- uniques.fa headers carry abundance/size annotations (--sizeout) and a Uni_ relabel prefix
- The clustering/denoising command reports the number of chimeras removed
- Minimum-size thresholds are consistent: -minsize 10 (unoise3/cluster_otus) matches --minuniquesize 10 (derep_fulllength)

## Limitations

- USEARCH is proprietary (VSEARCH v2.7.1+ or later is the open-source alternative and may differ in sensitivity since VSEARCH uses a full Needleman-Wunsch global aligner rather than USEARCH's heuristic seed-and-extend)
- For very large datasets, the USEARCH route may be impractical; the VSEARCH --cluster_size/--uchime_denovo route is the recommended substitute
- No changelog for the protocol/repository is provided, so parameter provenance across versions is not documented

## Evidence

- [methods] ASV denoising of dereplicated sequences, sequence names are relabeled from 'Zotu' to 'ASV_' and the denoised sequences are written as temp/otus.fa: [evidence span withheld — anchor status: not_found]
- [methods] run usearch -unoise3 temp/uniques.fa with -minsize 10 to generate zotus.fa, then relabel Zotu to ASV_: [evidence span withheld — anchor status: not_found]
- [methods] cluster_otus temp/uniques.fa with -minsize 10 and -relabel OTU_: [evidence span withheld — anchor status: not_found]
- [intro] VSEARCH v2.7.1+52 or later ... an open-source tool for performing tasks such as sequence clustering, chimera detection, and dereplication: "VSEARCH v2.7.1+52 or later: VSEARCH is an open‐source tool for performing tasks such as sequence clustering, chimera detection, and dereplication."
- [readme] VSEARCH README: supports de novo and reference based chimera detection, clustering, full-length and prefix dereplication: [evidence span withheld — anchor status: external_doc]
