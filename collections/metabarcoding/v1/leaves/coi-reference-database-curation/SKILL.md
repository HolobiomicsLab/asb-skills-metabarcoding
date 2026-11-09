---
name: coi-reference-database-curation
description: Use when building or comparing COI (or other barcode-gene) reference
  databases for metabarcoding classification, when you need to quantify the effect
  of clustering radius or filtering on reference composition, or when merging BOLD-
  and GenBank-derived resources.
when_to_use_negative:
- The input is already a feature/ASV table and the goal is diversity metric computation
  or count transformation — this skill only covers reference database construction
  and comparison.
- The goal is to benchmark taxonomic classifiers (SINTAX, BLAST+LCA, Naive Bayes,
  BOLD API) on a fixed reference set rather than to build or curate the reference
  database itself.
- The marker gene is not COI or the target taxa are non-arthropod — the Family/Order
  completeness filters and BOLD arthropod retrieval steps here are COI/arthropod-specific.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0114
tools:
- name: bold (R package)
  role: Downloads arthropod records from the BOLD database as raw input for reference
    curation
  repo: https://github.com/ropensci/bold
- name: seqkit
  role: Filters FASTA records by ID pattern / taxonomy string, e.g., discarding sequences
    lacking Family-level information with --invert-match
- name: QIIME1 pick_otus.py
  role: Groups identical sequences (--similarity 1.0) prior to consensus taxonomy
    assignment and dereplication
- name: create_consensus_taxonomy.py
  role: Walters consensus/LCA script that resolves conflicting taxonomies among identical
    dereplicated sequences
  repo: https://gist.github.com/walterst/bd69a19e75748f79efeb
- name: VSEARCH
  role: Dereplicates (--derep_fulllength) and clusters the reference set (--cluster_size
    --id 0.99 --strand both); also converts Palmer COI.udb to FASTA with --udb2fasta
  repo: https://github.com/torognes/vsearch
- name: CO1Classifier (Porter)
  role: GenBank-derived Eukaryote CO1 training FASTA used as the comparison database
    in overlap analyses
  repo: https://github.com/terrimporter/CO1Classifier
- name: R (figure4_database_comparisons.R)
  role: Tabulates shared/distinct taxonomic names per rank (Class-Species) across
    databases and renders the upset plot
  repo: https://github.com/devonorourke/tidybug
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:d24cba9d95e32f776bc6523a6783650b77a6fa5cbb68bf778f4971fa0e4f7919
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ropensci/bold
  tools:
  - CO1Classifier (Porter)
  - QIIME 2
  - QIIME1 pick_otus.py
  - R
  - R (figure4_database_comparisons.R)
  - VSEARCH
  - bold (R package)
  - create_consensus_taxonomy.py
  - csvtk
  - seqkit
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# coi-reference-database-curation

## Summary

Curate arthropod COI reference databases by downloading BOLD (and GenBank-derived) records, filtering taxonomically incomplete sequences, dereplicating with LCA consensus taxonomy resolution, and clustering at defined identity radii (e.g., 99%) to quantify how clustering radius shapes database composition and taxonomic ambiguity.

## When to use

Use when building or comparing COI (or other barcode-gene) reference databases for metabarcoding classification, when you need to quantify the effect of clustering radius or filtering on reference composition, or when merging BOLD- and GenBank-derived resources.

## When NOT to use

- The input is already a feature/ASV table and the goal is diversity metric computation or count transformation — this skill only covers reference database construction and comparison.
- The goal is to benchmark taxonomic classifiers (SINTAX, BLAST+LCA, Naive Bayes, BOLD API) on a fixed reference set rather than to build or curate the reference database itself.
- The marker gene is not COI or the target taxa are non-arthropod — the Family/Order completeness filters and BOLD arthropod retrieval steps here are COI/arthropod-specific.

## Inputs

- BOLD arthropod COI records (retrieved with the bold R package)
- GenBank-derived COI training FASTA (e.g., Porter Eukaryote CO1 v3.2 from CO1Classifier releases)
- amptk COI database (e.g., Palmer COI.udb, convertible to FASTA with vsearch --udb2fasta)
- FASTA files with embedded taxonomy strings (e.g., boldCustom.allArth.filt2.fasta)
- ID pattern files for taxonomic filtering (e.g., idsNoFam.txt)

## Outputs

- Taxonomically filtered FASTA (e.g., boldCOI.arth filt3.fasta; 3,051,814 records)
- Dereplicated FASTA with LCA-corrected taxonomy (e.g., boldCOI.derep.fasta; 1,841,946 records) and matching taxonomy table (boldCOI.arth_derep.txt)
- Clustered centroid FASTA at a chosen identity radius (e.g., boldCOI.clust99.fasta; 407,356 sequences) with centroid LCA mapping (pid99_outmap.txt)
- Per-rank taxa tables (e.g., palmer.taxa.txt.gz, porter.taxa.txt.gz, derep.taxa.txt.gz) and overlap/upset-plot summary of shared vs unique names

