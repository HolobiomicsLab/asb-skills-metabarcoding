---
name: differential-abundance-testing-microbiome
description: Use when comparing taxa between exactly two sample groups in 16S rRNA
  ASV/genera count tables, benchmarking which DA tools call significant features,
  or deciding whether to rarefy and prevalence-filter before DA testing.
when_to_use_negative:
- When the goal is diversity or ordination analysis rather than between-group DA testing
  — the authors state that 'whether rarefying is advisable for other analyses, such
  as prior to computing diversity metrics, is beyond the scope of this work'.
- When the metadata has more than two sample groupings — the pipeline metadata file
  supports a 'maximum 2 groupings'.
- When a single tool's output is sufficient for the biological claim — the paper found
  tools identify drastically different significant ASVs, so single-tool results are
  fragile.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ALDEx2
  role: CLR-based DA test; most consistent across studies and highest precision (0.82
    at AUROC 0.7) alongside ANCOM-II
- name: ANCOM-II
  role: Most stringent tool on 10% prevalence-filtered data (mean 3.8% significant
    ASVs) and top precision (0.99)
- name: DESeq2
  role: Negative-binomial count-based DA tool run via run_all_tools.sh
- name: edgeR
  role: Negative-binomial DA tool; loosest caller (mean 32.5% significant ASVs filtered)
    and highest false-positive outlier (0.69–27.9%)
- name: LEfSe
  role: Percentage-then-rank-based DA tool run in the 'hackathon' conda environment
- name: corncob
  role: Beta-binomial DA model run on each dataset via the pipeline
- name: metagenomeSeq
  role: Zero-inflated Gaussian DA model in the 14-tool suite
- name: Maaslin2
  role: Multivariable DA framework run per dataset
- name: limma voom
  role: voom-transformed linear-model DA variant (TMMwsp unfiltered mean 40.5% significant
    ASVs)
- name: ANCOM
  role: Additive log-ratio DA tool (archived standalone functions); excluded from
    extended false-positive replicates
  repo: https://www.bioconductor.org/packages/release/bioc/html/ANCOMBC.html
merged_aliases:
- microbiome-16s-differential-abundance-testing
- microbiome-differential-abundance-benchmarking
- microbiome-differential-abundance-testing
merged_alias_records:
- alias: microbiome-16s-differential-abundance-testing
  slug: microbiome-16s-differential-abundance-testing
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
- alias: microbiome-differential-abundance-benchmarking
  slug: microbiome-differential-abundance-benchmarking
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
- alias: microbiome-differential-abundance-testing
  slug: microbiome-differential-abundance-testing
  jaccard_score: 1.0
  method: token-set-jaccard
  decision: auto
provenance:
  source_task_ids:
  - task_003
  - task_008
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:7a7421578f93647c0e05fd625f7a190df07b080db62a52f733ea92fc924314f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALDEx2
  - ANCOM
  - ANCOM-II
  - DESeq2
  - LEfSe
  - Maaslin2
  - ROCR
  - corncob
  - dplyr
  - edgeR
  - ggplot2
  - limma voom
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# differential-abundance-testing-microbiome

## Summary

Runs and interprets 14 differential abundance (DA) methods (ALDEx2 1.18.0, ANCOM-II, corncob 0.1.0, DESeq2 1.26.0, edgeR 3.28.1, LEfSe, Maaslin2 0.99.12, metagenomeSeq 1.28.2, limma voom, ANCOM, and rarefied/CLR test variants) on 16S rRNA ASV count tables with two sample groups, quantifying how many and which ASVs/genera each tool calls significant under common pre-processing choices.

## When to use

Use when comparing taxa between exactly two sample groups in 16S rRNA ASV/genera count tables, benchmarking which DA tools call significant features, or deciding whether to rarefy and prevalence-filter before DA testing.

## When NOT to use

- When the goal is diversity or ordination analysis rather than between-group DA testing — the authors state that 'whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work'.
- When the metadata has more than two sample groupings — the pipeline metadata file supports a 'maximum 2 groupings'.
- When a single tool's output is sufficient for the biological claim — the paper found tools identify drastically different significant ASVs, so single-tool results are fragile.

## Inputs

- 16S rRNA ASV count table (absolute abundance, TSV) per dataset
- Two-column metadata TSV mapping sample names to two sample groupings
- Rarefied ASV table when testing rarefied variants (rarefaction depth 2000)
- Genus-level collapsed count tables for cross-study consistency analysis
- Raw 16S rRNA Microbiome Datasets from figshare deposit 14531724

## Outputs

- Per-tool significant-ASV/genera tables for each dataset (filtered and unfiltered variants)
- Per-tool-per-dataset percentage of significant ASVs (significant_asv_percentages.tsv)
- Observed vs. expected fold-difference and p-value of significant-genus overlap per tool (e.g., ALDEx2 1.353, p<0.001)
- False-positive percentages per tool under randomized sample-group labels
- ROC/AUROC, precision, recall, F1 summaries per tool on relative-abundance and CLR inputs

