---
name: vsearch-dereplication-and-clustering
description: Use when building a custom COI reference database from BOLD/GenBank-derived
  FASTA records and you must dereplicate identical sequences, curate taxonomy via
  LCA consensus, and assess how a clustering radius (e.g., 99% identity) changes reference
  composition.
when_to_use_negative:
- Your goal is denoising amplicon reads into ASVs (e.g., DADA2/Deblur on raw FASTQ)
  rather than constructing a reference database — that is a separate tidybug workflow
  step
- Your sequences are amino acid translations or you need local alignment — the VSEARCH
  README states it does not support amino acid sequences or local alignments
- You only need to classify queries against an existing database (e.g., vsearch-SINTAX
  or q2-feature-classifier BLAST+LCA) rather than build one
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: seqkit
  role: Filters the BOLD FASTA for Family-level information via seqkit grep --pattern-file
    idsNoFam.txt --invert-match before dereplication
- name: QIIME1 pick_otus.py
  role: Groups identical (--similarity 1.0) sequences into pid100_otus prior to LCA
    consensus curation
  repo: https://github.com/devonorourke/tidybug
- name: create_consensus_taxonomy.py
  role: Applies the Walters consensus LCA approach to resolve conflicting taxonomies
    among identical dereplicated sequences
  repo: https://gist.github.com/walterst/bd69a19e75748f79efeb
- name: VSEARCH
  role: Performs full-length dereplication (--derep_fulllength --relabel_keep --notrunclabels)
    and 99%-identity size-based clustering (--cluster_size --id 0.99 --strand both)
    of the reference set
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:833b7a9f07e1dff6938813c5d1983ef111b54e9309819b8a1c9b70ecf1177265
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/devonorourke/tidybug
  tools:
  - QIIME 2
  - QIIME1 pick_otus.py
  - VSEARCH
  - create_consensus_taxonomy.py
  - seqkit
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.125
n_claims: 8
---

# vsearch-dereplication-and-clustering

## Summary

Construct a non-redundant arthropod COI reference database by dereplicating BOLD-derived FASTA records with VSEARCH --derep_fulllength, resolving conflicting taxonomies among identical sequences with an LCA consensus script, then clustering dereplicated centroids at 99% identity with vsearch --cluster_size to quantify how clustering radius shrinks the reference set (1,841,946 → 407,356 records) while raising ambiguous species-name fractions (5.6% → 7.2%).

## When to use

Use when building a custom COI reference database from BOLD/GenBank-derived FASTA records and you must dereplicate identical sequences, curate taxonomy via LCA consensus, and assess how a clustering radius (e.g., 99% identity) changes reference composition.

## When NOT to use

- Your goal is denoising amplicon reads into ASVs (e.g., DADA2/Deblur on raw FASTQ) rather than constructing a reference database — that is a separate tidybug workflow step
- Your sequences are amino acid translations or you need local alignment — the VSEARCH README states it does not support amino acid sequences or local alignments
- You only need to classify queries against an existing database (e.g., vsearch-SINTAX or q2-feature-classifier BLAST+LCA) rather than build one

## Inputs

- BOLD-derived arthropod COI FASTA file (e.g., boldCustom.allArth.filt2.fasta)
- Pattern file of sequence IDs to invert-match (idsNoFam.txt) for Family-level filtering
- Two-column taxonomy mapping file (tmp_nolabels.taxa) parsed from the filtered FASTA
- OTU membership table from pick_otus.py (tmp_nolabels_otus.txt) for LCA consensus curation

## Outputs

- Filtered FASTA with Family-level taxonomy retained (boldCustom.allArth.filt3.fasta; 3,051,814 records)
- Consensus LCA taxonomy map for identical sequences (outmap.txt; pid99_outmap.txt for 99% centroids)
- Dereplicated reference FASTA (boldCOI.derep.fasta; 1,841,946 unique records) and rank-prefixed taxonomy file (boldCOI.arth_derep.txt)
- 99%-identity centroid FASTA (boldCOI.clust99.fasta; 407,356 centroids)

## How to apply

