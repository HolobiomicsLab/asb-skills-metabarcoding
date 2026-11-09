---
name: vsearch-otu-clustering
description: Use when paired-end amplicon reads (e.g., Illumina .fastq.gz) have been
  pre-processed and dereplicated and the study objective calls for OTU clustering
  rather than ASV inference in a metabarcoding workflow.
when_to_use_negative:
- The study objective requires exact sequence variants (ASVs) rather than OTUs — use
  Swarm v2 inference instead (e.g., 12S rRNA and COI in PEMA support ASVs but not
  VSEARCH OTUs).
- Input is raw, unassembled reads — VSEARCH clustering must follow PANDAseq assembly,
  obiuniq dereplication, and chimera removal, not operate directly on raw .fastq files.
- Input is not amplicon/metabarcoding data — this skill is scoped to 16S/18S rRNA,
  ITS, and COI marker gene analysis.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: VSEARCH
  role: Performs chimera removal prior to clustering and clusters dereplicated reads
    into (M)OTUs as the OTU-based condition of the pipeline
  repo: https://github.com/hariszaf/pema
- name: PEMA
  role: Containerized pipeline that invokes VSEARCH within its QC/pre-processing,
    clustering, and taxonomy assignment stages
  repo: https://github.com/hariszaf/pema
- name: PANDAseq
  role: Assembles the overlapping paired-end reads that are then dereplicated and
    passed to VSEARCH
- name: OBITools
  role: obiuniq groups all identical sequences in every sample, keeping track of their
    abundances, before VSEARCH clustering
- name: Swarm v2
  role: ASV-inference alternative run in parallel with the VSEARCH OTU condition for
    comparison
- name: CREST LCAClassifier
  role: Assigns taxonomy to the VSEARCH-derived OTUs against the Silva database
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:0fc70d976aa6f3c9edffc8f373a04cf1215b6270d2b36fa8057d9ce73b2097da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - FastQC
  - OBITools
  - PANDAseq
  - PEMA
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# vsearch-otu-clustering

## Summary

Cluster dereplicated, chimera-free amplicon reads into (M)OTUs with VSEARCH as an alternative to Swarm-based ASV inference within the PEMA metabarcoding pipeline, producing an OTU table with taxonomies for 16S, 18S, ITS, and COI data.

## When to use

Use when paired-end amplicon reads (e.g., Illumina .fastq.gz) have been pre-processed and dereplicated and the study objective calls for OTU clustering rather than ASV inference in a metabarcoding workflow.

## When NOT to use

- The study objective requires exact sequence variants (ASVs) rather than OTUs — use Swarm v2 inference instead (e.g., 12S rRNA and COI in PEMA support ASVs but not VSEARCH OTUs).
- Input is raw, unassembled reads — VSEARCH clustering must follow PANDAseq assembly, obiuniq dereplication, and chimera removal, not operate directly on raw .fastq files.
- Input is not amplicon/metabarcoding data — this skill is scoped to 16S/18S rRNA, ITS, and COI marker gene analysis.

## Inputs

- Dereplicated paired-end amplicon sequences (.fastq.gz from Illumina, assembled by PANDAseq and dereplicated with obiuniq)
- parameters.tsv specifying OTU clustering (VSEARCH) condition for the chosen marker gene (16S/18S/ITS)
- Reference database for downstream taxonomy assignment (Silva 128/132, Unite, or MIDORI)

## Outputs

- OTU table with taxa abundances per sample
- Chimera-filtered sequence set (VSEARCH chimera removal output)
- Singleton-filtered OTU table (optional)
- Taxonomy assignments per OTU (via CREST LCAClassifier / Silva)

## How to apply

In PEMA, VSEARCH is invoked twice: first for chimera removal on the PANDAseq-assembled, obiuniq-dereplicated sequences, and then for clustering of reads into (M)OTUs as the OTU-based alternative to Swarm v2 ASV inference. After clustering, any singletons (sequences with only 1 read) may be removed as a filter step. Taxonomy is then assigned to the OTUs with the CREST LCAClassifier against the Silva database (versions 128 or 132 for 16S/18S), Unite (ITS), or MIDORI (COI). The choice between OTUs (VSEARCH) and ASVs (Swarm) should be based on each study's objective(s); PEMA's checkpoint support lets users run both conditions without complete re-execution. In the 18S mock community benchmark, the VSEARCH OTU condition was run in parallel with Swarm (d = 1) under two strictness settings, and genus-level precision, recall, and F1 (F1 range 74%–86%) were computed against the 12 known algal members.

## Related tools

- **VSEARCH** (Performs chimera removal prior to clustering and clusters dereplicated reads into (M)OTUs as the OTU-based condition of the pipeline) — https://github.com/hariszaf/pema
- **PEMA** (Containerized pipeline that invokes VSEARCH within its QC/pre-processing, clustering, and taxonomy assignment stages) — https://github.com/hariszaf/pema
- **PANDAseq** (Assembles the overlapping paired-end reads that are then dereplicated and passed to VSEARCH)
- **OBITools** (obiuniq groups all identical sequences in every sample, keeping track of their abundances, before VSEARCH clustering)
- **Swarm v2** (ASV-inference alternative run in parallel with the VSEARCH OTU condition for comparison)
- **CREST LCAClassifier** (Assigns taxonomy to the VSEARCH-derived OTUs against the Silva database)

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds  # with parameters.tsv set to the VSEARCH OTU clustering condition
```

## Evaluation signals

- Chimera removal ran before clustering — VSEARCH is invoked for chimera removal prior to the (M)OTU clustering step
- Singleton filter applied — sequences with only 1 read occurring after OTU clustering may be removed
- Taxonomy assigned to every OTU — CREST LCAClassifier output against Silva (128/132), Unite, or MIDORI is present
- Benchmark consistency — mock community genus-level F1-scores fall in the reported 74%–86% range
- Parameter file version match — parameters.tsv version corresponds to the PEMA version used (e.g., v.2.1.4)

## Limitations

- VSEARCH OTU clustering is not supported for the COI and 12S rRNA marker genes in PEMA (COI and 12S support ASV inference with Swarm only).
- Whether to use ASVs instead of OTUs should be based on each study's objective(s); the article advises against a blanket choice.
- For COI, a minimum of 20 GB of RAM is required at the taxonomy assignment step, which constrains where the full OTU-to-taxonomy workflow can run.
- Software version numbers for PEMA components are not stated in the article's discussion section, complicating exact reproduction.

## Evidence

- [intro] Two clustering algorithms, VSEARCH [17] and CROP [18], are used for the clustering of reads in (M)OTUs: "Two clustering algorithms, VSEARCH [17] and CROP [18], are used for the clustering of reads in (M)OTUs"
- [methods] The VSEARCH package [17] is then invoked for chimera removal: "The VSEARCH package [17] is then invoked for chimera removal"
- [methods] the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances: "the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances"
- [readme] PEMA supports both OTU clustering (VSEARCH) and ASV inference (Swarm).: [evidence span withheld — anchor status: not_found]
- [methods] Overall the F1-scores ranged from 74% to 86%: "Overall the F1-scores ranged from 74% to 86%"
