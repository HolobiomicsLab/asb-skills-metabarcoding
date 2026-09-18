---
name: 16s-genus-level-taxonomic-profiling
description: Use when 16S rRNA amplicon reads (e.g., QIAseq-panel Illumina libraries
  from sputum) need genus-level ASV profiling and Greengenes-based taxonomy, or when
  comparing hypervariable-region accuracy against a mock community.
when_to_use_negative:
- Third-generation full-length 16S rRNA sequencing (e.g., Nanopore) is available —
  this skill is calibrated to the 300 bp Illumina MiSeq limit and the study did not
  compare against full-length reads
- Your pipeline requires the most current reference taxonomy — this workflow trains
  regions with the outdated Greengenes database rather than updated SILVA versions
- The sample type or regions differ (e.g., non-respiratory niches or V4–V6 comparisons)
  — findings are specific to sputum and the V1–V2/V3–V4/V5–V7/V7–V9 combinations
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: Deblur
  role: Denoises paired-end 16S reads and infers bacterial amplicon sequence variants
    (ASVs) at the genus level from QIAseq/Illumina libraries
- name: FastQC
  role: Version 0.11.9 assesses Q30 quality and reads per sample and filters raw reads
    with a cut-off score of up to 3011
- name: Greengenes
  role: Reference database from which taxonomic assignments are obtained and collapsed
    at the genus level
- name: QIIME 2
  role: Provides the non-chimeric 16S rRNA database used to identify chimeras (best
    hit similarity rate < 99%)
- name: DADA2
  role: Alternative denoiser reported to identify more real variants and output sequences
    than other methods
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:93bd0b190eab902464e9c143c528d7896927287178868f72373c2e00f1ca8c00
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Deblur
  - FastQC
  - Greengenes
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# 16s-genus-level-taxonomic-profiling

## Summary

Infers bacterial amplicon sequence variants (ASVs) from 16S rRNA reads with Deblur and assigns Greengenes taxonomy collapsed at the genus level for respiratory (sputum) samples. Accuracy is benchmarked per combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) against the ZymoBIOMICS mock community via ROC curves.

## When to use

Use when 16S rRNA amplicon reads (e.g., QIAseq-panel Illumina libraries from sputum) need genus-level ASV profiling and Greengenes-based taxonomy, or when comparing hypervariable-region accuracy against a mock community.

## When NOT to use

- Third-generation full-length 16S rRNA sequencing (e.g., Nanopore) is available — this skill is calibrated to the 300 bp Illumina MiSeq limit and the study did not compare against full-length reads
- Your pipeline requires the most current reference taxonomy — this workflow trains regions with the outdated Greengenes database rather than updated SILVA versions
- The sample type or regions differ (e.g., non-respiratory niches or V4–V6 comparisons) — findings are specific to sputum and the V1–V2/V3–V4/V5–V7/V7–V9 combinations

## Inputs

- Paired-end 16S rRNA amplicon FASTQ reads from 33 sputum samples (QIAseq 16S/ITS screening panel libraries, Illumina MiSeq, ≤300 bp)
- ZymoBIOMICS microbial community standard control sequencing data (10-species mock)
- Greengenes reference database for taxonomic assignment
- Greengenes-collapsed genus-level ASV abundance table grouped by combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9)
- Standard-of-care culture results (for cross-checking recovered genera)

## Outputs

- FastQC quality summaries (Q30 read counts, per-sequence scores)
- Non-chimeric ASV set inferred by Deblur
- Greengenes-collapsed genus-level ASV abundance table per hypervariable region
- ROC curves / AUC per combined region for classifier accuracy versus ZymoBIOMICS
- LEfSe discriminant-genera table with log10 LDA scores (threshold > 2) ranked per region

## How to apply

