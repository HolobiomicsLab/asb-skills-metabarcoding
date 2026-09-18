---
name: fasta-header-taxonomy-parsing
description: Use when building a curated COI reference database from BOLD FASTA records
  whose headers carry taxonomy strings, and you need header-stripped sequences plus
  a parallel taxonomy table for dereplication, LCA curation, and clustering.
when_to_use_negative:
- Input headers carry no taxonomy (e.g. raw amplicon reads or denoised ASVs awaiting
  classification) — there is no lineage to parse out
- The taxonomy assignments will come from a probabilistic classifier (RDP/Naive Bayes,
  SINTAX, BOLD Classification API) instead of constructing a curated LCA-curbed reference
  database
- Input is already a QIIME 2 artifact or feature table with taxonomy strings embedded
  — parsing FASTA headers would be redundant
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: seqkit
  role: grep with --pattern-file idsNoFam.txt --invert-match to discard sequences
    lacking Family-level information before header parsing
  repo: https://github.com/devonorourke/tidybug
- name: QIIME1 pick_otus.py
  role: groups identical sequences (--similarity 1.0, --threads 24) into OTUs whose
    members need consensus taxonomy via the parsed mapping file
  repo: https://github.com/devonorourke/tidybug
- name: create_consensus_taxonomy.py
  role: Walters LCA consensus script that resolves conflicting taxonomies among identical
    sequences using the two-column taxonomy map
  repo: https://gist.github.com/walterst/9ddb926fece4b7c0e12c
- name: VSEARCH
  role: dereplicates (--derep_fulllength --relabel_keep --notrunclabels) and clusters
    (--cluster_size --id 0.99) the header-stripped FASTA, then centroids are matched
    to LCA headers
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:1bd0922ff411ba552bfe32bb272ce9009d544c59c76734319c33281d8e705861
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# fasta-header-taxonomy-parsing

## Summary

Recover lineage information embedded in FASTA headers by splitting a filtered FASTA into a two-column taxonomy mapping file (tmp_nolabels.taxa) and a header-stripped FASTA, resolving conflicting taxonomies among identical sequences with Walters' LCA consensus, and re-attaching rank-prefixed lineages (k__…s__) to dereplicated and 99%-clustered centroids.

## When to use

Use when building a curated COI reference database from BOLD FASTA records whose headers carry taxonomy strings, and you need header-stripped sequences plus a parallel taxonomy table for dereplication, LCA curation, and clustering.

## When NOT to use

- Input headers carry no taxonomy (e.g. raw amplicon reads or denoised ASVs awaiting classification) — there is no lineage to parse out
- The taxonomy assignments will come from a probabilistic classifier (RDP/Naive Bayes, SINTAX, BOLD Classification API) instead of constructing a curated LCA-curbed reference database
- Input is already a QIIME 2 artifact or feature table with taxonomy strings embedded — parsing FASTA headers would be redundant

## Inputs

- Family-filtered FASTA (boldCustom.allArth.filt3.fasta, 3,051,814 records) with taxonomy embedded in headers
- OTU/centroid map files (tmp_nolabels_otus.txt, pid99_otus) from QIIME1 pick_otus.py or vsearch --cluster_size
- LCA consensus output outmap.txt from create_consensus_taxonomy.py

## Outputs

- Two-column taxonomy mapping file (tmp_nolabels.taxa)
- Header-stripped FASTA (tmp_nolabels.fasta)
- Rank-prefixed reference table boldCOI.arth_derep.txt (1,841,946 records) and pid99_outmap.txt for 407,356 centroids

## How to apply

Start from the Family-filtered FASTA (boldCustom.allArth.filt3.fasta, 3,051,814 records) and split each record into a two-column taxonomy mapping file (tmp_nolabels.taxa) and a header-stripped FASTA (tmp_nolabels.fasta) so sequences can be grouped with QIIME1 pick_otus.py --similarity 1.0. Run the Walters create_consensus_taxonomy.py LCA script on tmp_nolabels.taxa, tmp_nolabels.fasta, and the OTU map to resolve conflicting taxonomies among identical sequences, yielding outmap.txt. After vsearch --derep_fulllength --relabel_keep --notrunclabels dereplication (1,841,946 unique records), match outmap.txt entries to the dereplicated headers and add k__/p__/c__/o__/f__/g__/s__ prefixes to generate boldCOI.arth_derep.txt; repeat the mapping on 99% vsearch --cluster_size centroids (pid99_outmap.txt). The rank-prefix format is required so downstream QIIME 2 classifiers accept the reference.

## Related tools

- **seqkit** (grep with --pattern-file idsNoFam.txt --invert-match to discard sequences lacking Family-level information before header parsing) — https://github.com/devonorourke/tidybug
- **QIIME1 pick_otus.py** (groups identical sequences (--similarity 1.0, --threads 24) into OTUs whose members need consensus taxonomy via the parsed mapping file) — https://github.com/devonorourke/tidybug
- **create_consensus_taxonomy.py** (Walters LCA consensus script that resolves conflicting taxonomies among identical sequences using the two-column taxonomy map) — https://gist.github.com/walterst/9ddb926fece4b7c0e12c
- **VSEARCH** (dereplicates (--derep_fulllength --relabel_keep --notrunclabels) and clusters (--cluster_size --id 0.99) the header-stripped FASTA, then centroids are matched to LCA headers) — https://github.com/torognes/vsearch

## Examples

```
vsearch --derep_fulllength tmp_nolabels.fasta --relabel_keep --notrunclabels --output boldCOI.derep.fasta
```

## Evaluation signals

- Header-stripped FASTA and taxonomy map have equal record counts (3,051,814 records after Family filtering)
- Dereplicated FASTA contains exactly 1,841,946 unique sequences and 99% clustering yields 407,356 centroids, as reported in the database curation document
- Every output lineage carries the full rank-prefix scaffold k__/p__/c__/o__/f__/g__/s__ from the outmap.txt matches
- Records lacking Family-level information (regex 'f__;') were removed before parsing (seqkit grep --invert-match)
- Conflicting taxonomies among identical sequences are resolved to a single consensus line per OTU in outmap.txt

## Limitations

- Clustering at 99% identity increases taxonomic ambiguity: ambiguous species names rise from 5.6% (dereplicated) to 7.2% of records
- Sequences lacking Family and/or Order information were discarded outright, and no attempt was made to determine how pervasive missing Phylum/Class information is in BOLD
- BOLD no longer provides packaged versioned releases (since 2015), so database content is time-stamped to the 24 February 2019 download via the bold R package

## Evidence

- [methods] Split filt3 into a two-column taxonomy mapping file (tmp_nolabels.taxa) and a header-stripped FASTA (tmp_nolabels.fasta).: [evidence span withheld — anchor status: not_found]
- [methods] match outmap.txt entries to the dereplicated headers and add k__/p__/c__/o__/f__/g__/s__ prefixes to generate boldCOI.arth_derep.txt: [evidence span withheld — anchor status: not_found]
- [methods] we can dereplicate the `tmp_nolabels.fasta` directly: "we can dereplicate the `tmp_nolabels.fasta` directly"
- [intro] Thus, we examined how the effects of reference database choice and subsequent filtering criteria (clustering radius) on reference composition: "Thus, we examined how the effects of reference database choice and subsequent filtering criteria (clustering radius) on reference composition"
- [readme] Large database files are available at: https://osf.io/k3eh6/: "Large database files are available at: https://osf.io/k3eh6/"
