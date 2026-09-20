---
name: least-common-ancestor-taxonomy-resolution
description: Use when merging taxonomic labels from multiple identical or near-identical
  COI reference records, or when classifier output provides multiple hits without
  coverage metadata and a conservative single lineage per sequence is needed.
when_to_use_negative:
- When each reference sequence already carries a single, complete, non-redundant lineage
  (e.g., a curated RDP-formatted training set) — consensus collapsing adds no information.
- When the goal is maximal species-level resolution rather than a conservative assignment
  — LCA will truncate to the deepest rank at which all member taxonomies agree.
- When headers or classifier hits lack comparable rank-delimited taxonomy strings,
  since the consensus script cannot reconcile incompatible lineage formats.
edam_operation: http://edamontology.org/operation_0454
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: VSEARCH
  role: Collapses identical reference sequences with --derep_fulllength to produce
    the distinct-sequence fasta that receives the consensus taxonomy label
  repo: https://github.com/torognes/vsearch
- name: create_consensus_taxonomy.py
  role: Walterst/Walters consensus-LCA script applied to pick_otus.py clusters to
    derive the least-common-ancestor taxonomy per cluster
  repo: https://gist.github.com/walterst/bd69a19e75748f79efeb
- name: QIIME1 pick_otus.py
  role: Clusters identical sequences at --similarity 1.0 prior to consensus taxonomy
    assignment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:fe127aec4bf42434ff51da721ae3ecabc1800527a5d53c35cca8ff9627e8bc66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - QIIME1 pick_otus.py
  - R
  - VSEARCH
  - create_consensus_taxonomy.py
  - csvtk
  - seqkit
  edam_operation: http://edamontology.org/operation_0454
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# Least-common-ancestor taxonomy resolution

## Summary

Applies an LCA / consensus-taxonomy approach to resolve taxonomic strings for dereplicated COI reference sequences (e.g., Porter GenBank-derived records clustered at 100% identity) and to classify metabarcoding hits (e.g., BOLD API results lacking query coverage), yielding a single consensus lineage per sequence or cluster.

## When to use

Use when merging taxonomic labels from multiple identical or near-identical COI reference records, or when classifier output provides multiple hits without coverage metadata and a conservative single lineage per sequence is needed.

## When NOT to use

- When each reference sequence already carries a single, complete, non-redundant lineage (e.g., a curated RDP-formatted training set) — consensus collapsing adds no information.
- When the goal is maximal species-level resolution rather than a conservative assignment — LCA will truncate to the deepest rank at which all member taxonomies agree.
- When headers or classifier hits lack comparable rank-delimited taxonomy strings, since the consensus script cannot reconcile incompatible lineage formats.

## Inputs

- COI reference fasta with taxonomy-annotated headers (e.g., Porter Eukaryote CO1 v3.2 training fasta from CO1Classifier release)
- OTU cluster output from pick_otus.py --similarity 1.0
- Dereplicated fasta from vsearch --derep_fulllength
- Classifier hit lists lacking query coverage (e.g., BOLD Classification API output)

## Outputs

- Consensus (LCA) taxonomy string per dereplicated sequence or cluster
- Rank-restricted taxa tables (Class-through-Species): palmer.taxa.txt.gz, porter.taxa.txt.gz, derep.taxa.txt.gz
- Dereplicated reference set (e.g., 515,780 distinct Porter arthropod sequences)

## How to apply

First dereplicate the reference fasta: strip taxonomy from headers, cluster identical sequences with QIIME1 pick_otus.py --similarity 1.0, then collapse with vsearch --derep_fulllength. Apply the create_consensus_taxonomy.py script (Walterst gist implementing Tony Walters' consensus approach, adapted from Mike Robeson's SILVA_to_RDP formatting instructions) to assign each cluster the least-common-ancestor taxonomy across its member records. Extract Class-through-Species strings (discarding redundant kingdom/phylum fields) into taxa tables such as palmer.taxa.txt.gz, porter.taxa.txt.gz, and derep.taxa.txt.gz. The LCA is also applied to BOLD Classification API hits: because query coverage is not returned, an LCA process is run over the remaining hits. Rationale: GenBank-derived records carry inconsistent or duplicated lineages, so a consensus label prevents over-precise assignments when member taxonomies disagree.

## Related tools

- **VSEARCH** (Collapses identical reference sequences with --derep_fulllength to produce the distinct-sequence fasta that receives the consensus taxonomy label) — https://github.com/torognes/vsearch
- **create_consensus_taxonomy.py** (Walterst/Walters consensus-LCA script applied to pick_otus.py clusters to derive the least-common-ancestor taxonomy per cluster) — https://gist.github.com/walterst/bd69a19e75748f79efeb
- **QIIME1 pick_otus.py** (Clusters identical sequences at --similarity 1.0 prior to consensus taxonomy assignment)

## Examples

```
vsearch --derep_fulllength tmp_nolabels.fasta
```

## Evaluation signals

- Dereplicated Porter arthropod set contains 515,780 distinct sequences after pick_otus.py --similarity 1.0 plus vsearch --derep_fulllength.
- Upset plot (figure4_upsetPlot.png) shows both shared and unique taxonomic names at Class-through-Species ranks for Porter, Palmer, and tidybug (BOLD) databases, consistent with 4,251 Porter-only and 3,885 tidybug-only genera among 12,788 shared genera.
- Every dereplicated cluster carries exactly one consensus lineage, with no kingdom/phylum fields retained in the taxa tables.
- LCA over BOLD API hits resolves a single lineage where hits disagree; unmatched records remain labeled rather than force-assigned.
- Record counts after filtering match expectations (Palmer: 1,617,885 to 1,565,831 after seqkit grep 'p:Arthropoda'; Porter: 1,280,577 to 883,979 after grep ';Arthropoda;').

## Limitations

- The BOLD Classification API's reference database content and classification parameters are undocumented, so it is unclear what drives missing assignments or whether the tidybug database matched the API's references.
- No attempt was made to determine how pervasive missing Phylum/Class information is among records, so LCA inputs may carry hidden rank-level missingness.
- There is no single curated and versioned COI reference database resource, and BOLD discontinued packaged versioned releases in 2015, limiting reproducibility of consensus-taxonomy inputs.

## Evidence

- [methods] incorporated the [consensus approach to reclassifying taxa](https://gist.github.com/walterst/9ddb926fece4b7c0e12c) script written by William (Tony) Walters: "incorporated the [consensus approach to reclassifying taxa](https://gist.github.com/walterst/9ddb926fece4b7c0e12c) script written by William (Tony) Walters"
- [methods] Because query coverage was not a parameter that was provided in the output, we applied an LCA process to those remaining hits: "Because query coverage was not a parameter that was provided in the output, we applied an LCA process to those remaining hits"
- [methods] all sequences that lacked Family and/or Order information were discarded: "all sequences that lacked Family and/or Order information were discarded"
- [readme] Complete details including scripts and data files used in this manuscript are available at the GitHub repo for this project: https://github.com/devonorourke/tidybug: "Complete details including scripts and data files used in this manuscript are available at the GitHub repo for this project: https://github.com/devonorourke/tidybug"
- [readme] This repository contains training sets that can be used with the Ribosomal Database Project classifier (Wang et al., 2007) to taxonomically assign Eukaryote CO1 mtDNA sequences.: [evidence span withheld — anchor status: not_found]
