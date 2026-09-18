---
name: rarefied-vs-nonrarefied-validation
description: Use when alpha diversity metrics have been computed on non-rarefied data
  to preserve information, and you need to confirm the metrics' behaviors and category
  conclusions are robust under rarefaction before reporting them.
when_to_use_negative:
- Your feature table is already rarefied and rarefaction robustness is not the question
  — the non-rarefied baseline results must exist first for comparison
- Metrics such as ACE or Faith need recomputation on synthetic datasets — the paper
  reports these could not be recalculated due to technical limitations
- You are validating denoiser sensitivity (DADA2 removes singletons, DEBLUR keeps
  them) rather than rarefaction sensitivity — that requires a different comparison
  design
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME2 2022.2.0
  role: Diversity module used to recompute the 19 alpha diversity metrics on rarefied
    datasets, matching the original non-rarefied calculation
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment hosting QIIME2 for the rarefied and non-rarefied alpha
    diversity calculations
- name: Jupyter
  role: Notebooks used to load rarefied tables and aggregated alpha metric files,
    align values per dataset, and write the comparison output
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:15b8f118f1c1f3f9189623c854761b102e8ffdb19aa52bea776240a1bf37b25a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - QIIME2 2022.2.0
  - anaconda
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# rarefied-vs-nonrarefied-validation

## Summary

Validates that the characterizations of 19 microbial alpha diversity metrics (grouped into Richness, Dominance, Phylogenetics, and Information categories) hold when metrics are recomputed on rarefied datasets and compared metric-by-metric against the primary non-rarefied results computed with QIIME2's diversity module.

## When to use

Use when alpha diversity metrics have been computed on non-rarefied data to preserve information, and you need to confirm the metrics' behaviors and category conclusions are robust under rarefaction before reporting them.

## When NOT to use

- Your feature table is already rarefied and rarefaction robustness is not the question — the non-rarefied baseline results must exist first for comparison
- Metrics such as ACE or Faith need recomputation on synthetic datasets — the paper reports these could not be recalculated due to technical limitations
- You are validating denoiser sensitivity (DADA2 removes singletons, DEBLUR keeps them) rather than rarefaction sensitivity — that requires a different comparison design

## Inputs

- Rarefied dataset tables (feature tables per dataset)
- Per-dataset aggregated alpha metric files (alpha-out_dataset-name.tsv)
- Sample metadata per dataset (SraRunTable.txt or similar accession info from ./datasets/xx_dataset_alias/)
- Repository Jupyter notebooks and requirements.txt

## Outputs

- rarefied_vs_nonrarefied_alpha_metrics.tsv (per-metric, per-dataset comparison table)
- Per-dataset alpha diversity metric values recomputed under rarefaction
- Confirmation that conclusions for the Richness, Dominance, Phylogenetics, and Information categories are unchanged

## How to apply

Set up the analysis environment (Python venv with requirements.txt, or an anaconda environment with QIIME2 2022.2.0) and load the repository's Jupyter notebooks. Load the rarefied dataset tables together with the per-dataset aggregated metric files (alpha-out_dataset-name.tsv, later combined as alpha_aggregated.tsv with sample metadata). Recompute all 19 alpha diversity metrics on the rarefied data using QIIME2's diversity module, mirroring the pipeline used for the 13 publicly available human microbiome projects (4,596 stool samples). Align rarefied and non-rarefied metric values per dataset and compare them metric-by-metric across all datasets, checking that conclusions for the four metric categories (Richness, Dominance, Phylogenetics, Information) are unchanged. Rationale: non-rarefied calculation preserves as much information as possible, and rarefied recomputation is the standard robustness check that the metric behaviors are not artifacts of uneven sequencing depth.

## Related tools

- **QIIME2 2022.2.0** (Diversity module used to recompute the 19 alpha diversity metrics on rarefied datasets, matching the original non-rarefied calculation) — https://qiime2.org/
- **anaconda** (Virtual environment hosting QIIME2 for the rarefied and non-rarefied alpha diversity calculations)
- **Jupyter** (Notebooks used to load rarefied tables and aggregated alpha metric files, align values per dataset, and write the comparison output)

## Examples

```
python -m venv venv/; source venv/bin/activate; pip install -r requirements.txt
```

## Evaluation signals

- Every one of the 19 metrics is recomputed on rarefied data for each of the 13 stool-sample projects, with no missing metric-dataset pairs (ACE/Faith exceptions only apply to the synthetic datasets, not this comparison)
- Rarefied metric values align sample-for-sample with the non-rarefied values from alpha-out_dataset-name.tsv / alpha_aggregated.tsv before comparison
- The comparison table rarefied_vs_nonrarefied_alpha_metrics.tsv is produced and shows that conclusions for all four metric categories (Richness, Dominance, Phylogenetics, Information) are unchanged under rarefaction
- Any behavior changes after min-max normalization of metric values are absent or not meaningful, consistent with the paper's normalization-based behavior analysis
- Workflow version matches the paper (QIIME 2022.2.0) or a newer compatible version is documented

## Limitations

- Rarefied dataset availability is only described as 'data available in repository' without explicit accession or location, so retrieval may require exploring dataset folders for accession numbers
- Raw sample data is not shared in the repository due to size; only accession numbers are provided, so full replication requires re-downloading from SRA via sra-tools
- ACE and Faith could not be recalculated on synthetic datasets due to technical limitations, and Faith is algorithmic and 'continuously suffering changes', so its rarefied validation may be less stable
- Workflows were run on QIIME 2022.2.0; newer versions should be compatible but are untested in this context

## Evidence

- [results] Alpha diversity metrics calculated with non-rarefied data; results also validated using rarefied datasets: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets"
- [readme] All alpha diversity metrics were caluclated using QIIME2's diversity module.: "All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv."
- [readme] Workflow version and compatibility: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
- [results] Dataset scale of the validation: "calculated for the sequence datasets obtained for a total of 4,596 stool samples included in 13 publicly available human microbiome projects"
