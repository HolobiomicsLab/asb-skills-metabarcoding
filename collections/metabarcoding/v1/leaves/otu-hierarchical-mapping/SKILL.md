---
name: otu-hierarchical-mapping
description: Use when demultiplexed, quality-filtered rRNA amplicon or shotgun reads
  (FASTA) must be classified against a reference taxonomy and hierarchical OTU clusters
  simultaneously, with confidence values per level, to build OTU count tables or compare
  OTU consistency across sequencing runs.
when_to_use_negative:
- Input reads are not demultiplexed and quality filtered — MAPseq expects cleaned
  FASTA input and is not a demultiplexing or QC tool
- You need de novo OTU clustering with no suitable full-length rRNA reference set
  — MAPseq is a reference-based classifier mapping reads to precomputed OTU clusters
  and known taxonomies
- You need organism-level (species) OTU identifications where reference coverage is
  sparse — low reference coverage can drive confidence (and hence assigned labels)
  down, e.g. the authors dropped the LTP taxonomy due to low coverage
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MAPseq
  role: Performs the k-mer search-based reference classification, assigning each rRNA
    read to NCBI taxonomy and hierarchical OTU mappings with confidence estimates
    in one run
  repo: https://github.com/jfmrod/mapseq
- name: MAPseq 1.2.6
  role: Specific version used for the Human Microbiome Project two-run OTU consistency
    analysis; its -otutable option (fixed in 1.2.6 to use counts from all samples)
    generates the multi-sample OTU count tables
  repo: https://github.com/jfmrod/mapseq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:a9bda7a3896daa9af783a939accec4b0988806e3e68dd176b5c4d14a7574e0ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - MAPseq 1.2.6
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# otu-hierarchical-mapping

## Summary

Assign rRNA sequence reads to a reference taxonomy and to hierarchical operational taxonomic unit (OTU) clusters in a single MAPseq run, with per-level confidence estimates, to obtain taxonomy classifications and OTU mappings suitable for cross-run reproducibility analyses of amplicon datasets.

## When to use

Use when demultiplexed, quality-filtered rRNA amplicon or shotgun reads (FASTA) must be classified against a reference taxonomy and hierarchical OTU clusters simultaneously, with confidence values per level, to build OTU count tables or compare OTU consistency across sequencing runs.

## When NOT to use

- Input reads are not demultiplexed and quality filtered — MAPseq expects cleaned FASTA input and is not a demultiplexing or QC tool
- You need de novo OTU clustering with no suitable full-length rRNA reference set — MAPseq is a reference-based classifier mapping reads to precomputed OTU clusters and known taxonomies
- You need organism-level (species) OTU identifications where reference coverage is sparse — low reference coverage can drive confidence (and hence assigned labels) down, e.g. the authors dropped the LTP taxonomy due to low coverage

## Inputs

- Demultiplexed, quality-filtered rRNA read sequences in FASTA format (e.g. rawseqs.fa, from amplicon or shotgun sequencing)
- Optional custom nucleotide FASTA reference plus one or more tab-separated taxonomy files with a '#cutoff:', '#name:', '#levels:' header (accession:coordinates and semicolon-separated lineage per row)
- MAPseq .mseq output files from multiple samples (for -otucounts / -otutable summarization)

## Outputs

- .mseq classification file per sample: per-read best reference hit, alignment statistics (score, identity, matches, mismatches, gaps, start/end positions), and hierarchical taxonomy/OTU assignments with confidence values ('simple' or 'confidences' output format)
- Per-sample OTU/taxonomy count summary from 'mapseq -otucounts' (Taxonomy, TaxonomyLevel, Label, Counts table)
- Multi-sample OTU count table (TSV, samples as columns, OTU/taxonomic labels as rows) from 'mapseq -otutable'
- Reproducibility metrics when applied across paired runs: per-sample Jaccard indices and Spearman abundance correlations, assembled per tool (e.g. hmp_reproducibility_metrics.csv)

## How to apply

