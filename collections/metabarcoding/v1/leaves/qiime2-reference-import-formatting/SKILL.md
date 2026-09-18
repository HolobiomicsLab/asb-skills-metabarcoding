---
name: qiime2-reference-import-formatting
description: Use when reference for formatting BOLD/GenBank COI FASTA and taxonomy
  tables into dereplicated, LCA-curated, cluster-radius-filtered reference databases
  importable into QIIME 2 classifiers.
when_to_use_negative:
- Input is already a QIIME 2 artifact or feature table — this skill formats raw reference
  downloads, not processed feature data.
- The goal is taxonomic classification itself (e.g., via the BOLD Classification API,
  whose closed-source parameters are undocumented) rather than constructing the reference
  database.
- Working with non-arthropod or non-COI markers, since the seqkit 'f__;' regex filter
  and the 99/97/95% radius choices were tuned for arthropod COI metabarcoding.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0659
tools:
- name: seqkit
  role: grep --invert-match with idsNoFam.txt to drop BOLD records lacking Family-level
    taxonomy before dereplication
  repo: https://github.com/devonorourke/tidybug
- name: QIIME1 pick_otus.py
  role: group identical (--similarity 1.0) reference sequences into pid100 OTUs prior
    to LCA consensus
  repo: https://github.com/devonorourke/tidybug
- name: create_consensus_taxonomy.py
  role: Walters consensus-LCA script resolving conflicting taxonomies among identical
    sequences into outmap.txt
  repo: https://gist.github.com/walterst/bd69a19e75748f79efeb
- name: VSEARCH
  role: derep_fulllength dereplication and cluster_size clustering at 99% identity
    (--strand both) to generate centroid FASTA and radius-filtered reference sets
  repo: https://github.com/torognes/vsearch
- name: QIIME 2 / q2-feature-classifier
  role: downstream consumer of the formatted database (BLAST + LCA and Naive Bayes
    classifiers)
  repo: https://github.com/devonorourke/tidybug
- name: bold R package
  role: source of the raw arthropod BOLD records downloaded 24 February 2019 that
    feed the formatting workflow
  repo: https://github.com/ropensci/bold
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:87d4466accf737657478847cec9033f2824a16123ae69ddbe6a5680bcae4b60b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/devonorourke/tidybug
  tools:
  - QIIME 2
  - QIIME 2 / q2-feature-classifier
  - QIIME1 pick_otus.py
  - VSEARCH
  - bold R package
  - create_consensus_taxonomy.py
  - seqkit
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 12
---

# qiime2-reference-import-formatting

## Summary

Reconstructs the tidybug workflow that converts a raw BOLD arthropod COI download (3,051,818-record FASTA + taxonomy map) into a formatted, dereplicated, LCA-curated reference set ready for import into QIIME 2 (q2-feature-classifier) and VSEARCH-based classifiers, with taxonomy strings carrying k__/p__/c__/o__/f__/g__/s__ prefixes.

## When to use

Reference for formatting BOLD/GenBank COI FASTA and taxonomy tables into dereplicated, LCA-curated, cluster-radius-filtered reference databases importable into QIIME 2 classifiers.

## When NOT to use

- Input is already a QIIME 2 artifact or feature table — this skill formats raw reference downloads, not processed feature data.
- The goal is taxonomic classification itself (e.g., via the BOLD Classification API, whose closed-source parameters are undocumented) rather than constructing the reference database.
- Working with non-arthropod or non-COI markers, since the seqkit 'f__;' regex filter and the 99/97/95% radius choices were tuned for arthropod COI metabarcoding.

## Inputs

- BOLD-derived arthropod COI FASTA (boldCustom.allArth.filt2.fasta)
- Regex/pattern file idsNoFam.txt identifying taxonomically incomplete records ('f__;')
- Two-column taxonomy mapping file (tmp_nolabels.taxa)
- Header-stripped FASTA of COI sequences (tmp_nolabels.fasta)
- Walters create_consensus_taxonomy.py consensus-LCA script

## Outputs

- Family-retaining filtered FASTA (boldCustom.allArth.filt3.fasta, 3,051,818 records)
- pid100 OTU map of identical-sequence groups and consensus-LCA outmap.txt
- Dereplicated FASTA (boldCOI.derep.fasta, 1,841,946 unique records)
- Prefixed reference taxonomy (boldCOI.arth_derep.txt with k__/p__/c__/o__/f__/g__/s__ labels)
- 99%-identity centroid FASTA (boldCOI.clust99.fasta, 407,356 sequences) with pid99_outmap.txt

