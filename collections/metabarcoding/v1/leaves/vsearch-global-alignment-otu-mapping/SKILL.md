---
name: vsearch-global-alignment-otu-mapping
description: Use when a merged, quality-filtered amplicon FASTQ-derived FASTA (e.g.
  temp/filtered.fa) must be quantified against a curated OTU/ASV reference (result/raw/otus.fa)
  to build the raw per-sample feature table result/raw/otutab.txt.
when_to_use_negative:
- Input is already a feature/OTU table — no read-to-reference mapping is needed; proceed
  directly to abundance filtering or normalization (e.g. rarefaction with vegan).
- Reads are not yet merged/quality-filtered (no temp/filtered.fa); run cutadapt primer
  removal and vsearch --fastx_filter first.
- Alignments contain amino-acid sequences or require local alignment — VSEARCH explicitly
  does not support these.
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: VSEARCH
  role: Open-source global-aligner used with --usearch_global to map filtered reads
    to the OTU reference at --id 0.97 and emit the feature table; drop-in replacement
    for USEARCH -otutab.
  repo: https://github.com/torognes/vsearch
- name: USEARCH
  role: Proprietary alternative providing the -otutab command that the VSEARCH mapping
    step re-implements in the EasyAmplicon pipeline.
- name: EasyAmplicon
  role: Pipeline wrapper that supplies the filtered reads, OTU reference, and the
    subsequent R script (otutab_filter_nonBac.R) that consumes the mapped table.
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Supplies the otutab_filter_nonBac.R script and the RDP 16S v18 database used
    for SINTAX annotation of the OTU sequences.
  repo: https://github.com/YongxinLiu/EasyMicrobiome
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:0f7c91b12b5245146aadf3f7c747c05309e84a52f221db9b731272effdc73a60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# vsearch-global-alignment-otu-mapping

## Summary

Constructs the per-sample OTU/ASV feature table by globally aligning all quality-filtered amplicon reads against the dereplicated OTU/ASV reference with VSEARCH --usearch_global (USEARCH -otutab equivalent), producing the raw abundance table that downstream filtering (otutab_filter_nonBac.R) operates on.

## When to use

Use when a merged, quality-filtered amplicon FASTQ-derived FASTA (e.g. temp/filtered.fa) must be quantified against a curated OTU/ASV reference (result/raw/otus.fa) to build the raw per-sample feature table result/raw/otutab.txt.

## When NOT to use

- Input is already a feature/OTU table — no read-to-reference mapping is needed; proceed directly to abundance filtering or normalization (e.g. rarefaction with vegan).
- Reads are not yet merged/quality-filtered (no temp/filtered.fa); run cutadapt primer removal and vsearch --fastx_filter first.
- Alignments contain amino-acid sequences or require local alignment — VSEARCH explicitly does not support these.

## Inputs

- Quality-filtered amplicon sequences FASTA (temp/filtered.fa)
- Dereplicated, chimera-filtered OTU/ASV reference FASTA (result/raw/otus.fa)
- Optional: sample metadata to verify table column count

## Outputs

- Raw per-sample feature/OTU table result/raw/otutab.txt (tab-separated, features x samples)
- Downstream cleaned table result/otutab.txt after otutab_filter_nonBac.R

## How to apply

First complete upstream EasyAmplicon steps: primer removal with cutadapt, length/expected-error filtering with vsearch --fastx_filter, dereplication, and chimera removal (e.g. --uchime_ref against ${db}/usearch/rdp_16s_v18.fa) yielding the OTU reference result/raw/otus.fa. Then run vsearch --usearch_global on the quality-filtered sequences (temp/filtered.fa) with the OTU reference as --db at a 97% identity threshold (--id 0.97) to write the tab-separated per-sample abundance table result/raw/otutab.txt; this mirrors the USEARCH -otutab step but uses VSEARCH's open-source, 64-bit full dynamic-programming (Needleman-Wunsch) global aligner for higher alignment sensitivity. Optionally annotate the OTU sequences with vsearch --sintax against RDP 16S v18 (--sintax_cutoff 0.1) so the table can subsequently be cleaned of Eukarya, Chloroplasts, Mitochondria, and Archaea lineages via the EasyMicrobiome script otutab_filter_nonBac.R. Validate the result with csvtk -t stat: the cleaned table's column count must equal the number of samples in the metadata.

## Related tools

- **VSEARCH** (Open-source global-aligner used with --usearch_global to map filtered reads to the OTU reference at --id 0.97 and emit the feature table; drop-in replacement for USEARCH -otutab.) — https://github.com/torognes/vsearch
- **USEARCH** (Proprietary alternative providing the -otutab command that the VSEARCH mapping step re-implements in the EasyAmplicon pipeline.)
- **EasyAmplicon** (Pipeline wrapper that supplies the filtered reads, OTU reference, and the subsequent R script (otutab_filter_nonBac.R) that consumes the mapped table.) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Supplies the otutab_filter_nonBac.R script and the RDP 16S v18 database used for SINTAX annotation of the OTU sequences.) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
vsearch --usearch_global temp/filtered.fa --db result/raw/otus.fa --id 0.97 --otutabout result/raw/otutab.txt
```

## Evaluation signals

- csvtk -t stat on the cleaned result/otutab.txt reports a column count equal to the number of samples in the metadata.
- Every feature ID in result/raw/otutab.txt matches a sequence header in result/raw/otus.fa.
- Sum of counts per sample (column sums) is non-zero and consistent with post-filter read counts; empty samples indicate a mapping failure.
- The discard file result/raw/otus.sintax.discard contains only Eukarya, Chloroplasts, Mitochondria, or Archaea lineages — bacterial rows must remain in the cleaned table.
- Re-running the mapping at --id 0.97 reproduces the same table (deterministic given the same reference and filtered reads).

## Limitations

- The mapping threshold is fixed at 97% identity in the protocol; sequences divergent from the reference OTUs are dropped from the table.
- Raw tables include plastid, mitochondrial, and archaeal counts that must be removed with otutab_filter_nonBac.R before diversity analysis.
- VSEARCH supports only nucleotide sequences and global alignments — amino-acid queries and local alignment are unsupported by design.
- Exact filter thresholds (fastq_minlen 1200, fastq_maxlen 1800, --fastq_maxee) are amplicon-region specific (full-length 16S) and must be adjusted for other regions.

## Evidence

- [methods] Remove Chloroplasts, Mitochondria, and Non-Bacteria: Typically, an R script (e.g., `otutab_filter_nonBac.R`) is used to filter out Eukarya, Chloroplasts, Mitochondria, and Archaea: "Remove Chloroplasts, Mitochondria, and Non-Bacteria: Typically, an R script (e.g., `otutab_filter_nonBac.R`) is used to filter out Eukarya, Chloroplasts, Mitochondria, and Archaea"
- [methods] vsearch --uchime_ref temp/otus.fa --db ${db}/usearch/rdp_16s_v18.fa --nonchimeras result/raw/otus.fa: "vsearch --uchime_ref temp/otus.fa --db ${db}/usearch/rdp_16s_v18.fa --nonchimeras result/raw/otus.fa"
- [readme] ./vsearch --usearch_global queries.fsa --db database.fsa --id 0.9 --alnout alnout.txt: [evidence span withheld — anchor status: external_doc]
- [abstract] integrates popular tools such as USEARCH and VSEARCH, offering a streamlined workflow from raw data to results: "integrates popular tools such as USEARCH and VSEARCH, offering a streamlined workflow from raw data to results"
