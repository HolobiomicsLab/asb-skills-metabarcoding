---
name: otu-table-singleton-filtering
description: Use when you have an initial OTU table (e.g., from VSEARCH, SWARM, CROP
  or DADA2) and want to remove single-read OTUs as a baseline noise-removal step,
  or to benchmark it against co-occurrence-based post-clustering curation such as
  LULU.
when_to_use_negative:
- 'When the goal is reliable α-diversity: the article shows singleton removal retains
  far less improvement (mean R2 gain 0.03) than LULU curation (0.27), and discarding
  the low-abundance tail drops rare but real OTUs.'
- When the input is already a curated OTU table (e.g., LULU-curated) — additional
  singleton removal is redundant since curation merges errors while preserving total
  read counts.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3305
tools:
- name: VSEARCH
  role: Produces one of the initial OTU tables that singleton filtering is applied
    to and evaluated against
- name: SWARM
  role: Produces one of the initial OTU tables that singleton filtering is applied
    to and evaluated against
  repo: https://github.com/torognes/swarm
- name: CROP
  role: Produces one of the initial OTU tables that singleton filtering is applied
    to and evaluated against
- name: DADA2
  role: Produces one of the initial OTU tables that singleton filtering is applied
    to and evaluated against
- name: LULU
  role: 'Benchmark comparator: co-occurrence-based post-clustering curation whose
    R2 improvement (0.27) is contrasted with singleton removal (0.03)'
  repo: https://github.com/tobiasgf/lulu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:6200b9d78b4361b1fac772a5cec04a8caaa6506c7a66b559ff891554df062393
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - CROP
  - DADA2
  - LULU
  - SWARM
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# otu-table-singleton-filtering

## Summary

Filters singleton OTUs (total read count of one) from an OTU table as a traditional noise-removal strategy, then evaluates whether the resulting richness better corresponds with independent ground-truth (e.g., plant survey richness). In the Frøslev et al. validation, singleton removal yielded only marginal gains (mean R2 improvement 0.03) versus LULU post-clustering curation (0.27).

## When to use

Use when you have an initial OTU table (e.g., from VSEARCH, SWARM, CROP or DADA2) and want to remove single-read OTUs as a baseline noise-removal step, or to benchmark it against co-occurrence-based post-clustering curation such as LULU.

## When NOT to use

- When the goal is reliable α-diversity: the article shows singleton removal retains far less improvement (mean R2 gain 0.03) than LULU curation (0.27), and discarding the low-abundance tail drops rare but real OTUs.
- When the input is already a curated OTU table (e.g., LULU-curated) — additional singleton removal is redundant since curation merges errors while preserving total read counts.

## Inputs

- Initial OTU table (samples as columns, OTUs as rows with unique OTU ids as row names), e.g., produced with VSEARCH, SWARM, CROP or DADA2
- Paired independent ground-truth richness data (e.g., per-site vascular plant field-survey species richness for 130 Danish sites)

## Outputs

- Singleton-filtered OTU table (OTUs with total read count of one removed)
- Per-site OTU richness values computed from the filtered table
- R2 correspondence values (OTU richness vs. survey plant richness) and R2 improvement over the uncurated baseline, tabulated for comparison against LULU curation (e.g., comparison_table.csv)

## How to apply

Load the initial OTU table (samples as columns, unique OTU ids as rows) paired with per-site richness data from an independent survey (here, vascular plant field surveys at 130 Danish sites). Filter out every OTU whose total read count across all samples equals one, then recompute per-sample OTU richness from the filtered table. Regress per-site OTU richness against per-site survey species richness to obtain R2, and compute the R2 improvement over the uncurated baseline. Compare this improvement against the same metric obtained from LULU curation (run with minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95) to judge whether simple singleton filtering suffices; in the source study it recovered only ~0.03 mean R2 improvement versus ~0.27 for LULU.

## Related tools

- **VSEARCH** (Produces one of the initial OTU tables that singleton filtering is applied to and evaluated against)
- **SWARM** (Produces one of the initial OTU tables that singleton filtering is applied to and evaluated against) — https://github.com/torognes/swarm
- **CROP** (Produces one of the initial OTU tables that singleton filtering is applied to and evaluated against)
- **DADA2** (Produces one of the initial OTU tables that singleton filtering is applied to and evaluated against)
- **LULU** (Benchmark comparator: co-occurrence-based post-clustering curation whose R2 improvement (0.27) is contrasted with singleton removal (0.03)) — https://github.com/tobiasgf/lulu

## Examples

```
Rscript -e 'ot <- read.table("otutable.txt",header=TRUE,row.names=1); filt <- ot[rowSums(ot)>1,]; write.table(filt,"singleton_removed.txt",sep="\t",quote=FALSE)'
```

## Evaluation signals

- Verify no OTU in the filtered table has a total read count across samples equal to one.
- Verify per-site OTU richness is recomputed on the same 130 sites/samples as the uncurated baseline.
- Check that the mean R2 improvement over the uncurated tables for singleton removal is approximately 0.03 (vs ~0.27 for LULU curation).
- Confirm the comparison table spans all clustering methods (VSEARCH, SWARM, CROP, DADA2) so singleton-removal vs LULU gains are directly comparable.
- Ensure regression of filtered OTU richness against field-survey richness uses paired site-level data (plant richness 6–93 per site).

## Limitations

- Singleton removal offers only marginal gains in OTU-richness vs plant-richness correspondence compared to LULU curation (0.03 vs 0.27 mean R2 improvement).
- Discarding the tail of the OTU rank-abundance distribution likely retains only dominant species, which is much less adequate for α-diversity than composition/turnover.
- LULU curation (and by comparison, this filtering) carries a small 'cost of curation' — a loss of real (rare, low-abundance) species — and may not be suitable for identifying single rare species.

## Evidence

- [intro] Furthermore, we evaluated the effect of singleton removal in comparison to curation: "Furthermore, we evaluated the effect of singleton removal in comparison to curation"
- [intro] current approaches retain only dominant species: "current approaches are likely to retain only dominant species, which may be adequate for assessing composition and turnover, but much less so for α-diversity"
- [results] Singleton removal vs LULU R2 improvement finding: [evidence span withheld — anchor status: not_found]
- [discussion] LULU merge preserves read counts: "After identification of these extra OTUs, they can be merged with their parent OTUs in order to preserve the total read count and reduce the OTU number of the table"
- [readme] README OTU table requirements: "The requirements of LULU are that the OTU table has samples as columns and OTUs as rows, and that it has unique OTU id's as row names."