## How to apply

Start from a BOLD-derived arthropod FASTA (boldCustom.allArth.filt2.fasta) and use seqkit grep with --pattern-file idsNoFam.txt --invert-match to discard sequences lacking Family-level information (regex 'f__;'), yielding boldCustom.allArth.filt3.fasta with 3,051,818 records. Split this into a two-column taxonomy mapping file (tmp_nolabels.taxa) and a header-stripped FASTA, then run QIIME1 pick_otus.py --similarity 1.0 to group identical sequences and apply the Walters create_consensus_taxonomy.py LCA script to resolve conflicting taxonomies among identical copies. Dereplicate with vsearch --derep_fulllength --relabel_keep --notrunclabels (expecting 1,841,946 unique records) and prepend k__/p__/c__/o__/f__/g__/s__ prefixes to generate boldCOI.arth_derep.txt for QIIME 2-compatible import. Optionally cluster at --id 0.99 with vsearch --cluster_size --strand both to produce 407,356 centroids, reapplying the LCA mapping (pid99_outmap.txt) — noting this raises the ambiguous species-name fraction from 5.6% to 7.2%.

## Related tools

- **seqkit** (grep --invert-match with idsNoFam.txt to drop BOLD records lacking Family-level taxonomy before dereplication) — https://github.com/devonorourke/tidybug
- **QIIME1 pick_otus.py** (group identical (--similarity 1.0) reference sequences into pid100 OTUs prior to LCA consensus) — https://github.com/devonorourke/tidybug
- **create_consensus_taxonomy.py** (Walters consensus-LCA script resolving conflicting taxonomies among identical sequences into outmap.txt) — https://gist.github.com/walterst/bd69a19e75748f79efeb
- **VSEARCH** (derep_fulllength dereplication and cluster_size clustering at 99% identity (--strand both) to generate centroid FASTA and radius-filtered reference sets) — https://github.com/torognes/vsearch
- **QIIME 2 / q2-feature-classifier** (downstream consumer of the formatted database (BLAST + LCA and Naive Bayes classifiers)) — https://github.com/devonorourke/tidybug
- **bold R package** (source of the raw arthropod BOLD records downloaded 24 February 2019 that feed the formatting workflow) — https://github.com/ropensci/bold

## Examples

```
vsearch --cluster_size boldCOI.derep.fasta --id 0.99 --strand both --threads 24 --centroids boldCOI.clust99.fasta --uc pid99_otus.txt
```

## Evaluation signals

- Filtered FASTA retains exactly 3,051,818 records after seqkit Family-level filtering.
- Dereplicated FASTA contains exactly 1,841,946 unique sequences after vsearch --derep_fulllength.
- 99% clustering yields exactly 407,356 centroid sequences in boldCOI.clust99.fasta.
- All taxonomy strings carry full k__/p__/c__/o__/f__/g__/s__ prefixes and match entries in outmap.txt / pid99_outmap.txt.
- Ambiguous species-name fraction can be compared across formats: 5.6% in the dereplicated set vs 7.2% after 99% clustering.

## Limitations

- BOLD sequences are not exclusively ATCG; non-ambiguous character distribution must be checked before import (Filter2 in the database-construction docs).
- Clustering at 99% identity increases taxonomic ambiguity (ambiguous species names rise from 5.6% to 7.2%), so centroid-level reference sets trade size for label ambiguity.
- It is unclear whether the locally built tidybug database contents match those behind the BOLD Classification API, and BOLD has not issued packaged versioned releases since 2015, complicating reproducibility.
- Records lacking Family and/or Order information are discarded outright, and no attempt was made to quantify prevalence of records missing Phylum/Class information.

## Evidence

- [methods] Cluster radii produce distinct reference sets: "We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id"
- [methods] Consensus LCA script source: "incorporated the [consensus approach to reclassifying taxa](https://gist.github.com/walterst/9ddb926fece4b7c0e12c) script written by William (Tony) Walters"
- [methods] Dereplication step precedes radius filtering: "we can dereplicate the `tmp_nolabels.fasta` directly:
``nvsearch \
--derep_fulllength tmp_nolabels.fasta"
- [methods] BOLD records contain ambiguous characters requiring format checks: "The BOLD sequences do not exlcusively contain non-ambiguous DNA characters. To see the distribution of which characters are present:"
- [methods] Family/Order completeness filter defines the input set: "all sequences that lacked Family and/or Order information were discarded"
