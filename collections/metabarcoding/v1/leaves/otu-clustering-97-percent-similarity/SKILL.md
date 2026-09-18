---
name: otu-clustering-97-percent-similarity
description: Use when input is a length/quality-filtered amplicon FASTA (e.g. temp/filtered.fa)
  and the goal is to collapse redundant reads into representative OTU_ or ASV_ sequences
  before building an OTU table.
when_to_use_negative:
- Input is already a representative-sequence file (temp/otus.fa) or an OTU table —
  downstream steps like chimera_ref filtering or OTU table construction are the appropriate
  skills instead
- Sequences have not been dereplicated with a minimum abundance threshold (minuniquesize/minsize
  10) — clustering raw filtered reads with singletons inflates spurious OTUs and noise
- The goal is closed-reference picking against a curated database rather than de novo
  clustering/denoising from the sample data itself
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: USEARCH
  role: Performs dereplication-adjacent denoising (-unoise3) and de novo 97% OTU clustering
    with built-in chimera filtering (-cluster_otus -minsize 10 -relabel OTU_)
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: VSEARCH
  role: Open-source dereplication (--derep_fulllength --minuniquesize 10 --sizeout
    --relabel Uni_) and alternative large-dataset clustering route (--cluster_size
    --id 0.97 plus --uchime_denovo)
  repo: torognes/vsearch
- name: EasyAmplicon
  role: Pipeline that orchestrates the dereplication → clustering/denoising → temp/otus.fa
    workflow steps
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:3bc91d17826456122bb391d1d14fa89acb82bada5f3046d7a2ba51920af67cc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# otu-clustering-97-percent-similarity

## Summary

Reduces redundancy in filtered amplicon reads and selects representative OTU/ASV sequences by dereplication followed by 97% de novo OTU clustering (usearch -cluster_otus) or ASV denoising (usearch -unoise3), producing the representative-sequence file temp/otus.fa in the EasyAmplicon pipeline.

## When to use

Use when input is a length/quality-filtered amplicon FASTA (e.g. temp/filtered.fa) and the goal is to collapse redundant reads into representative OTU_ or ASV_ sequences before building an OTU table.

## When NOT to use

- Input is already a representative-sequence file (temp/otus.fa) or an OTU table — downstream steps like chimera_ref filtering or OTU table construction are the appropriate skills instead
- Sequences have not been dereplicated with a minimum abundance threshold (minuniquesize/minsize 10) — clustering raw filtered reads with singletons inflates spurious OTUs and noise
- The goal is closed-reference picking against a curated database rather than de novo clustering/denoising from the sample data itself

## Inputs

- temp/filtered.fa — length/quality-filtered amplicon FASTA (e.g. vsearch --fastx_filter with --fastq_minlen 1200 --fastq_maxlen 1800 and --fastq_maxee)
- temp/uniques.fa — dereplicated unique sequences with abundance annotations (size=), relabeled with Uni_ prefix

## Outputs

- temp/otus.fa — representative-sequence FASTA with headers prefixed OTU_ or ASV_
- zotus.fa (intermediate) — denoised zero-radius OTU sequences before Zotu→ASV_ relabeling
- temp/otus_raw.fa (large-dataset VSEARCH route) — cluster centroids before chimera filtering
- Log-reported chimera counts removed by clustering/denoising

## How to apply

First dereplicate temp/filtered.fa with vsearch --derep_fulllength using --minuniquesize 10, --sizeout, and --relabel Uni_ to write abundance-annotated unique sequences to temp/uniques.fa. Then choose one of two routes: (a) 97% OTU clustering with usearch -cluster_otus temp/uniques.fa -minsize 10 -relabel OTU_, which includes de novo chimera filtering, or (b) ASV denoising with usearch -unoise3 temp/uniques.fa -minsize 10 to produce zotus.fa, then relabel 'Zotu' to 'ASV_' with sed to obtain temp/otus.fa. For very large datasets, substitute the VSEARCH route: vsearch --cluster_size at --id 0.97 to obtain temp/otus_raw.fa centroids, followed by vsearch --uchime_denovo to keep non-chimeras as temp/otus.fa. Reference-based dechimerization (e.g. vsearch --uchime_ref against RDP rdp_16s_v18) is an alternative to the default de novo approach. The de novo chimera filtering built into cluster_otus is the pipeline default.

## Related tools

- **USEARCH** (Performs dereplication-adjacent denoising (-unoise3) and de novo 97% OTU clustering with built-in chimera filtering (-cluster_otus -minsize 10 -relabel OTU_)) — https://github.com/YongxinLiu/EasyAmplicon
- **VSEARCH** (Open-source dereplication (--derep_fulllength --minuniquesize 10 --sizeout --relabel Uni_) and alternative large-dataset clustering route (--cluster_size --id 0.97 plus --uchime_denovo)) — torognes/vsearch
- **EasyAmplicon** (Pipeline that orchestrates the dereplication → clustering/denoising → temp/otus.fa workflow steps) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
usearch -cluster_otus temp/uniques.fa -minsize 10 -relabel OTU_ -output temp/otus.fa
```

## Evaluation signals

- temp/otus.fa exists and is valid FASTA with headers carrying the OTU_ or ASV_ prefix (Uni_ prefix and size= abundance annotations retained in temp/uniques.fa)
- The clustering/denoising command reports the number of chimeras removed; de novo dechimerization via cluster_otus is included by default
- Every sequence in temp/uniques.fa has abundance annotation (sizeout) and passed the minuniquesize/minsize 10 threshold
- Denoised ASV route: sequence names were relabeled from 'Zotu' to 'ASV_' (no 'Zotu' prefixes remain in temp/otus.fa)
- Large-dataset VSEARCH route: non-chimera centroids selected by vsearch --uchime_denovo are written to temp/otus.fa

## Limitations

- USEARCH's default heuristic seed-and-extend aligner can yield less accurate alignments than VSEARCH's full Needleman-Wunsch global aligner, which may affect clustering sensitivity
- USEARCH requires licensing considerations and has memory limits on very large databases; VSEARCH's 64-bit design handles much more than 4GB of memory
- De novo dechimerization (cluster_otus) is the default; reference-based dechimerization requires an appropriate database such as RDP rdp_16s_v18
- The minsize 10 / minuniquesize 10 abundance threshold discards rare unique sequences, which may exclude genuinely low-abundance taxa

## Evidence

- [methods] Dereplication (derep_fulllength): "Use tools like **VSEARCH** to merge identical sequences into a single one to reduce redundant calculations"
- [methods] Chimera removal (uchime_ref / de novo): "By default, de novo dechimerization is used (as described above with `cluster_otus`). Reference-based dechimerization can also be performed"
- [readme] VSEARCH capabilities: [evidence span withheld — anchor status: external_doc]
- [readme] VSEARCH accuracy rationale: [evidence span withheld — anchor status: external_doc]
- [abstract] Pipeline scope: "including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis, diversity analysis, and data visualization"