Assess raw reads with FastQC (version 0.11.9), confirming Q30-quality sequences and reads per sample, and filter raw reads with a cut-off score of up to 3011. Identify chimeras using the non-chimeric 16S rRNA database from QIIME 2, discarding reads whose best hit similarity rate is less than 99%. Denoise paired-end data with the Deblur pipeline (deleting non-redundant reads) to infer bacterial ASVs at the genus level; DADA2 is the reported alternative that identified more real variants and output sequences. Assign taxonomy using the Greengenes database collapsed at the genus level, keeping the genus-level ASV abundance table grouped by combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9). Validate by ROC-curve cross-validation of the microbiota classifier against the 10-species ZymoBIOMICS mock standard (V1–V2 gave the highest AUC, sensitivity, and specificity) and cross-check discriminant genera against standard-of-care cultures. For discriminant genera, run LEfSe on the region-grouped genus table with an LDA threshold > 2 after scaling relative abundances to (1, 10exp9) and computing log10-transformed LDA scores, supported by the default 30-fold bootstrapping; genus tables are available in the authors' GitHub repository.

## Related tools

- **Deblur** (Denoises paired-end 16S reads and infers bacterial amplicon sequence variants (ASVs) at the genus level from QIAseq/Illumina libraries)
- **FastQC** (Version 0.11.9 assesses Q30 quality and reads per sample and filters raw reads with a cut-off score of up to 3011)
- **Greengenes** (Reference database from which taxonomic assignments are obtained and collapsed at the genus level)
- **QIIME 2** (Provides the non-chimeric 16S rRNA database used to identify chimeras (best hit similarity rate < 99%))
- **DADA2** (Alternative denoiser reported to identify more real variants and output sequences than other methods)

## Examples

```
run_lefse.py genus_abundances_lefse.txt -o lda_scores.txt -l 2.0 -n 30
```

## Evaluation signals

- FastQC output shows Q30-quality sequences and raw reads were filtered with a cut-off score of up to 3011
- Chimeric reads are flagged where best hit similarity to the QIIME 2 non-chimeric 16S rRNA database is less than 99%
- The final table is a Greengenes-collapsed genus-level ASV table; rarefaction curves estimated in R (version 3.4.1) indicate adequate sequencing depth
- ROC cross-validation against the 10-species ZymoBIOMICS control reproduces V1–V2 having the highest AUC, sensitivity, and specificity among the four regions
- LEfSe genera pass the LDA threshold > 2 (log10 LDA scores, abundances scaled to (1, 10exp9), 30-fold bootstrap) and reproduce distinct per-region genus sets (e.g., Pseudomonas recovered only in V1–V2)

## Limitations

- The 300 bp length limit of the Illumina MiSeq platform limits 16S rRNA profiling, and no comparison to full-length 16S rRNA sequencing was performed
- Regions were trained with the outdated Greengenes database despite the availability of more updated SILVA versions
- The ZymoBIOMICS mock microbial community contains only 10 species, which affects sensitivity and specificity of taxonomic classification
- Richness/entropy are niche-dependent, so results from sputum of chronic respiratory disease patients lack direct comparability with other studies; suitable Nanopore platforms are not yet widely available for clinical application

## Evidence

- [abstract] Deblur infers bacterial ASVs at genus level: "We used the Deblur algorithm to identify bacterial amplicon sequence variants (ASVs) at the genus level"
- [abstract] V1–V2 most sensitive/specific versus mock control: "Comparing the taxa of microbial community standard control with the taxa samples, V1–V2 combination exhibits the most sensitivity and specificity."
- [methods] Greengenes taxonomy collapsed at genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [methods] Chimera removal via QIIME 2 non-chimeric database: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
- [results] FastQC Q30 quality assessment: "FastQC was used to obtain quality sequences with Q30 and to observe the number of reads per sample"
- [discussion] Mock community has only 10 species (classification limitation): "mock microbial communities there are only 10 species and this fact affects in the sensitivity and specificity related with taxonomic classification"
- [discussion] Regions trained with outdated Greengenes instead of SILVA: "despite the availability of more updated versions of SILVA, the different regions were trained with Greengenes"
