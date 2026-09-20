---
name: reference-database-recapture-analysis
description: Use when you have a set of exact amplicon sequences (e.g., Deblur 90-bp
  V4 tags) and want to compute what fraction of a reference database's sequences they
  exactly match, or conversely what fraction of your sequences hit the references.
when_to_use_negative:
- Your query sequences are partial-length or expected to differ from references —
  a 100% similarity global alignment will grossly undercount recapture; use a similarity-threshold
  or closed-reference OTU picking workflow instead
- Your sequences have not been error-corrected to exact sequence variants (e.g., raw
  denoised reads or clustered OTUs), since the recaptured fraction is defined against
  exact Deblur-derived tag sequences
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0632
tools:
- name: VSEARCH
  role: Runs usearch_global in global alignment mode at --id 1.0 to find exact matches
    between EMP tag sequences and reference database sequences
- name: Deblur
  role: Produces the exact-sequence 90-bp V4 tag sequences (starting after primer
    515f) that serve as the query set
  repo: https://github.com/cuttlefishh/deblur
- name: Python
  role: Extracts unique tag sequences from the observation table and computes/writes
    the four recapture fractions to CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:0d0f2d682408360beecd2f28a122397ae0ed9c711384fdf52ab3e757d023d575
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Python
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reference-database-recapture-analysis

## Summary

Quantifies how much of the unique sequence diversity in existing 16S rRNA reference databases (Greengenes 13.8, SILVA 128) is recaptured by a set of exact environmental sequences — here the EMP Deblur 90-bp V4 tag sequences — via VSEARCH global alignment at 100% similarity. It measures overlap between an exact-sequence catalogue and full-length references without OTU clustering.

## When to use

Use when you have a set of exact amplicon sequences (e.g., Deblur 90-bp V4 tags) and want to compute what fraction of a reference database's sequences they exactly match, or conversely what fraction of your sequences hit the references.

## When NOT to use

- Your query sequences are partial-length or expected to differ from references — a 100% similarity global alignment will grossly undercount recapture; use a similarity-threshold or closed-reference OTU picking workflow instead
- Your sequences have not been error-corrected to exact sequence variants (e.g., raw denoised reads or clustered OTUs), since the recaptured fraction is defined against exact Deblur-derived tag sequences

## Inputs

- Deblur 90-bp observation table (BIOM) of exact V4 tag sequences
- Greengenes 13.8 full-length 16S rRNA reference sequences (FASTA)
- SILVA 128 reference sequences subset to bacterial and archaeal (non-eukaryotic) taxa (FASTA)

## Outputs

- reference_recapture_fractions.csv with four fractions: fraction of Greengenes and SILVA references hit by >=1 tag, and fraction of tag sequences matching >=1 reference in each database
- Counts of matched reference sequences and matched tag sequences per database

## How to apply

Load the Deblur 90-bp observation table (V4 sequences starting after primer 515f) and extract the set of unique tag sequences. Run VSEARCH usearch_global in global alignment mode with --id 1.0, using the tag sequences as queries against the full-length Greengenes 13.8 reference sequences; repeat against the SILVA 128 references subset to non-eukaryotic (bacterial and archaeal) taxa. Count reference sequences hit by at least one tag and tag sequences matching at least one reference, in each database, then compute the four fractions (reference-recaptured and tag-matched, per database) and write them to a CSV. Exact-sequence matching at 100% identity is the point: the paper's analytical approach favors exact sequences over clustered OTUs, so no similarity threshold relaxation or clustering is applied.

## Related tools

- **VSEARCH** (Runs usearch_global in global alignment mode at --id 1.0 to find exact matches between EMP tag sequences and reference database sequences)
- **Deblur** (Produces the exact-sequence 90-bp V4 tag sequences (starting after primer 515f) that serve as the query set) — https://github.com/cuttlefishh/deblur
- **Python** (Extracts unique tag sequences from the observation table and computes/writes the four recapture fractions to CSV)

## Examples

```
vsearch --usearch_global emp_deblur_90bp_tags.fna --db greengenes_13_8.fna --id 1.0 --strand both --matched greengenes_exact_hits.fna --blastout6 greengenes_hits.tsv
```

## Evaluation signals

- The computed fractions reproduce the published values: 46% of Greengenes and 45% of SILVA references recaptured; 10% and 13% of tag sequences matching, respectively
- reference_recapture_fractions.csv contains exactly four fractions (2 databases x 2 directions), all within [0, 1]
- Every reported match comes from a global alignment at exactly --id 1.0; verify no hits were produced at lower identity
- The SILVA comparison uses only non-eukaryotic (bacterial and archaeal) reference sequences
- Tag query count equals the number of unique 90-bp sequences in the Deblur observation table

## Limitations

- Only exact-sequence matches are counted; sequences with even one mismatch are not recaptured, so fractions represent a lower bound on similarity-based overlap
- Results depend on the specific reference versions (Greengenes 13.8, SILVA 128) and the 90-bp V4 tag length; different versions or amplicon regions will change the fractions
- SILVA 128 must be subset to non-eukaryotic taxa before alignment, or eukaryotic references will deflate the recaptured fraction

## Evidence

- [supplementary] Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm.: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [methods] Exact-sequence OTU picking method and reference OTU picking: [evidence span withheld — anchor status: not_found]
- [abstract] Use of exact sequences instead of clustered OTUs as the analytical basis: "Coordinated protocols and new analytical methods, particularly the use of exact sequences instead of clustered operational taxonomic units"
- [methods] EMP 16S Release 1 contents available for meta-analysis: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
