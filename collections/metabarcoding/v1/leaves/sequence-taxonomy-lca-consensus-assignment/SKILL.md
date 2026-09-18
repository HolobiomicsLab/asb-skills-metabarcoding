---
name: sequence-taxonomy-lca-consensus-assignment
description: Use when constructing a taxonomic reference database for COI metabarcoding
  from BOLD/GenBank FASTA + taxonomy tables, especially when duplicate sequences carry
  conflicting lineage strings and need LCA consensus labels before classifier training.
when_to_use_negative:
- Input is already a dereplicated, taxonomy-assigned feature table (e.g., a QIIME
  2 classifier-ready artifact) rather than raw BOLD records with conflicting lineage
  strings
- The goal is per-record BOLD/GenBank metadata retention — dereplication and 99% clustering
  collapse sequences to centroids and lose one-to-one record identity
- The marker is not COI or the study needs the BOLD Classification API directly; note
  the API's closed-source reference database and undefined parameters make it non-equivalent
  to this curated local database
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: seqkit
  role: grep --invert-match with a pattern file to discard sequences lacking Family-level
    information before taxonomy curation
  repo: https://github.com/devonorourke/tidybug
- name: QIIME1 pick_otus.py
  role: groups identical (--similarity 1.0) reference sequences so the LCA consensus
    can be computed among taxonomy conflicts
- name: create_consensus_taxonomy.py
  role: Walters consensus LCA script that resolves conflicting taxonomies among identical
    sequences into outmap.txt
  repo: https://gist.github.com/walterst/bd69a19e75748f79efeb
- name: VSEARCH
  role: --derep_fulllength dereplication and --cluster_size --id 0.99 --strand both
    clustering to produce reference centroids
  repo: https://github.com/torognes/vsearch
- name: bold (R package)
  role: downloads the arthropod COI records from BOLD that constitute the source database
  repo: https://github.com/ropensci/bold
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:7befb201dfb92344e23c3a50f2c5ea7d6cd0380b79fbe85ad4787858e47692a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/devonorourke/tidybug
  tools:
  - QIIME 2
  - QIIME1 pick_otus.py
  - VSEARCH
  - bold (R package)
  - create_consensus_taxonomy.py
  - seqkit
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# sequence-taxonomy-lca-consensus-assignment

## Summary

Build a curated arthropod COI reference database (BOLD-derived) by dereplicating full-length sequences, resolving taxonomic conflicts among identical and 99%-clustered sequences with a Walters-style lowest-common-ancestor (LCA) consensus script, and emitting centroids labeled with full k__/p__/c__/o__/f__/g__/s__ rank prefixes. Clustering at 99% shrinks the reference (1,841,946 dereplicated to 407,356 centroids) at the cost of more ambiguous species labels.

## When to use

Use when constructing a taxonomic reference database for COI metabarcoding from BOLD/GenBank FASTA + taxonomy tables, especially when duplicate sequences carry conflicting lineage strings and need LCA consensus labels before classifier training.

## When NOT to use

- Input is already a dereplicated, taxonomy-assigned feature table (e.g., a QIIME 2 classifier-ready artifact) rather than raw BOLD records with conflicting lineage strings
- The goal is per-record BOLD/GenBank metadata retention — dereplication and 99% clustering collapse sequences to centroids and lose one-to-one record identity
- The marker is not COI or the study needs the BOLD Classification API directly; note the API's closed-source reference database and undefined parameters make it non-equivalent to this curated local database

## Inputs

- BOLD-derived arthropod COI FASTA (boldCustom.allArth.filt2.fasta) with embedded taxonomy strings
- Pattern file of IDs lacking Family-level info (idsNoFam.txt) for invert-match filtering
- Two-column sequence-to-taxonomy mapping file (tmp_nolabels.taxa)
- Header-stripped FASTA of sequences (tmp_nolabels.fasta)
- OTU mapping file from 100%-identity picking (tmp_nolabels_otus.txt)

## Outputs

- Family-filtered FASTA (boldCustom.allArth.filt3.fasta; 3,051,814 records)
- Dereplicated reference FASTA (boldCOI.derep.fasta; 1,841,946 unique sequences)
- LCA consensus map (outmap.txt) and prefixed label table (boldCOI.arth_derep.txt with k__/p__/c__/o__/f__/g__/s__ prefixes)
- 99%-identity centroid FASTA (boldCOI.clust99.fasta; 407,356 centroids) with pid99_outmap.txt LCA labels

