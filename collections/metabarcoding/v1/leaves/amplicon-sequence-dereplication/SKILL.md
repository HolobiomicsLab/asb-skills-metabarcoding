---
name: amplicon-sequence-dereplication
description: Use when filtered/merged amplicon reads (e.g., temp/filtered.fa) must
  be de-redundized and reduced to a representative OTU or ASV set before building
  an OTU table.
when_to_use_negative:
- Input is already a feature/OTU table or already-dereplicated representative sequences
  (e.g., result/raw/otus.fa) — dereplication and selection have already been performed.
- Reference-based chimera checking is required instead of de novo; that step uses
  vsearch --uchime_ref against databases like rdp_16s_v18 and is a separate downstream
  filter step.
- Working with long-read (PacBio/Nanopore) amplicon data, which EasyAmplicon 2 handles
  with a different pipeline.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: VSEARCH
  role: Performs --derep_fulllength dereplication with --minuniquesize 10, --sizeout,
    --relabel Uni_; for large datasets substitutes --cluster_size --id 0.97 and --uchime_denovo
  repo: https://github.com/torognes/vsearch
- name: USEARCH
  role: Runs -unoise3 -minsize 10 denoising or -cluster_otus -minsize 10 -relabel
    OTU_ clustering (with de novo chimera filtering) to produce temp/otus.fa
- name: EasyAmplicon
  role: Pipeline that orchestrates the dereplication and OTU/ASV selection steps and
    defines the temp/ file naming convention
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:78b2a38841e82354ac904739b166519a94546357ff15ac65b968b4f90f57a0a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# amplicon-sequence-dereplication

## Summary

Reduces redundancy in filtered amplicon reads by collapsing identical full-length sequences into unique sequences with abundance annotations, then selects representative OTU/ASV sequences via 97% de novo clustering (usearch -cluster_otus) or denoising (usearch -unoise3), writing the representative set to temp/otus.fa.

## When to use

Use when filtered/merged amplicon reads (e.g., temp/filtered.fa) must be de-redundized and reduced to a representative OTU or ASV set before building an OTU table.

## When NOT to use

- Input is already a feature/OTU table or already-dereplicated representative sequences (e.g., result/raw/otus.fa) — dereplication and selection have already been performed.
- Reference-based chimera checking is required instead of de novo; that step uses vsearch --uchime_ref against databases like rdp_16s_v18 and is a separate downstream filter step.
- Working with long-read (PacBio/Nanopore) amplicon data, which EasyAmplicon 2 handles with a different pipeline.

## Inputs

- temp/filtered.fa — filtered amplicon sequences in FASTA format
- temp/uniques.fa — dereplicated unique sequences with abundance annotations (--sizeout), relabeled with Uni_ prefix

## Outputs

- temp/uniques.fa — unique sequences with size annotations (from dereplication)
- temp/otus.fa — representative ASV/OTU sequences in FASTA format with ASV_ or OTU_ headers

## How to apply

First dereplicate temp/filtered.fa with vsearch --derep_fulllength using --minuniquesize 10, --sizeout, and --relabel Uni_ to write unique sequences with abundance annotations to temp/uniques.fa. Then choose one of two routes: (1) ASV denoising — run usearch -unoise3 temp/uniques.fa with -minsize 10 to generate zotus.fa, then relabel the 'Zotu' prefixes to 'ASV_' via sed to obtain temp/otus.fa; or (2) 97% OTU clustering — run usearch -cluster_otus temp/uniques.fa with -minsize 10 and -relabel OTU_, which includes de novo chimera filtering and directly yields temp/otus.fa. For very large datasets, substitute the VSEARCH route: vsearch --cluster_size --id 0.97 to get temp/otus_raw.fa centroids, then vsearch --uchime_denovo to keep non-chimeras as temp/otus.fa. The minsize 10 threshold ensures low-abundance uniques (potential sequencing errors) do not seed OTUs/ASVs.

## Related tools

- **VSEARCH** (Performs --derep_fulllength dereplication with --minuniquesize 10, --sizeout, --relabel Uni_; for large datasets substitutes --cluster_size --id 0.97 and --uchime_denovo) — https://github.com/torognes/vsearch
- **USEARCH** (Runs -unoise3 -minsize 10 denoising or -cluster_otus -minsize 10 -relabel OTU_ clustering (with de novo chimera filtering) to produce temp/otus.fa)
- **EasyAmplicon** (Pipeline that orchestrates the dereplication and OTU/ASV selection steps and defines the temp/ file naming convention) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
vsearch --derep_fulllength temp/filtered.fa --minuniquesize 10 --sizeout --relabel Uni_ --output temp/uniques.fa && usearch -cluster_otus temp/uniques.fa -minsize 10 -relabel OTU_ -otus temp/otus.fa
```

## Evaluation signals

- temp/otus.fa exists and is valid FASTA with sequence headers carrying the ASV_ or OTU_ prefix.
- temp/uniques.fa headers carry the Uni_ prefix and abundance (size=) annotations from --sizeout.
- The clustering/denoising command reports the number of chimeras removed.
- ASV route: Zotu labels in zotus.fa are successfully relabeled to ASV_ in temp/otus.fa (sed step verified).
- No unique sequence with abundance below minuniquesize/minsize 10 is retained as an OTU/ASV seed.

## Limitations

- USEARCH -unoise3 and -cluster_otus are memory-limited; for very large datasets the VSEARCH route (--cluster_size --id 0.97 plus --uchime_denovo) is substituted instead.
- OTU clustering via -cluster_otus includes only de novo chimera filtering; reference-based dechimerization (vsearch --uchime_ref with e.g. rdp_16s_v18) is a separate downstream step.
- The minsize 10 threshold discards rare uniques, so singletons/doubletons cannot become OTUs/ASVs.

## Evidence

- [full_text] Dereplicate temp/filtered.fa with vsearch --derep_fulllength using --minuniquesize 10, --sizeout, and --relabel Uni_ to write unique sequences with abundance annotations to temp/uniques.fa.: [evidence span withheld — anchor status: not_found]
- [full_text] ASV denoising route with minsize 10 and relabeling from Zotu to ASV_.: [evidence span withheld — anchor status: not_found]
- [full_text] OTU clustering route with de novo chimera filtering included.: [evidence span withheld — anchor status: not_found]
- [full_text] VSEARCH alternative route for very large datasets.: [evidence span withheld — anchor status: not_found]
- [intro] VSEARCH is the open-source tool used for clustering, chimera detection, and dereplication in the pipeline.: "VSEARCH v2.7.1+52 or later: VSEARCH is an open‐source tool for performing tasks such as sequence clustering, chimera detection, and dereplication."
- [methods] Article confirms dereplication is performed with VSEARCH to merge identical sequences and reduce redundant calculations.: "Use tools like **VSEARCH** to merge identical sequences into a single one to reduce redundant calculations"
- [readme] README confirms EasyAmplicon integrates USEARCH and VSEARCH from raw data to results.: "in 2023 we developed EasyAmplicon, a comprehensive, user‐friendly pipeline that integrates popular tools such as USEARCH and VSEARCH"
