---
name: prevalence-filtering-16s-count-tables
description: Use when preparing ASV/OTU abundance tables for differential abundance
  (DA) testing in microbiome studies where the effect of rare-taxa pre-processing
  on tool output must be controlled or compared across datasets.
when_to_use_negative:
- The goal is computing diversity metrics rather than DA testing — the authors state
  that whether rarefying is advisable for other analyses is beyond the scope of this
  work
- You need a statistically optimal rare-feature cut-off — the 10% threshold was arbitrarily
  selected and the optimal cut-off is unknown
- Filtering must depend on the test statistic or be applied after testing — the article
  stresses filtering should be independent of the test statistic (Independent Filtering)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3308
tools:
- name: ALDEx2
  role: DA tool run on the 10% prevalence-filtered tables; among the most consistent
    tools in the comparison
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ANCOM-II
  role: DA tool run on filtered tables; most stringent (mean 3.8% significant ASVs
    on 10%-filtered data)
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: DESeq2
  role: DA tool run on filtered tables (negative binomial model), including rarefied
    and CLR variants
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: edgeR
  role: DA tool run on filtered tables; identified the most significant ASVs (mean
    32.5% on filtered data)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: LEfSe
  role: DA tool run on filtered tables in the 'hackathon' conda environment; identified
    the most significant ASVs (mean 27.5%)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: corncob
  role: DA tool run on filtered tables (beta-binomial model)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: metagenomeSeq
  role: DA tool run on filtered tables (zero-inflated Gaussian model)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: Maaslin2
  role: DA tool run on filtered tables
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: limma voom
  role: DA tool run on filtered tables (TMMwsp normalization); unfiltered mean 40.5%
    significant ASVs used as a validation reference
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:3d094e89d901722495288be495a781a6a89624617a6a05e6c8fb51c010af40b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - ALDEx2
  - ANCOM-II
  - DESeq2
  - LEfSe
  - Maaslin2
  - corncob
  - edgeR
  - limma voom
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# prevalence-filtering-16s-count-tables

## Summary

Filters rare ASVs from 16S rRNA gene count tables at a prevalence threshold (10% in this study) before running the 14 differential abundance tools, so that tools are tested on features present in a non-trivial fraction of samples and results under filtered vs. unfiltered pre-processing can be compared.

## When to use

Use when preparing ASV/OTU abundance tables for differential abundance (DA) testing in microbiome studies where the effect of rare-taxa pre-processing on tool output must be controlled or compared across datasets.

## When NOT to use

- The goal is computing diversity metrics rather than DA testing — the authors state that whether rarefying is advisable for other analyses is beyond the scope of this work
- You need a statistically optimal rare-feature cut-off — the 10% threshold was arbitrarily selected and the optimal cut-off is unknown
- Filtering must depend on the test statistic or be applied after testing — the article stresses filtering should be independent of the test statistic (Independent Filtering)

## Inputs

- ASV table TSV (features x samples, absolute abundances) from the figshare 16S rRNA Microbiome Datasets (14531724)
- Two-column metadata TSV (sample names, sample groupings, max 2 groups)
- config.sh with DATA_DIR, ANCOM_DIR, and TOOL_DIR paths set

## Outputs

- Per-tool DA result tables computed on the 10% prevalence-filtered ASV tables (38 datasets x 14 tools)
- Counts of significant ASVs per tool per dataset suitable for Fig. 1a/b-style aggregation (e.g. significant_asv_percentages.tsv)
- Filtered-dataset outputs comparable to unfiltered runs for assessing pre-processing sensitivity

## How to apply

Run the pipeline's run_all_tools.sh with -F 0.1 to apply a 10% prevalence filter, so features present in fewer than 10% of samples are excluded before DA testing; optionally set -D 2000 to rarefy to depth 2000 for the rarefied tool variants (DESeq2/edgeR/limma voom rarefied vs. CLR variants). Use run_0.1_filt_datasets.sh to rerun the full 14-tool suite (ALDEx2 1.18.0, ANCOM-II, corncob 0.1.0, DESeq2 1.26.0, edgeR 3.28.1, LEfSe, MaAsLin2 0.99.12, metagenomeSeq 1.28.2, limma voom TMMwsp, and rarefied/CLR test variants) across all 38 datasets and compare against run_no_filt_datasets.sh output. The rationale is that filtering must be independent of the test statistic (Independent Filtering), and prevalence filtering is the authors' recommended pre-processing step, though the 10% threshold was arbitrarily selected rather than optimized.

## Related tools

- **ALDEx2** (DA tool run on the 10% prevalence-filtered tables; among the most consistent tools in the comparison) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ANCOM-II** (DA tool run on filtered tables; most stringent (mean 3.8% significant ASVs on 10%-filtered data)) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **DESeq2** (DA tool run on filtered tables (negative binomial model), including rarefied and CLR variants) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **edgeR** (DA tool run on filtered tables; identified the most significant ASVs (mean 32.5% on filtered data)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **LEfSe** (DA tool run on filtered tables in the 'hackathon' conda environment; identified the most significant ASVs (mean 27.5%)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **corncob** (DA tool run on filtered tables (beta-binomial model)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **metagenomeSeq** (DA tool run on filtered tables (zero-inflated Gaussian model)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **Maaslin2** (DA tool run on filtered tables) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **limma voom** (DA tool run on filtered tables (TMMwsp normalization); unfiltered mean 40.5% significant ASVs used as a validation reference) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Per-tool mean percentages of significant ASVs on filtered data reproduce reported values, e.g. ANCOM-II filtered mean 3.8% (SD 5.9%) and edgeR filtered mean 32.5%
- Filtering is applied at exactly -F 0.1 (10% prevalence) and rarefied variants use -D 2000; any other depth/threshold invalidates comparison to the paper
- Per-tool outputs on the Human-ALL (MALL) dataset match the deposited expected results in Misc_datafiles/Expected_MALL_output
- Filtered and unfiltered runs produce differing numbers of significant ASVs per tool, demonstrating that pre-processing materially changes results
- The number of ASVs tested after filtering is less than or equal to the unfiltered ASV count for every dataset

## Limitations

- The 10% prevalence cut-off was arbitrarily selected; more work is needed to estimate an optimal cut-off
- It remains unclear whether filtering rare taxa has much effect on DA results in practice
- Filtering must be independent of the test statistic (Independent Filtering), so it cannot be tuned per tool based on significance outcomes
- Whether rarefying is advisable for other analyses, such as computing diversity metrics, is beyond the scope of this work

## Evidence

- [readme] README: filter flag semantics: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [readme] README: filtered invocation: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [discussion] Discussion: recommendation and arbitrary cut-off: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing, although we acknowledge that more work is needed to estimate an optimal cut-off"
- [intro] Intro: unclear effect of filtering: "it remains unclear whether filtering rare taxa has much effect on DA results in practice"
- [intro] Methods: filtering must be test-statistic independent: [evidence span withheld — anchor status: not_found]
- [abstract] Methods: pre-processing dependence of results: "these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing"