First discard sequences lacking Family-level information with seqkit grep --pattern-file idsNoFam.txt --invert-match (regex 'f__;') to yield boldCustom.allArth.filt3.fasta (3,051,814 records), then split it into a two-column taxonomy mapping file (tmp_nolabels.taxa) and a header-stripped FASTA. Group identical sequences with QIIME1 pick_otus.py --similarity 1.0, then run the Walters create_consensus_taxonomy.py LCA script to resolve conflicting taxonomies among identical sequences (outmap.txt). Dereplicate with vsearch --derep_fulllength --relabel_keep --notrunclabels to obtain boldCOI.derep.fasta (1,841,946 unique records), prefix outmap.txt entries with k__/p__/c__/o__/f__/g__/s__ rank tokens to form boldCOI.arth_derep.txt, and finally cluster with vsearch --cluster_size --id 0.99 --strand both --threads 24 to produce boldCOI.clust99.fasta (407,356 centroids) plus an LCA map for the 99% centroids (pid99_outmap.txt). The rationale, per the article, is to measure how the clustering-radius filtering criterion affects reference composition — clustering reduces the number of representative sequences but conflates distinct species names, increasing taxonomic ambiguity.

## Related tools

- **seqkit** (Filters the BOLD FASTA for Family-level information via seqkit grep --pattern-file idsNoFam.txt --invert-match before dereplication)
- **QIIME1 pick_otus.py** (Groups identical (--similarity 1.0) sequences into pid100_otus prior to LCA consensus curation) — https://github.com/devonorourke/tidybug
- **create_consensus_taxonomy.py** (Applies the Walters consensus LCA approach to resolve conflicting taxonomies among identical dereplicated sequences) — https://gist.github.com/walterst/bd69a19e75748f79efeb
- **VSEARCH** (Performs full-length dereplication (--derep_fulllength --relabel_keep --notrunclabels) and 99%-identity size-based clustering (--cluster_size --id 0.99 --strand both) of the reference set) — https://github.com/torognes/vsearch

## Examples

```
vsearch --cluster_size boldCOI.derep.fasta --id 0.99 --threads 24 --strand both --centroids boldCOI.clust99.fasta
```

## Evaluation signals

- Record-count invariant chain: filtered FASTA retains 3,051,814 records → dereplicated FASTA contains exactly 1,841,946 unique sequences → 99% clustering yields 407,356 centroids
- LCA maps (outmap.txt, pid99_outmap.txt) headers match dereplicated/centroid FASTA headers one-to-one and carry k__/p__/c__/o__/f__/g__/s__ rank prefixes
- Ambiguity check: fraction of ambiguous species names should rise from 5.6% (dereplicated) to 7.2% after 99% clustering; a large deviation signals clustering or LCA misapplication
- VSEARCH run uses --strand both and --threads 24 for clustering, and --notrunclabels for dereplication, matching the documented parameters
- No centroid in boldCOI.clust99.fasta exceeds 99% identity to another centroid (vsearch cluster_size output self-consistency)

## Limitations

- Clustering at coarser radii (99%) increases taxonomic ambiguity — the fraction of ambiguous species names rises from 5.6% to 7.2% — so a smaller database comes at the cost of less-resolved labels
- There is no single curated and versioned reference resource widely used in animal diet studies, and BOLD discontinued packaged versioned releases in 2015, so reproducing the exact input set depends on the 24 February 2019 BOLD download date
- VSEARCH does not support amino acid sequences or local alignments, and validation counts (1,841,946 / 407,356) are specific to this BOLD arthropod COI snapshot and will differ for other downloads

## Evidence

- [intro] Thus, we examined how the effects of reference database choice and subsequent filtering criteria (clustering radius) on reference composition: "Thus, we examined how the effects of reference database choice and subsequent filtering criteria (clustering radius) on reference composition"
- [methods] We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id: "We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id"
- [methods] all sequences that lacked Family and/or Order information were discarded: "all sequences that lacked Family and/or Order information were discarded"
- [readme] It also supports FASTQ file analysis, filtering, conversion and merging of paired-end reads.: [evidence span withheld — anchor status: not_found]