## How to apply

First discard BOLD records lacking Family/Order information (e.g., seqkit grep --pattern-file idsNoFam.txt --invert-match on boldCustom.allArth.filt2.fasta to yield 3,051,814 records in filt3). Split filt3 into a two-column taxonomy mapping (tmp_nolabels.taxa) and a header-stripped FASTA, then run QIIME1 pick_otus.py --similarity 1.0 to group identical sequences, and apply create_consensus_taxonomy.py (the Walters consensus LCA approach) to resolve taxonomy conflicts among identical sequences into outmap.txt. Dereplicate full-length with vsearch --derep_fulllength --relabel_keep --notrunclabels to obtain 1,841,946 unique records, and propagate the k__/p__/c__/o__/f__/g__/s__-prefixed LCA labels to the dereplicated headers. Finally, cluster the dereplicated FASTA with vsearch --cluster_size --id 0.99 --strand both (also at 97% and 95% for sensitivity comparisons) to produce 407,356 centroids, remapping the LCA consensus onto the 99% centroids (pid99_outmap.txt). Rationale: identical sequences in BOLD often disagree in rank assignment, so the consensus step prevents arbitrary label selection, while clustering trades reference size for increased taxonomic ambiguity (ambiguous species names rise from 5.6% dereplicated to 7.2% after 99% clustering).

## Related tools

- **seqkit** (grep --invert-match with a pattern file to discard sequences lacking Family-level information before taxonomy curation) — https://github.com/devonorourke/tidybug
- **QIIME1 pick_otus.py** (groups identical (--similarity 1.0) reference sequences so the LCA consensus can be computed among taxonomy conflicts)
- **create_consensus_taxonomy.py** (Walters consensus LCA script that resolves conflicting taxonomies among identical sequences into outmap.txt) — https://gist.github.com/walterst/bd69a19e75748f79efeb
- **VSEARCH** (--derep_fulllength dereplication and --cluster_size --id 0.99 --strand both clustering to produce reference centroids) — https://github.com/torognes/vsearch
- **bold (R package)** (downloads the arthropod COI records from BOLD that constitute the source database) — https://github.com/ropensci/bold

## Examples

```
vsearch --cluster_size boldCOI.derep.fasta --id 0.99 --centroids boldCOI.clust99.fasta --strand both --threads 24 --notrunclabels
```

## Evaluation signals

- Family-filtered FASTA retains exactly 3,051,814 records
- Dereplicated FASTA contains 1,841,946 unique full-length sequences after vsearch --derep_fulllength
- 99% clustering (vsearch --cluster_size --id 0.99 --strand both) yields 407,356 centroids
- Every dereplicated and centroid header carries a complete k__/p__/c__/o__/f__/g__/s__-prefixed label matched to an outmap.txt / pid99_outmap.txt entry
- Ambiguity check: fraction of ambiguous species names should rise from ~5.6% in the dereplicated set to ~7.2% after 99% clustering

## Limitations

- All BOLD sequences lacking Family and/or Order information were discarded, and no attempt was made to determine how pervasive Phylum/Class missingness is in the source records
- Clustering at 99% identity increases taxonomic ambiguity (ambiguous species names rise from 5.6% to 7.2%), so tighter centroids carry less precise labels
- The curated database's contents may not be identical to the references the BOLD Classification API uses; the API also omits query coverage in its output, complicating comparisons

## Evidence

- [methods] incorporated the consensus approach to reclassifying taxa script written by William (Tony) Walters: "incorporated the [consensus approach to reclassifying taxa](https://gist.github.com/walterst/9ddb926fece4b7c0e12c) script written by William (Tony) Walters"
- [methods] clustering at 99%, 97%, and 95% identity on the dereplicated reference: "We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id"
- [methods] dereplicate full-length sequences with VSEARCH before LCA label propagation: "we can dereplicate the `tmp_nolabels.fasta` directly:
``nvsearch \
--derep_fulllength tmp_nolabels.fasta"
- [methods] taxonomy incompleteness filter discards records lacking Family or Order ranks: "all sequences that lacked Family and/or Order information were discarded"
- [readme] the tidybug repository documents this COI filtering and database-construction workflow: "Methods and data describing filtering COI datasets and database construction."
