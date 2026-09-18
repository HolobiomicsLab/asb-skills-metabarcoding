---
name: amplicon-sequence-variant-fetching-via-redbiom
description: Use when a downstream step (e.g., q2-clawback assemble_weights_from_Qiita)
  needs the sequence variants of samples sharing a metadata category value (e.g.,
  an EMPO 3 habitat) and those samples live in Qiita.
when_to_use_negative:
- You need to load or modify data in redbiom — the default host is read-only and requires
  a locally installed Redis/Webdis instance with REDBIOM_HOST set
- The target samples are not in Qiita/redbiom or lack a searchable metadata category,
  so bespoke weights cannot be assembled for them
- You already hold a feature table locally and only need per-sample sequence variants
  — use q2-clawback's sequence_variants_from_samples directly instead of re-fetching
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0622
tools:
- name: redbiom
  role: Cache service used to search Qiita samples by metadata and fetch their sample
    data (sequence variants) into BIOM for a chosen context
  repo: biocore/redbiom
- name: q2-clawback
  role: Provides fetch_Qiita_samples and summarize_Qiita_metadata_category_and_contexts,
    which wrap redbiom retrieval for weight assembly
  repo: github.com/BenKaehler/q2-clawback
- name: Qiita
  role: Source sample-metadata and processing-data repository queried through redbiom
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:9db2031c3de4a65c5a524d45858a8291868f41a4f40243bc845190eaf597ee92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/redbiom
  tools:
  - QIIME 2
  - Qiita
  - q2-clawback
  - redbiom
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# amplicon-sequence-variant-fetching-via-redbiom

## Summary

Retrieves amplicon sequence variants (features) for environment-specific sample sets from Qiita via the redbiom cache service, producing BIOM feature tables that feed q2-clawback weight assembly.

## When to use

Use when a downstream step (e.g., q2-clawback assemble_weights_from_Qiita) needs the sequence variants of samples sharing a metadata category value (e.g., an EMPO 3 habitat) and those samples live in Qiita.

## When NOT to use

- You need to load or modify data in redbiom — the default host is read-only and requires a locally installed Redis/Webdis instance with REDBIOM_HOST set
- The target samples are not in Qiita/redbiom or lack a searchable metadata category, so bespoke weights cannot be assembled for them
- You already hold a feature table locally and only need per-sample sequence variants — use q2-clawback's sequence_variants_from_samples directly instead of re-fetching

## Inputs

- Qiita metadata search term or category/value (e.g., an EMPO 3 habitat label)
- redbiom context name (processing type, e.g., 16S V4 Greengenes closed-reference OTU picking)
- redbiom host configuration (default qiita.ucsd.edu:7329, or REDBIOM_HOST environment variable)

## Outputs

- BIOM feature table of fetched samples and their sequence variants (e.g., beer_example.biom)
- Optional sample metadata table via 'redbiom fetch sample-metadata --context <ctx>'
- List of matching sample IDs from 'redbiom search metadata'

## How to apply

Enumerate the target Qiita metadata category and available processing contexts (e.g., with q2-clawback's summarize_Qiita_metadata_category_and_contexts), then use redbiom to find matching samples — e.g., 'redbiom search metadata <term>' — and fetch their sample data with 'redbiom fetch samples --context <ctx> --output <file>.biom'. The context (e.g., Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506) selects the processing type to search over (16S vs WGS, OTU picking vs ASVs). Within a context, per-study data are distinguished by tags, so sample IDs may appear as '<tag>_<sample-id>' internally and are coerced to '<sample-id>.<tag>' on write; redbiom warns on ambiguities where one sample ID maps to multiple redbiom IDs. The default host qiita.ucsd.edu:7329 is read-only, so retrieval works out of the box but loading data requires a local redbiom instance via REDBIOM_HOST. The resulting BIOM table is then passed to sequence_variants_from_samples to extract the variants for classifier-based weight assembly.

## Related tools

- **redbiom** (Cache service used to search Qiita samples by metadata and fetch their sample data (sequence variants) into BIOM for a chosen context) — biocore/redbiom
- **q2-clawback** (Provides fetch_Qiita_samples and summarize_Qiita_metadata_category_and_contexts, which wrap redbiom retrieval for weight assembly) — github.com/BenKaehler/q2-clawback
- **Qiita** (Source sample-metadata and processing-data repository queried through redbiom)

## Examples

```
export ctx=Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506; redbiom search metadata beer | head | redbiom fetch samples --context $ctx --output beer_example.biom
```

## Evaluation signals

- Output BIOM file exists and is loadable (requires biom-format >= 2.1.5) with one column per fetched sample
- Sample IDs in the output are valid QIIME 2 IDs of the form <sample-id>.<tag> — redbiom flags any ambiguities where a sample ID mapped to multiple redbiom IDs
- The chosen --context matches the processing type of interest (e.g., 16S V4 Greengenes) rather than an unrelated preparation
- Feature IDs correspond to the expected reference space for the context (e.g., Greengenes OTU IDs as in 'redbiom search taxon --context $ctx s__aureus')
- Downstream sequence_variants_from_samples / assemble_weights_from_Qiita consume the table without empty-sample errors, yielding classified abundances for every fetched sample

## Limitations

- The default redbiom host (qiita.ucsd.edu:7329) is read-only; loading your own data requires a local server with Redis and Webdis and an explicit REDBIOM_HOST setting
- Searches and fetches are context-specific: data loaded under different processing types (tags) are separate, and mismatched tags create ambiguous sample IDs
- redbiom supports one-to-many sample-metadata-to-data relationships (biological/technical replicates), so a metadata search can return multiple tagged copies of the same sample

## Evidence

- [readme] redbiom is a cache service: [evidence span withheld — anchor status: not_found]
- [readme] context definition: [evidence span withheld — anchor status: external_doc]
- [readme] fetch example: [evidence span withheld — anchor status: external_doc]
- [readme] read-only default host: [evidence span withheld — anchor status: external_doc]
- [discussion] Qiita weight assembly motivation: "it is trivial to assemble weights for all stool samples with human hosts from Qiita"
