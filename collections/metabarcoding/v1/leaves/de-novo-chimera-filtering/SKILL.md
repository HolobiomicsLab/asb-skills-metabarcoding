---
name: de-novo-chimera-filtering
description: Use when dereplicated amplicon sequences (temp/uniques.fa from vsearch
  --derep_fulllength with --minuniquesize 10 and --sizeout) must be screened for chimeras
  before building OTUs or ASVs, and no suitable reference chimera database is available
  or de novo mode is preferred.
when_to_use_negative:
- A curated reference database (e.g., rdp_16s_v18.fa) is available and reference-based
  dechimerization (vsearch --uchime_ref) is preferred — the article describes both
  modes and reference-based screening can be more sensitive for known chimeras
- Input is raw or merged reads that have not been quality filtered and dereplicated
  — chimera detection requires abundance-annotated unique sequences with -minsize
  10 support
- The goal is taxonomy assignment or OTU table construction rather than chimera removal
  of representative sequences
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: USEARCH
  role: Runs -cluster_otus with -minsize 10 and -relabel OTU_, performing de novo
    chimera filtering bundled with 97% OTU clustering to produce temp/otus.fa
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: VSEARCH
  role: 'Open-source alternative for large datasets: --cluster_size at --id 0.97 gives
    centroids, then --uchime_denovo removes chimeras; the paper notes VSEARCH is an
    open-source tool for sequence clustering and chimera detection'
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:2317d6cd63236a4647079c1bb3ce3c4f712f44e06b988bef88db02b05432f640
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - USEARCH
  - VSEARCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# de-novo-chimera-filtering

## Summary

Removes PCR chimeric amplicon sequences without a reference database, as performed in the EasyAmplicon pipeline either implicitly during 97% OTU clustering with USEARCH's -cluster_otus or explicitly on VSEARCH cluster centroids with --uchime_denovo, to yield the non-chimeric representative-sequence file temp/otus.fa.

## When to use

Use when dereplicated amplicon sequences (temp/uniques.fa from vsearch --derep_fulllength with --minuniquesize 10 and --sizeout) must be screened for chimeras before building OTUs or ASVs, and no suitable reference chimera database is available or de novo mode is preferred.

## When NOT to use

- A curated reference database (e.g., rdp_16s_v18.fa) is available and reference-based dechimerization (vsearch --uchime_ref) is preferred — the article describes both modes and reference-based screening can be more sensitive for known chimeras
- Input is raw or merged reads that have not been quality filtered and dereplicated — chimera detection requires abundance-annotated unique sequences with -minsize 10 support
- The goal is taxonomy assignment or OTU table construction rather than chimera removal of representative sequences

## Inputs

- temp/uniques.fa — dereplicated full-length amplicon sequences with abundance annotations (vsearch --derep_fulllength --minuniquesize 10 --sizeout --relabel Uni_)
- temp/otus_raw.fa — VSEARCH cluster centroids (vsearch --cluster_size --id 0.97), when using the VSEARCH alternative route

## Outputs

- temp/otus.fa — representative non-chimeric OTU/ASV sequences in FASTA format with OTU_/ASV_ prefixes
- Command-line report of the number of chimeras detected and removed

## How to apply

Start from the dereplicated, abundance-annotated FASTA (temp/uniques.fa, sequences relabeled Uni_ with --sizeout). Primary route: run USEARCH 'usearch -cluster_otus temp/uniques.fa -minsize 10 -relabel OTU_' — this command includes de novo chimera filtering by default and writes representative non-chimeric OTUs directly to temp/otus.fa. For very large datasets, substitute the VSEARCH route: cluster with 'vsearch --cluster_size ... --id 0.97' to obtain centroid sequences (temp/otus_raw.fa), then run 'vsearch --uchime_denovo' on the centroids and keep only non-chimeras as temp/otus.fa. The article notes de novo dechimerization is the default; reference-based screening (e.g., vsearch --uchime_ref against rdp_16s_v18.fa) is an alternative when a curated database is appropriate. Rationale: de novo mode detects chimeras from abundance and sequence relationships within the sample set itself, matching the pipeline's default cluster_otus behavior.

## Related tools

- **USEARCH** (Runs -cluster_otus with -minsize 10 and -relabel OTU_, performing de novo chimera filtering bundled with 97% OTU clustering to produce temp/otus.fa) — https://github.com/YongxinLiu/EasyAmplicon
- **VSEARCH** (Open-source alternative for large datasets: --cluster_size at --id 0.97 gives centroids, then --uchime_denovo removes chimeras; the paper notes VSEARCH is an open-source tool for sequence clustering and chimera detection) — https://github.com/torognes/vsearch

## Examples

```
usearch -cluster_otus temp/uniques.fa -minsize 10 -relabel OTU_ -otus temp/otus.fa
```

## Evaluation signals

- temp/otus.fa exists and is a valid FASTA file after the de novo chimera-filtering step
- Sequence headers in temp/otus.fa carry the OTU_ (or ASV_ after Unoise3 relabeling) prefix
- The clustering/denoising command reports the number of chimeras removed; verify it is non-zero-plausible and logged
- Downstream reference dechimerization (vsearch --uchime_ref temp/otus.fa --db ${db}/usearch/rdp_16s_v18.fa --nonchimeras result/raw/otus.fa) should flag few or no additional chimeras if de novo filtering worked
- For the VSEARCH route, sequences in temp/otus.fa are a subset of temp/otus_raw.fa centroids (only non-chimeras retained)

## Limitations

- De novo dechimerization is the pipeline default, but reference-based dechimerization is also supported and may be preferable when a suitable reference database exists
- USEARCH's -cluster_otus requires -minsize 10 abundance support, so low-abundance chimeras/OTUs below this threshold are discarded along with genuine rare sequences
- No changelog or limitations section is provided for the protocol repository, so version-specific behavioral changes are not documented here

## Evidence

- [methods] Default de novo dechimerization with cluster_otus: "By default, de novo dechimerization is used (as described above with `cluster_otus`). Reference-based dechimerization can also be performed"
- [readme] VSEARCH de novo chimera detection capability: [evidence span withheld — anchor status: external_doc]
- [intro] VSEARCH as open-source chimera detection tool: "VSEARCH is an open‐source tool for performing tasks such as sequence clustering, chimera detection, and dereplication."
