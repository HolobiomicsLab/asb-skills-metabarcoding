---
name: metabarcoding-diversity-data-reproduction
description: 'Use when a source article reports species/OTU occurrence numbers only
  as Venn-diagram segments (e.g. I-source total 1074: B-I 774, N-I 669, G-I 548; P-source
  total 1006: B-P 882, N-P 553, G-P 415) and a machine-readable count table is needed.'
when_to_use_negative:
- Input is already a machine-readable OTU/ASV feature table or BIOM file — count extraction
  from figures/Venn segments would be redundant and lossy
- The study reports abundance (read counts) rather than delimited species/OTU occurrence
  counts — Venn-segment reproduction does not recover abundances
- Exact farm/sample metadata are withheld and cannot be mapped to segments — note
  this study states 'The exact location of these farms cannot be disclosed based on
  the farmer demand'
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SILVAngs
  role: Online automatic pipeline based on the SILVA rDNA database that produced the
    16S OTU/species delimitations underlying the reported occurrence counts
- name: cd-hit-est v3.1.2
  role: Clustered reads into OTUs (identity criteria 1.00 and 0.98) whose species
    counts are reproduced by this skill
- name: Scythe v0.994
  role: Upstream 3'-end adapter trimmer used in the read QC pipeline preceding OTU
    clustering (recommended to run before quality-based trimming)
  repo: http://github.com/vsbuffalo/scythe
- name: Sickle
  role: Sliding-window quality trimmer used after Scythe to remove low-quality read
    ends before downstream OTU analysis
  repo: https://github.com/najoshi/sickle
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:1df242941d93821403367d4fbaf112b497d54eb164a6aa127e04f144ab610c14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vsbuffalo/scythe
  tools:
  - SILVAngs
  - Scythe v0.994
  - Sickle
  - cd-hit-est v3.1.2
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabarcoding-diversity-data-reproduction

## Summary

Reconstructs species occurrence counts from a metabarcoding study by extracting reported Venn-diagram segment counts (inflow I-source vs pond P-source water samples across three trout farm locations B, N, G) and assembling them into a tidy occurrence-count table whose totals reconcile to the article's reported species counts.

## When to use

Use when a source article reports species/OTU occurrence numbers only as Venn-diagram segments (e.g. I-source total 1074: B-I 774, N-I 669, G-I 548; P-source total 1006: B-P 882, N-P 553, G-P 415) and a machine-readable count table is needed.

## When NOT to use

- Input is already a machine-readable OTU/ASV feature table or BIOM file — count extraction from figures/Venn segments would be redundant and lossy
- The study reports abundance (read counts) rather than delimited species/OTU occurrence counts — Venn-segment reproduction does not recover abundances
- Exact farm/sample metadata are withheld and cannot be mapped to segments — note this study states 'The exact location of these farms cannot be disclosed based on the farmer demand'

## Inputs

- Article full text (e.g. DOI 10.1038/s41598-020-80236-x) containing Venn-diagram figures/segment counts of bacterial species per water source and farm location
- Reported segment counts: I-source (B-I 774, N-I 669, G-I 548; total 1074) and P-source (B-P 882, N-P 553, G-P 415; total 1006)
- Optional: supplementary tables (e.g. supplementary table 1 with sequencing output and quality stats) for cross-checking sample identity

## Outputs

- venn_species_occurrence_counts.csv — tidy table with columns source_type (inflow/pond), location (B/N/G), and integer species count
- Reconciliation report verifying source totals equal 1074 (I-sources) and 1006 (P-sources)

## How to apply

Retrieve the reported Venn-diagram counts from the article: I-sources total 1074 species (B-I 774, N-I 669, G-I 548) and P-sources total 1006 species (B-P 882, N-P 553, G-P 415), derived from 16S rRNA metabarcoding of inflow- and pond-water samples from three semi-intensive freshwater trout farms. Compute per-location totals (B, N, G) by combining the corresponding I-source and P-source counts and correcting for overlap segments in the reported diagrams. Derive unique (single-location) species counts and reported shared-species counts for each location pair from the Venn-diagram segments. Assemble all values into a tidy occurrence-count table with columns for source type (inflow/pond), location (B, N, G), and integer species count, then write it to venn_species_occurrence_counts.csv. Validate that the reconstructed source totals reconcile to the reported 1074 (I-sources) and 1006 (P-sources) species and that per-location sums are consistent with the reported Venn-diagram segments.

## Related tools

- **SILVAngs** (Online automatic pipeline based on the SILVA rDNA database that produced the 16S OTU/species delimitations underlying the reported occurrence counts)
- **cd-hit-est v3.1.2** (Clustered reads into OTUs (identity criteria 1.00 and 0.98) whose species counts are reproduced by this skill)
- **Scythe v0.994** (Upstream 3'-end adapter trimmer used in the read QC pipeline preceding OTU clustering (recommended to run before quality-based trimming)) — http://github.com/vsbuffalo/scythe
- **Sickle** (Sliding-window quality trimmer used after Scythe to remove low-quality read ends before downstream OTU analysis) — https://github.com/najoshi/sickle

## Examples

```
python scripts/reconstruct_venn_counts.py --i-source B-I=774,N-I=669,G-I=548 --p-source B-P=882,N-P=553,G-P=415 --expected-totals I=1074,P=1006 --output venn_species_occurrence_counts.csv
```

## Evaluation signals

- Reconstructed I-source counts sum exactly to 1074 (774 + 669 + 548 minus overlap corrections)
- Reconstructed P-source counts sum exactly to 1006 (882 + 553 + 415 minus overlap corrections)
- Output CSV has exactly the columns source_type, location, and integer species count, with location restricted to {B, N, G} and source_type to {inflow, pond}
- Per-location sums (I-source + P-source with overlap correction) are internally consistent across all reported Venn segments
- Within-source, B > N > G ordering holds for both I-sources (774 > 669 > 548) and P-sources (882 > 553 > 415)

## Limitations

- Counts are transcribed from published Venn diagrams, so overlap segments must be inferred/corrected manually; reconstruction is only as accurate as the reported segments
- Exact farm locations are withheld at farmers' request, limiting geographic reproducibility of the occurrence table
- The supplementary information was found empty (no changelog/content), so supplementary table 1 with sequencing output and quality stats could not be used for cross-validation
- Counts reflect species delimitations under specific pipeline parameters (SILVAngs defaults; OTU identity criteria 1.00 and 0.98) and do not transfer to other clustering thresholds

## Evidence

- [full_text] Venn-diagram I-source and P-source totals: [evidence span withheld — anchor status: not_found]
- [discussion] OTU clustering parameters: "clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2"
- [discussion] SILVAngs pipeline: "SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline based on the SILVA rDNA database"
- [discussion] Location disclosure limitation: "The exact location of these farms cannot be disclosed based on the farmer demand."
- [readme] Scythe run before quality trimming (readme rationale): "Thus, it is recommended you run Scythe *before* quality-based trimming, as part of a read quality control pipeline."