## How to apply

Point the pipeline config.sh (DATA_DIR at the figshare 16S rRNA Microbiome Datasets deposit 14531724, ANCOM_DIR and TOOL_DIR at the repo's Ancom2_Script and Tool_scripts folders) under R 3.6.3 with pinned package versions (ALDEx2 1.18.0, DESeq2 1.26.0, edgeR 3.28.1, metagenomeSeq 1.28.2, corncob 0.1.0, Maaslin2 0.99.12, phyloseq 1.29.0, compositions 1.40.2), with LEfSe in a conda env named 'hackathon'. Run 'run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1' where -D is the rarefaction depth (2000) and -F is the prevalence filter (0.1 = 10%); omit these flags to run the unfiltered variant, and drive the 38-dataset sweeps with run_0.1_filt_datasets.sh / run_no_filt_datasets.sh. The metadata must be a two-column TSV (sample name, group; maximum 2 groupings), and the ANCOM feature table must be absolute, not relative, abundance. Count significant ASVs per tool per dataset and divide by tested ASVs to get percentage significant (e.g., filtered ANCOM-II mean 3.8%, edgeR 32.5%), assess cross-study consistency as observed-vs-expected overlap of significant genera with a permutation null, and gauge false positives by permuting case/control labels. Because results depend heavily on pre-processing, follow the paper's recommendation of a consensus approach: focus on features flagged by most tools and explicitly report the application of multiple DA methods.

## Related tools

- **ALDEx2** (CLR-based DA test; most consistent across studies and highest precision (0.82 at AUROC 0.7) alongside ANCOM-II)
- **ANCOM-II** (Most stringent tool on 10% prevalence-filtered data (mean 3.8% significant ASVs) and top precision (0.99))
- **DESeq2** (Negative-binomial count-based DA tool run via run_all_tools.sh)
- **edgeR** (Negative-binomial DA tool; loosest caller (mean 32.5% significant ASVs filtered) and highest false-positive outlier (0.69–27.9%))
- **LEfSe** (Percentage-then-rank-based DA tool run in the 'hackathon' conda environment)
- **corncob** (Beta-binomial DA model run on each dataset via the pipeline)
- **metagenomeSeq** (Zero-inflated Gaussian DA model in the 14-tool suite)
- **Maaslin2** (Multivariable DA framework run per dataset)
- **limma voom** (voom-transformed linear-model DA variant (TMMwsp unfiltered mean 40.5% significant ASVs))
- **ANCOM** (Additive log-ratio DA tool (archived standalone functions); excluded from extended false-positive replicates) — https://www.bioconductor.org/packages/release/bioc/html/ANCOMBC.html

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Per-tool mean percentages of significant ASVs must reproduce reported values (e.g., limma voom TMMwsp unfiltered mean 40.5%; ANCOM-II filtered mean 3.8%, SD 5.9%; edgeR filtered mean 32.5%).
- Tool outputs on the Human-ALL (MALL) dataset must match the deposited expected results in Misc_datafiles/Expected_MALL_output (~20 min on a single core).
- Cross-study consistency fold differences must match Table 2 (ALDEx2 1.353, p<0.001; edgeR 1.105, p=0.002) with one fold-difference row per DA tool.
- Randomized-label false-positive rates must fall in reported ranges (edgeR filtered mean 0.69–27.9%; ALDEx2 and t-test on rarefied data 0% unfiltered).
- Discrimination metrics must match Supplementary Fig. 3–4 (ANCOM-II precision 0.99, ALDEx2 0.82 at AUROC threshold 0.7 on relative abundance; Wilcoxon (CLR) F1 0.70).

## Limitations

- The randomized approach for estimating FDR is not a perfect representation of real data, so few significant ASVs under it do not prove low practical FDR.
- The 10% prevalence-filter cut-off was arbitrarily selected; the optimal threshold is unknown.
- The cross-study consistency null assumed all genera equally likely to be significant by chance, which is invalid due to prevalence differences; obesity-dataset consistency was no better than chance for most tools.
- The intersection of tool outputs is not guaranteed to be more reliable — tools may simply be picking up the same noise.
- No direct simple recommendation of a single tool could be made from the analysis.

## Evidence

- [abstract] 14 tools, consensus: "ALDEx2 and ANCOM-II produce the most consistent results across studies and agree best with the intersect of results from different approaches."
- [readme] prevalence filter flag: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [readme] filtered invocation: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [discussion] consensus recommendation: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing, although we acknowledge that more work is needed to estimate an optimal cut-off rather than just"
- [readme] ANCOM input format: [evidence span withheld — anchor status: not_found]