## How to apply

Download arthropod records from BOLD (via the bold R package, e.g., on 24 February 2019) and/or GenBank-derived training sets (Porter CO1Classifier, Palmer amptk COI). Filter for taxonomic completeness by discarding all sequences that lacked Family and/or Order information using seqkit grep --pattern-file with --invert-match (e.g., boldCustom.allArth.filt2.fasta to filt3.fasta, 3,051,814 records). Group identical sequences with QIIME1 pick_otus.py --similarity 1.0, resolve conflicting taxonomies among identical sequences with the Walters create_consensus_taxonomy.py LCA script, and dereplicate with vsearch --derep_fulllength (expected 1,841,946 unique records). Finally cluster the dereplicated FASTA with vsearch --cluster_size --id 0.99 --strand both to produce centroids (expected 407,356), then re-apply LCA mapping to the centroids; compare shared/unique taxonomic names across databases per rank (Class-Species) using R to demonstrate complementarity of GenBank- vs BOLD-derived sets.

## Related tools

- **bold (R package)** (Downloads arthropod records from the BOLD database as raw input for reference curation) — https://github.com/ropensci/bold
- **seqkit** (Filters FASTA records by ID pattern / taxonomy string, e.g., discarding sequences lacking Family-level information with --invert-match)
- **QIIME1 pick_otus.py** (Groups identical sequences (--similarity 1.0) prior to consensus taxonomy assignment and dereplication)
- **create_consensus_taxonomy.py** (Walters consensus/LCA script that resolves conflicting taxonomies among identical dereplicated sequences) — https://gist.github.com/walterst/bd69a19e75748f79efeb
- **VSEARCH** (Dereplicates (--derep_fulllength) and clusters the reference set (--cluster_size --id 0.99 --strand both); also converts Palmer COI.udb to FASTA with --udb2fasta) — https://github.com/torognes/vsearch
- **CO1Classifier (Porter)** (GenBank-derived Eukaryote CO1 training FASTA used as the comparison database in overlap analyses) — https://github.com/terrimporter/CO1Classifier
- **R (figure4_database_comparisons.R)** (Tabulates shared/distinct taxonomic names per rank (Class-Species) across databases and renders the upset plot) — https://github.com/devonorourke/tidybug

## Examples

```
vsearch --cluster_size boldCOI.derep.fasta --id 0.99 --centroids boldCOI.clust99.fasta --sizein --sizeout --relabel_keep --threads 24 --strand both
```

## Evaluation signals

- Filtered FASTA retains exactly 3,051,818→3,051,814-range records matching the reported 3,051,814 after Family/Order completeness filtering (regex 'f__;')
- Dereplication with vsearch --derep_fulllength yields the expected 1,841,946 unique records
- 99% identity clustering (vsearch --cluster_size --id 0.99) yields the expected 407,356 centroid sequences
- Ambiguous-species-name fraction increases from 5.6% in the dereplicated set to 7.2% after 99% clustering, quantifying the ambiguity cost of tighter clustering radii
- Upset plot of Class-through-Species names shows both shared and unique names per database (e.g., of 12,788 shared genera, 4,251 unique to Porter and 3,885 unique to tidybug), confirming complementarity

## Limitations

- Tighter clustering radii reduce the reference set but increase taxonomic ambiguity (ambiguous species names rose from 5.6% to 7.2% at 99% identity), so there is a trade-off between size and label confidence
- There is no single curated and versioned resource widely used in animal diet studies, making comparisons among studies extremely difficult; BOLD also discontinued packaged versioned releases in 2015
- Reference composition depends on download date and source (BOLD accessed 24 February 2019), and it was unclear whether the tidybug database contents matched those used by the closed-source BOLD Classification API

## Evidence

- [intro] we examined how the effects of reference database choice and subsequent filtering criteria (clustering radius) on reference composition: "we examined how the effects of reference database choice and subsequent filtering criteria (clustering radius) on reference composition"
- [methods] Family/Order completeness filter: "all sequences that lacked Family and/or Order information were discarded"
- [methods] clustering radii tested: "We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id"
- [methods] BOLD record download: "We accessed arthropod records from the BOLD database using the bold R package (Chamberlain, 2017) on 24 February 2019"
- [abstract] complementarity of GenBank and BOLD: "available reference datasets from GenBank and BOLD for the animal marker gene cytochrome oxidase I (COI) can be complementary"
- [readme] repo purpose: "Methods and data describing filtering COI datasets and database construction."
- [discussion] lack of versioned reference resources: "yet there is no single curated and versioned resource widely used in animal diet studies, making comparisons among studies extremely difficult"
