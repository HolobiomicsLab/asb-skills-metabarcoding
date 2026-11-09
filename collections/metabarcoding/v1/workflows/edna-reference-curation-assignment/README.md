# edna-reference-curation-assignment-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **assemble_reference** — retrieve COI records, screen human contaminants, and preserve complete lineages  →  `co1-reference-sequence-assembly`, `taxonomic-lineage-retrieval-and-formatting`, `reference-database-validation-against-deposit`, `rdp-training-set-construction`
2. **curate_reference** — filter taxonomy, dereplicate identical sequences, reconcile conflicts, and cluster centroids  →  `coi-reference-database-curation`, `fasta-reference-database-curation`, `bold-c-oi-sequence-retrieval-replace`
3. **format_reference** — build an OBITools-compatible combined COI reference index  →  `coi-reference-database-formatting`, `cross-release-reference-database-comparison`
4. **assign_motus** — assign MOTUs to the most specific consistent taxon and retain ambiguous matches  →  `metabarcoding-taxonomic-assignment`, `obitools-ecotag-workflow`, `ecotaxonomic-assignment-table-processing`, `taxonomy-table-integration-with-read-counts`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
