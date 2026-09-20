---
name: sintax-taxonomic-annotation
description: Use when a feature-table pipeline needs lineage assignments for OTU/ASV
  representative sequences (result/raw/otus.fa) so that plastid, mitochondrial, archaeal,
  and eukaryotic contamination can be flagged and removed.
when_to_use_negative:
- Input is already an annotated feature table or BIOM file — re-running SINTAX would
  duplicate existing lineage information
- Marker is not 16S rRNA (e.g., fungal ITS), where the RDP 16S v18 training set is
  inappropriate and UNITE should be used instead
- OTU/ASV reference sequences still contain chimeras — annotate only after uchime_ref/cluster_otus
  has produced result/raw/otus.fa
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: VSEARCH
  role: Runs the --sintax classification of OTU/ASV representative sequences against
    the RDP 16S v18 database with --sintax_cutoff 0.1
  repo: https://github.com/torognes/vsearch
- name: USEARCH
  role: Origin of the SINTAX command syntax that VSEARCH reproduces as a drop-in replacement
    for taxonomy assignment
- name: EasyMicrobiome
  role: Provides the RDP 16S v18 reference database and the otutab_filter_nonBac.R
    script that consumes the sintax output
  repo: https://github.com/YongxinLiu/EasyMicrobiome
- name: EasyAmplicon
  role: Pipeline wrapper that sequences the SINTAX annotation step between chimera
    removal and non-bacterial filtering
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:bfc97306c8ed5135756c3b537e381e6f5b054c218fbc5c76d99baf60906869a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sintax-taxonomic-annotation

## Summary

Annotates OTU/ASV representative sequences with taxonomy using VSEARCH's SINTAX classifier against the RDP 16S v18 database at a bootstrap cutoff of 0.1, producing the tab-deligned result/raw/otus.sintax file consumed downstream by the non-bacterial filtering step in EasyAmplicon.

## When to use

Use when a feature-table pipeline needs lineage assignments for OTU/ASV representative sequences (result/raw/otus.fa) so that plastid, mitochondrial, archaeal, and eukaryotic contamination can be flagged and removed.

## When NOT to use

- Input is already an annotated feature table or BIOM file — re-running SINTAX would duplicate existing lineage information
- Marker is not 16S rRNA (e.g., fungal ITS), where the RDP 16S v18 training set is inappropriate and UNITE should be used instead
- OTU/ASV reference sequences still contain chimeras — annotate only after uchime_ref/cluster_otus has produced result/raw/otus.fa

## Inputs

- OTU/ASV representative sequences (result/raw/otus.fa) produced after chimera removal
- RDP 16S v18 reference taxonomy database (${db}/usearch/rdp_16s_v18.fa from EasyMicrobiome)
- SINTAX bootstrap confidence cutoff (--sintax_cutoff 0.1)

## Outputs

- Tabbed SINTAX taxonomy file result/raw/otus.sintax with lineage strings and bootstrap confidences per OTU/ASV

## How to apply

After chimera removal yields the OTU/ASV reference (result/raw/otus.fa), classify each representative sequence with vsearch --sintax against the Ribosomal Database Project 16S training set (rdp_16s_v18.fa, located in the EasyMicrobiome database folder under ${db}/usearch/). Set --sintax_cutoff 0.1, the bootstrap support threshold below which ranks are left unassigned, and write the tabbed taxonomy output to result/raw/otus.sintax. VSEARCH is used here because it is the open-source drop-in replacement for USEARCH commands, using an optimal global (Needleman-Wunsch) aligner for sensitivity. The resulting sintax annotations are then paired with the raw feature table (result/raw/otutab.txt) as inputs to the EasyMicrobiome script otutab_filter_nonBac.R, so the annotation quality directly determines which lineages (Eukarya, Chloroplasts, Mitochondria, Archaea) get discarded. The RDP database is the default in EasyMicrobiome; SILVA, UNITE, and Greengenes are alternative pre-packaged reference options for other marker types.

## Related tools

- **VSEARCH** (Runs the --sintax classification of OTU/ASV representative sequences against the RDP 16S v18 database with --sintax_cutoff 0.1) — https://github.com/torognes/vsearch
- **USEARCH** (Origin of the SINTAX command syntax that VSEARCH reproduces as a drop-in replacement for taxonomy assignment)
- **EasyMicrobiome** (Provides the RDP 16S v18 reference database and the otutab_filter_nonBac.R script that consumes the sintax output) — https://github.com/YongxinLiu/EasyMicrobiome
- **EasyAmplicon** (Pipeline wrapper that sequences the SINTAX annotation step between chimera removal and non-bacterial filtering) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
vsearch --sintax result/raw/otus.fa --db ${db}/usearch/rdp_16s_v18.fa --sintax_cutoff 0.1 --tabbedout result/raw/otus.sintax
```

## Evaluation signals

- result/raw/otus.sintax exists, is tab-delimited, and contains one lineage line per sequence in result/raw/otus.fa (row counts match)
- Taxonomy strings follow SINTAX format with rank prefixes and confidence values; ranks below --sintax_cutoff 0.1 bootstrap are unassigned
- Downstream otutab_filter_nonBac.R successfully parses the sintax file and produces result/raw/otus.sintax.discard listing Eukarya/Chloroplast/Mitochondria/Archaea lineages
- csvtk -t stat on the cleaned result/otutab.txt shows the number of columns equals the number of samples in the metadata

## Limitations

- The 0.1 bootstrap cutoff is permissive; low-confidence assignments may propagate into the non-bacterial filter and misclassify borderline lineages
- RDP 16S v18 is specific to bacterial/archaeal 16S — ITS or eukaryotic amplicons need alternative databases (UNITE, SILVA)
- VSEARCH does not support amino acid sequences or local alignments, so classification is limited to nucleotide global-alignment-based SINTAX

## Evidence

- [methods] Annotate the OTU sequences with vsearch --sintax against the RDP 16S v18 database (${db}/usearch/rdp_16s_v18.fa) using --sintax_cutoff 0.1: [evidence span withheld — anchor status: not_found]
- [intro] RDP is located by default in the EasyMicrobiome database folder: "The Ribosomal Database Project (RDP) [38] is located by default in the EasyMicrobiome database folder."
- [methods] otutab_filter_nonBac.R filters out Eukarya, Chloroplasts, Mitochondria, and Archaea: "**Remove Chloroplasts, Mitochondria, and Non-Bacteria**: Typically, an R script (e.g., `otutab_filter_nonBac.R`) is used to filter out Eukarya, Chloroplasts, Mitochondria, and Archaea"
- [readme] VSEARCH is an open-source alternative to USEARCH with more accurate global alignments: [evidence span withheld — anchor status: external_doc]
- [methods] vsearch --uchime_ref temp/otus.fa --db ${db}/usearch/rdp_16s_v18.fa --nonchimeras result/raw/otus.fa: "vsearch --uchime_ref temp/otus.fa --db ${db}/usearch/rdp_16s_v18.fa --nonchimeras result/raw/otus.fa"