Run MAPseq (the task card used v1.2.6) on each demultiplexed, quality-filtered FASTA of rRNA reads, e.g. 'mapseq rawseqs.fa > rawseqs.fa.mseq', which classifies reads against the bundled reference (NCBI taxonomy plus precomputed high-quality OTU clusters) and writes an .mseq file where each line gives the best hit, alignment score, and the taxonomy/OTU assignment with a combined confidence at each hierarchical level. The combined confidence merges a score confidence (controlling misclassification) and an identity-cutoff confidence (controlling divergence from the reference); use a combined confidence cutoff of 0.4–0.5, since 0.5 gave the highest F1/2-score in the authors' benchmarks. To produce count summaries, run 'mapseq -otucounts <sample.mseq>' per sample, or build a multi-sample OTU/taxa table with 'mapseq -otutable <sample1.mseq> <sample2.mseq> ...' selecting taxonomy and level via the -ti and -tl flags; the resulting TSV imports directly into R via read.table(sep='\t', header=TRUE). For cross-run consistency, compute per-sample Jaccard indices of shared OTU identifications between runs and Spearman correlations of OTU abundances across samples, as in the Human Microbiome Project two-run analysis where MAPseq reached a median Jaccard of 0.52, outperforming VSEARCH, NINJA-OPS, and USEARCH.

## Related tools

- **MAPseq** (Performs the k-mer search-based reference classification, assigning each rRNA read to NCBI taxonomy and hierarchical OTU mappings with confidence estimates in one run) — https://github.com/jfmrod/mapseq
- **MAPseq 1.2.6** (Specific version used for the Human Microbiome Project two-run OTU consistency analysis; its -otutable option (fixed in 1.2.6 to use counts from all samples) generates the multi-sample OTU count tables) — https://github.com/jfmrod/mapseq

## Examples

```
mapseq -otutable sample1.mseq sample2.mseq sample3.mseq > map.otutable
```

## Evaluation signals

- Every read line in the .mseq output carries a combined confidence value per taxonomic/OTU level; assignments below the chosen 0.4–0.5 cutoff should be absent in the 'simple' output format
- The -otutable TSV must contain counts from ALL input .mseq samples (columns per sample) and be importable in R via read.table(sep='\t', header=TRUE) — note the pre-1.2.6 bug that used only the first sample's counts
- Per-sample cross-run Jaccard indices of shared OTU identifications must lie in [0, 1]
- On the 2194-sample Human Microbiome Project two-run benchmark, the OTU mappings should reproduce MAPseq's highest median cross-run Jaccard of 0.52 (Fig. 1g-h), ahead of VSEARCH, NINJA-OPS, and USEARCH
- Spearman correlation of OTU abundances between run 1 and run 2 across the 2194 samples should reflect MAPseq's best speed-versus-accuracy trade-off among the compared tools

## Limitations

- Combined confidence cutoff trades accuracy against coverage: 0.4–0.5 is recommended for highest F1/2-score, but low-confidence (divergent or poorly covered) reads receive no or truncated assignments
- Only move the whole MAPseq installation directory, not just the binary — data files are searched relative to the binary's path and a moved binary alone breaks the installation
- Compiling from source requires svn, autotools/autoconf, wget, git, libncurses5-dev, and libtool, plus ./setup.sh (downloads eutils and reference data) and ./bootstrap before ./configure; make
- Reference quality matters: mapref updates removed low-quality reference sequences that caused issues when classifying low-quality query sequences, and the LTP taxonomy was dropped due to low coverage
- Earlier versions had bugs affecting hierarchical OTU outputs (e.g. -otutable using only the first sample's counts before 1.2.6; double hits for long queries >1200 bp before 1.2.3), so version pinning is important for reproducibility

## Evidence

- [abstract] MAPseq provides in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences in both amplicon and shotgun sequencing strategies: "providing in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences in both amplicon and shotgun sequencing strategies"
- [readme] Recommended confidence cutoff of 0.4-0.5 from benchmarks: [evidence span withheld — anchor status: external_doc]
- [readme] OTU count table generation for multiple samples: "mapseq -otutable <sample1.mseq> <sample2.mseq> ...  

Generates a tab separated value (tsv) file with the counts for each sample (column wise) and OTU or taxonomic labels (row wise)."
- [readme] Combined confidence composition: [evidence span withheld — anchor status: external_doc]
