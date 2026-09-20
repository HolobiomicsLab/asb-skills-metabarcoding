# amplicon-asv-taxonomy-diversity-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **denoise_asvs** — quality-filter, denoise, merge, and remove chimeras into QIIME 2 ASV artifacts  →  `amplicon-sequence-denoising-dada2`, `asv-denoising`, `asv-feature-table-generation`, `dada2-asv-denoising-and-pair-merging`
2. **assign_taxonomy** — assign reference-backed taxonomy to the representative-sequence artifact  →  `taxonomic-classification-assignment`, `qiime2-artifact-generation`, `taxonomic-assignment-amplicon-sequences`, `qiime2-artifact-import--a9b98430d1674072`
3. **analyze_diversity** — calculate core diversity metrics, ordinations, and metadata-group significance  →  `alpha-beta-diversity-analysis`, `alpha-diversity-group-significance-comparison`, `permanova-group-significance-testing`, `non-rarefied-diversity-calculation`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
