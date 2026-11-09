---
name: mock-community-benchmark-evaluation
description: Use when comparing taxonomic profilers or reference libraries on 16S
  mock community sequencing data, to quantify genus-/species-level abundance error
  or alpha-diversity deviation against known ground-truth composition.
when_to_use_negative:
- Input is real environmental/clinical microbiome data with no known ground-truth
  composition — accuracy-based NRMSE against expected taxa cannot be computed
- The goal is ASV/OTU inference or read processing itself rather than comparative
  benchmarking of profiler/library pairs
- Evaluation targets higher taxonomic levels (phylum/class) — the paper specifically
  analyzes genus- and species-level performance
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: DADA2
  role: 16S amplicon profiler under evaluation, used as the QIIME 2 DADA2 plugin
  repo: github.com/aubreyodom/16SBenchmarking
- name: QIIME 2
  role: naive Bayes classifier profiler under evaluation
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: OTU-clustering profiler under evaluation
  repo: github.com/aubreyodom/16SBenchmarking
- name: PathoScope 2
  role: metagenomic read-mapping profiler under evaluation; best Shannon-deviation
    performer with RefSeq libraries
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: k-mer classifier under evaluation; lowest species-level log NRMSE with Standard
    database
  repo: github.com/aubreyodom/16SBenchmarking
- name: vegan
  role: computes Shannon and Simpson alpha-diversity indices on rarefied count tables
    via diversity()
- name: breakaway
  role: estimates taxonomic richness via breakaway_nof1 with log transform on rarefied
    count tables
- name: MetaScope
  role: provides the download_refseq() function to obtain up-to-date RefSeq indices
  repo: https://compbiomed.github.io/metascope-docs/index.html
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:9e466b07e196fc5ce5b06f94854f0d4b1ed82725e65d7f117905d239a4223bbe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - MetaScope
  - Mothur
  - PathoScope 2
  - QIIME 2
  - breakaway
  - vegan
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mock-community-benchmark-evaluation

## Summary

Benchmarks 16S amplicon taxonomic profiling pipelines (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) paired with reference libraries (SILVA 138, Greengenes 13_8, RefSeq 2018/2020, Kraken 2 Standard) against synthetic mock communities using accuracy-based (log NRMSE) and diversity-deviation (Shannon, Simpson, breakaway_nof1) metrics.

## When to use

Use when comparing taxonomic profilers or reference libraries on 16S mock community sequencing data, to quantify genus-/species-level abundance error or alpha-diversity deviation against known ground-truth composition.

## When NOT to use

- Input is real environmental/clinical microbiome data with no known ground-truth composition — accuracy-based NRMSE against expected taxa cannot be computed
- The goal is ASV/OTU inference or read processing itself rather than comparative benchmarking of profiler/library pairs
- Evaluation targets higher taxonomic levels (phylum/class) — the paper specifically analyzes genus- and species-level performance

## Inputs

- Measured taxonomic abundance count tables from each profiler/reference-library pairing (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x SILVA 138, Greengenes 13_8, RefSeq 2018/2020, Kraken Standard)
- Ground-truth mock community composition libraries (1,000,000 virtual reads) from the 16SBenchmarking repository
- Rarefied (10,000 reads) species-level count tables for alpha-diversity analysis

## Outputs

- Genus- and species-level mean log NRMSE per pipeline/library pair (nrmse_figure2ef.csv)
- Per-pairing and mean absolute alpha-diversity deviations for Shannon, Simpson, and log breakaway_nof1 (figure3_diversity_deviations.csv)

## How to apply

