---
name: naive-bayes-16s-region-classifier-training
description: Use when running QIIME 2 or Mothur 16S amplicon taxonomic assignment
  and genus/species-level classification quality depends on the classifier being trained
  on the specific PCR-amplified 16S rRNA region.
when_to_use_negative:
- The analysis uses read-mapping profilers (PathoScope 2 or Kraken 2), which align
  reads directly against a reference genome library and do not use naïve Bayes classifiers
- Only a generic, non-region-specific classifier is acceptable — e.g., exploratory
  work where genus/species-level accuracy of QIIME 2 or Mothur is not being evaluated
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: Runs the DADA2-plugin pipeline whose naïve Bayes classifier is trained on
    the PCR-amplified 16S region using SILVA or Greengenes reference indices
- name: Mothur
  role: Trains its naïve Bayes classifier on the PCR-amplified 16S rRNA gene region
    using Mothur-format SILVA 138 or Greengenes 13_8 reference files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:69b5b30b2611c995fa46a2f60957ec39ff6e263222a8c647f7231e18429d62ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2222222222222222
n_claims: 9
---

# naive-bayes-16s-region-classifier-training

## Summary

Train the naïve Bayes taxonomic classifiers used by QIIME 2 and Mothur on reference libraries (e.g., SILVA 138, Greengenes 13_8) so they are fitted to the exact 16S rRNA gene region amplified by the study's PCR primers, as done in the Faits & Odom-Mabey et al. 16S benchmarking study.

## When to use

Use when running QIIME 2 or Mothur 16S amplicon taxonomic assignment and genus/species-level classification quality depends on the classifier being trained on the specific PCR-amplified 16S rRNA region.

## When NOT to use

- The analysis uses read-mapping profilers (PathoScope 2 or Kraken 2), which align reads directly against a reference genome library and do not use naïve Bayes classifiers
- Only a generic, non-region-specific classifier is acceptable — e.g., exploratory work where genus/species-level accuracy of QIIME 2 or Mothur is not being evaluated

## Inputs

- 16S amplicon sequencing reads (FASTQ) from mock community samples
- SILVA 138 reference indices formatted for QIIME 2 (from QIIME 2 website) and Mothur (from Mothur website)
- Greengenes 13_8 reference indices for QIIME 2 (QIIME 2 website) and Mothur (gg_13_8_99 taxonomy files)
- Primer sequences defining the PCR-amplified 16S rRNA gene region

## Outputs

- Trained naïve Bayes classifier objects for QIIME 2 and/or Mothur
- Per-sample taxonomic assignments at genus and species level, aggregatable into a taxon-by-sample counts matrix (one CSV per pipeline/reference-library pairing)

## How to apply

Both QIIME 2 and Mothur assign taxonomy with naïve Bayes classifiers, which perform best when trained on the specific 16S rRNA gene region amplified by the PCR primers rather than on full-length reference sequences. Acquire the pre-formatted reference indices for the chosen library version — e.g., QIIME 2 SILVA indices from the QIIME 2 website, Mothur SILVA reference files from the Mothur website, Mothur Greengenes 13_8 (gg_13_8_99 taxonomy) from the Mothur website — then train each tool's classifier on the PCR-amplified region before classifying the mock community amplicon reads. In the benchmarking workflow this training step was applied per pipeline/reference-library pairing (QIIME 2 with Greengenes and SILVA; Mothur with SILVA 138 and Greengenes 13_8) before aggregating per-sample taxonomic calls into a taxon-by-sample counts matrix. Rationale: the paper found that QIIME 2 and Mothur underperformed read-mapping tools (PathoScope 2, Kraken 2), and proper region-specific training is the documented way to give the naïve Bayes classifiers their best chance at genus- and species-level accuracy.

## Related tools

- **QIIME 2** (Runs the DADA2-plugin pipeline whose naïve Bayes classifier is trained on the PCR-amplified 16S region using SILVA or Greengenes reference indices)
- **Mothur** (Trains its naïve Bayes classifier on the PCR-amplified 16S rRNA gene region using Mothur-format SILVA 138 or Greengenes 13_8 reference files)

## Evaluation signals

- The classifier was trained on reference sequences trimmed to the exact PCR-amplified 16S rRNA gene region defined by the study primers, not full-length references
- The correct library version and formatting were used: SILVA 138 for QIIME 2/Mothur and Greengenes 13_8 (gg_13_8_99 taxonomy) per the repository README sources
- Each pipeline/reference-library pairing yields a complete taxon-by-sample counts matrix with rows as taxa and columns as the 136 mock community samples
- Classifier output supports genus- and species-level calls comparable across the 11 pipeline/reference-library pairings in the benchmark

## Limitations

- The paper found the most-used 16S bioinformatics tools (including naïve Bayes-based QIIME 2 and Mothur) cannot produce high-quality genus- or species-level taxonomic calls, so even well-trained classifiers underperform read-mapping approaches
- SILVA and RefSeq/Kraken 2 Standard libraries were found superior to Greengenes, so Greengenes-trained classifiers are expected to be less accurate
- Mock communities provide only a semblance of ground truth; PCR amplification bias and reference database contamination (e.g., E. coli 16S sequence in T. flagellates RefSeq scaffolds) can confound classifier evaluation

## Evidence

- [discussion] Both QIIME 2 and Mothur take advantage of naïve Bayes classifiers, which work most efficiently when trained on the specific region of the 16S rRNA gene amplified by PCR primers: "Both QIIME 2 and Mothur take advantage of naïve Bayes classifiers, which work most efficiently when trained on the specific region of the 16S rRNA gene amplified by PCR primers"
- [readme] Mothur SILVA indices were obtained from the Mothur website: "Mothur SILVA indices were obtained from the [Mothur website.](https://mothur.org/wiki/silva_reference_files)"
- [readme] QIIME 2 Greengenes indices were obtained from the QIIME 2 website: "QIIME 2 Greengenes indices were obtained from the [QIIME 2 website.](https://docs.qiime2.org/2022.2/data-resources/?highlight=greengenes)"
- [other] Execute Mothur with the SILVA 138 and Greengenes 13_8 indices, training its naïve Bayes classifier on the PCR-amplified 16S rRNA gene region: [evidence span withheld — anchor status: not_found]
- [abstract] Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes: "Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes"
