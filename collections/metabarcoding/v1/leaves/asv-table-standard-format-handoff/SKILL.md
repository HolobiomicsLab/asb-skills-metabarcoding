---
name: asv-table-standard-format-handoff
description: Use when a DADA2/dadasnake amplicon workflow has finished denoising and
  taxonomy assignment and the final ASV/OTU table must be exported in community-standard
  formats for downstream analysis scripts.
when_to_use_negative:
- Input is already a feature table in the target format (e.g. an existing .biom file)
  — no export step is needed
- 'The workflow has not finished: no ''workflow.done'' file exists in the output directory
  and denoising/taxonomy rules are still running'
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_0699
- http://edamontology.org/topic_3174
tools:
- name: dadasnake
  role: Snakemake workflow whose postprocessing rules perform the final-table filtering
    and multi-format export (biom, phyloseq, text, R data sets)
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: Produces the denoised ASVs, sequences, and taxonomy that constitute the table
    being handed off
  repo: http://benjjneb.github.io/dada2/
- name: Snakemake
  role: Orchestrates the postprocessing rules and signals completion via the 'workflow.done'
    file in the output directory
  repo: https://snakemake.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:32741f1454dc7a0289334b5d1607487695409347ce7856da9ce245b4b64833e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_0699
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3
n_claims: 10
---

# ASV table standard-format hand-off

## Summary

Post-processing step of the dadasnake workflow that converts the final ASV table (>700,000 ASVs in the EMP use case) into interoperable formats — biom, R/phyloseq objects, FASTA, and tab-separated tables — for downstream microbial ecology analyses.

## When to use

Use when a DADA2/dadasnake amplicon workflow has finished denoising and taxonomy assignment and the final ASV/OTU table must be exported in community-standard formats for downstream analysis scripts.

## When NOT to use

- Input is already a feature table in the target format (e.g. an existing .biom file) — no export step is needed
- The workflow has not finished: no 'workflow.done' file exists in the output directory and denoising/taxonomy rules are still running

## Inputs

- Final ASV table (tab-separated counts per sample) from dadasnake postprocessing
- ASV sequences (from DADA2 denoising)
- Taxonomic classification results per ASV
- final_table_filtering settings from the dadasnake config yaml

## Outputs

- ASV table in biom format
- R/phyloseq object (RData)
- Tab-separated ASV/OTU table
- FASTA file of ASV sequences

## How to apply

After dadasnake completes (evidenced by a 'workflow.done' file in the output directory), run the final-table handling step on the denoised, chimera-removed ASV table. Configure the output directory and final_table_filtering settings in the human-readable yaml config so the post-processing step filters the final ASV and/or OTU tables before export. Export the same table in parallel representations: biom format for QIIME-style ecosystems, R/phyloseq objects for microbiome R users, and flexible text (tab-separated) files or R data sets for generic scripting; the ASV sequences themselves are handed off as a FASTA file. Rationale: this multi-format hand-off is a core design goal of dadasnake so that results from workflows processing 2.1 billion reads across >27,000 EMP samples integrate directly into microbial ecology analysis pipelines without reformatting.

## Related tools

- **dadasnake** (Snakemake workflow whose postprocessing rules perform the final-table filtering and multi-format export (biom, phyloseq, text, R data sets)) — https://github.com/a-h-b/dadasnake
- **DADA2** (Produces the denoised ASVs, sequences, and taxonomy that constitute the table being handed off) — http://benjjneb.github.io/dada2/
- **Snakemake** (Orchestrates the postprocessing rules and signals completion via the 'workflow.done' file in the output directory) — https://snakemake.readthedocs.io/en/stable/

## Examples

```
dadasnake -c -n RUNNAME -r config.yaml -t 50  # then verify workflow.done and collect biom, phyloseq, fasta, and tab-separated outputs
```

## Evaluation signals

- A 'workflow.done' file appears in the dadasnake output directory, indicating the postprocessing (including export) completed
- All four hand-off artifacts exist and are internally consistent: biom table, phyloseq object, tab-separated table, and ASV FASTA share the same ASV set and sample names
- Biom file validates/loads as valid biom format and the phyloseq object imports into R without sample/ASV mismatches
- In the EMP use case, the final table contains >700,000 ASVs and peak memory usage during final table handling is >190 GB — substantially smaller or larger tables suggest an upstream filtering or export error
- dadasnake's intuitive figures and tables reporting intermediary steps and statistics reflect the final filtering applied to the exported table

## Limitations

- Final table filtering settings must be chosen deliberately; postprocessing can remove ASVs before hand-off, so consumers must know which final_table_filtering settings were applied
- The full-table export of the >700,000-ASV EMP table requires >190 GB RAM, so small-memory nodes may fail during this step
- Output format coverage is fixed by dadasnake (biom, phyloseq, text/R data sets, FASTA); other ecosystems require further conversion

## Evidence

- [intro] Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts: "Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts"
- [abstract] with a taxonomic classification and the post-processing of the resultant tables: "with a taxonomic classification and the post-processing of the resultant tables"
- [methods] final_table_filtering|||||postprocessing||settings for filtering the final ASV and/or OTU tables: "final_table_filtering|||||postprocessing||settings for filtering the final ASV and/or OTU tables"
- [other] During final table handling, record the wall-clock duration (expected ~87 real hours) and peak memory usage (expected >190 GB) for the >700,000-ASV table: [evidence span withheld — anchor status: not_found]
- [other] Hand off the final ASV table in biom format, as R/phyloseq objects, and as fasta and tab-separated tables.: [evidence span withheld — anchor status: not_found]
