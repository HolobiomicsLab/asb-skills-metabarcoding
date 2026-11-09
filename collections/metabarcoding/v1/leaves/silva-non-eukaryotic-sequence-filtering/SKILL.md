---
name: silva-non-eukaryotic-sequence-filtering
description: Use when exact-matching 16S amplicon sequences (e.g., Deblur 90-bp V4
  tags) against SILVA and eukaryotic references would inflate or confound the fraction
  of recaptured reference sequences.
when_to_use_negative:
- When the query amplicons include eukaryotic 18S or ITS sequences, since excluding
  eukaryotic SILVA references would remove legitimate matches
- When the target reference is Greengenes 13.8, which already contains only bacterial/archaeal
  sequences and requires no eukaryote filter
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0080
tools:
- name: VSEARCH
  role: performs the global-alignment exact matching (--id 1.0) of tag sequences against
    the filtered non-eukaryotic SILVA reference set
- name: Deblur
  role: generates the exact-sequence 90-bp V4 tag sequences used as queries against
    the filtered SILVA reference
  repo: https://github.com/cuttlefishh/deblur
- name: Python
  role: scripted counting of reference hits and computation of recapture fractions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:3161e06b5974bd7ab0adbc8bf05dbb036489b82a10a4ada9a8bd837b143f6eea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Python
  - VSEARCH
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# SILVA non-eukaryotic sequence filtering

## Summary

Subsets the SILVA 128 reference database to non-eukaryotic (bacterial and archaeal) rRNA gene sequences before exact-sequence comparison, so that 16S amplicon matches (e.g., EMP Deblur 90-bp tags) are assessed against prokaryotic reference diversity only.

## When to use

Use when exact-matching 16S amplicon sequences (e.g., Deblur 90-bp V4 tags) against SILVA and eukaryotic references would inflate or confound the fraction of recaptured reference sequences.

## When NOT to use

- When the query amplicons include eukaryotic 18S or ITS sequences, since excluding eukaryotic SILVA references would remove legitimate matches
- When the target reference is Greengenes 13.8, which already contains only bacterial/archaeal sequences and requires no eukaryote filter

## Inputs

- SILVA 128 full-length 16S/18S rRNA reference sequences with taxonomy annotations
- Set of unique Deblur 90-bp V4 tag sequences (from the EMP observation table)

## Outputs

- Filtered SILVA 128 reference FASTA restricted to bacterial and archaeal (non-eukaryotic) taxa
- Fraction of non-eukaryotic SILVA references recaptured by at least one tag sequence (45% expected)

## How to apply

Obtain the SILVA 128 reference sequences and remove sequences annotated as eukaryotic, retaining bacterial and archaeal rRNA gene sequences. This mirrors the EMP comparison in which OTUs were picked closed-reference against Greengenes 13.8 or SILVA, and in which the SILVA subset was explicitly restricted to non-eukaryotic taxa. Feed the filtered reference set into a VSEARCH global-alignment search (--id 1.0) using the unique Deblur 90-bp tag sequences as queries, then count how many reference sequences are hit by at least one tag. Report the recaptured fraction alongside the Greengenes 13.8 result; validation requires reproducing the published value of 45% of SILVA references recaptured (46% for Greengenes). Rationale: because the EMP queries are 16S (bacterial/archaeal) tag sequences, eukaryotic SILVA entries can never be legitimately matched and would bias the recapture fraction downward.

## Related tools

- **VSEARCH** (performs the global-alignment exact matching (--id 1.0) of tag sequences against the filtered non-eukaryotic SILVA reference set)
- **Deblur** (generates the exact-sequence 90-bp V4 tag sequences used as queries against the filtered SILVA reference) — https://github.com/cuttlefishh/deblur
- **Python** (scripted counting of reference hits and computation of recapture fractions)

## Examples

```
vsearch --usearch_global emp_tags.fna --db silva128_non-euk.fasta --id 1.0 --blastnout hits.txt --strand both
```

## Evaluation signals

- The filtered reference FASTA contains no sequences annotated as eukaryotic taxa
- VSEARCH run at --id 1.0 against the filtered SILVA subset recaptures 45% of reference sequences
- 10% of tag sequences match at least one filtered SILVA reference (published expectation)
- Greengenes 13.8 counterpart run reports 46% recaptured, consistent with the paired analysis
- reference_recapture_fractions.csv contains all four fractions (reference recaptured and tag matched for both databases)

## Limitations

- Filtering quality depends entirely on SILVA 128 taxonomy annotations; misannotated eukaryotic or unclassified sequences can pass through
- Exact matching at 100% similarity captures only identical sequence regions, so the recapture fraction is a lower bound on true database overlap
- The 90-bp tag is a partial V4 sequence, so global-alignment hits against full-length references reflect substring coverage rather than full-length identity

## Evidence

- [methods] SILVA reference subset to non-eukaryotic taxa: [evidence span withheld — anchor status: not_found]
- [supplementary] Deblur tag sequence definition: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [methods] Reference databases used for OTU picking: [evidence span withheld — anchor status: not_found]
- [abstract] Exact sequences instead of clustered OTUs rationale: [evidence span withheld — anchor status: not_found]