Load measured taxonomic abundance tables from each profiler/library pair (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 each paired with SILVA 138, Greengenes 13_8, RefSeq 2018/2020, or the Kraken Standard library) plus the ground-truth mock community compositions from the 16SBenchmarking repository. For accuracy-based evaluation, restrict comparisons to the union of expected and detected taxa excluding double-zero pairs, compute the Normalized Root Mean Squared Error between measured and true read counts per sample, log-transform it, and aggregate mean log NRMSE per pairing at genus and species levels (e.g., Kraken 2+Standard species mean = 3.77; QIIME 2+Greengenes genus mean = 5.36). For diversity-based evaluation, rarefy count tables to 10,000 reads, compute Shannon and Simpson indices with vegan::diversity() and richness with log-transformed breakaway_nof1, then compare against indices computed on 1,000,000-virtual-read ground-truth libraries via absolute deviation (e.g., PathoScope RefSeq2020 Shannon deviation mean = 0.21). Verify aggregated values against the reported Figure 2E-F and Figure 3 reference numbers before writing per-pairing summary CSVs.

## Related tools

- **DADA2** (16S amplicon profiler under evaluation, used as the QIIME 2 DADA2 plugin) — github.com/aubreyodom/16SBenchmarking
- **QIIME 2** (naive Bayes classifier profiler under evaluation) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (OTU-clustering profiler under evaluation) — github.com/aubreyodom/16SBenchmarking
- **PathoScope 2** (metagenomic read-mapping profiler under evaluation; best Shannon-deviation performer with RefSeq libraries) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (k-mer classifier under evaluation; lowest species-level log NRMSE with Standard database) — github.com/aubreyodom/16SBenchmarking
- **vegan** (computes Shannon and Simpson alpha-diversity indices on rarefied count tables via diversity())
- **breakaway** (estimates taxonomic richness via breakaway_nof1 with log transform on rarefied count tables)
- **MetaScope** (provides the download_refseq() function to obtain up-to-date RefSeq indices) — https://compbiomed.github.io/metascope-docs/index.html

## Examples

```
Rscript nrmse_figure2ef.R --input measured_counts --truth mock_groundtruth.csv --level species --rarefy 10000
```

## Evaluation signals

- Computed mean log NRMSE values reproduce Figure 2E-F benchmarks (Kraken 2+Standard species mean 3.77, QIIME 2+Greengenes genus mean 5.36) within numerical tolerance
- Alpha-diversity deviation means reproduce Figure 3 reference values (Kraken 2+Greengenes Shannon deviation mean = 1.05, DADA2 breakaway deviation mean = 1.37)
- Double-zero pairs (both measured and true counts zero) are excluded before NRMSE computation
- Output files contain one row per pipeline/library pair at each taxonomic level
- Rarefaction to 10,000 reads is applied before Shannon/Simpson/breakaway_nof1 estimation

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step
- Mock species relative abundances may be affected by subtle pipetting variations when aliquoting spike-in DNA; species vary in lysis difficulty, causing underrepresentation in cDNA libraries
- Amplification bias occurs during PCR despite universal primer design, and contamination from reagents or the local environment complicates results
- No analysis pipeline could theoretically exist to perfectly measure a mock community

## Evidence

- [abstract] publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [abstract] PathoScope 2 and Kraken 2 outperformed DADA2, QIIME 2 (DADA2 plugin), and Mothur: "PathoScope 2 and Kraken 2, both tools designed for whole-genome metagenomics, outperformed DADA2, QIIME 2 using the DADA2 plugin, and Mothur"
- [abstract] the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes: "Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes"
- [intro] we specifically analyzed genus- and species-level performance across pairs of profilers and libraries: "we specifically analyzed genus- and species-level performance across pairs of profilers and libraries"
- [discussion] PathoScope 2 and Kraken 2 outperform QIIME 2, Mothur, and DADA2, even when comparing reads to identical reference databases: "our results present strong evidence that PathoScope and Kraken 2 outperform QIIME 2, Mothur, and DADA2, even when comparing reads to identical reference databases"
- [discussion] PathoScope is more sensitive and specific at taxon detection and has a lower error score: "PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur"
- [discussion] Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step: "provide a semblance of a “ground truth” to assess 16S amplicon sequencing analysis methods"
