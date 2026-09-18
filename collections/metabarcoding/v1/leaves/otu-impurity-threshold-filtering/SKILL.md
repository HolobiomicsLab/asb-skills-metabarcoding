---
name: otu-impurity-threshold-filtering
description: Use when profiling intragenomic 16S SNPs across 99%-identity full-length
  16S OTUs from PacBio CCS isolate data and needing to drop OTUs whose member isolates
  show >3% impurity at SNP positions before counting unique SNP profiles.
when_to_use_negative:
- Input is short-read 16S variable-region data (e.g., V4), which the paper shows cannot
  resolve intragenomic 16S copy variants at all
- The target variation is insertions/deletions — this filter deliberately ignores
  indels because PacBio CCS deletion errors are systematic
- The problem is within-isolate sequencing-error removal (9-IQR outlier flagging),
  not cross-isolate OTU homogeneity assessment
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: Cross_match
  role: Aligns each isolate's PacBio CCS reads to its most-abundant full-length 16S
    sequence, producing the alignments from which per-position substitution frequencies
    (and hence OTU impurity) are computed
- name: PacBio
  role: Sequencing platform whose systematic CCS deletion errors (coincident with
    homopolymer runs) motivate assessing impurity on substitutions only
- name: ruffus
  role: CGAT ruffus framework running the Figure 4 pipeline (pipeline_pb_strains,
    Python 3.6) that implements the OTU impurity filtering
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:be5cc11a51ccbd83ac8b052fec89214003129466772965449c4c9e9a638f8afd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cross_match
  - PacBio
  - ruffus
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# otu-impurity-threshold-filtering

## Summary

An OTU-level quality filter for intragenomic 16S SNP profiling: full-length 16S sequences clustered at 99% identity are discarded as OTUs when their SNP positions show greater than 3% impurity across member isolates, so that mixed taxa or systematic sequencing error do not inflate SNP-profile counts.

## When to use

Use when profiling intragenomic 16S SNPs across 99%-identity full-length 16S OTUs from PacBio CCS isolate data and needing to drop OTUs whose member isolates show >3% impurity at SNP positions before counting unique SNP profiles.

## When NOT to use

- Input is short-read 16S variable-region data (e.g., V4), which the paper shows cannot resolve intragenomic 16S copy variants at all
- The target variation is insertions/deletions — this filter deliberately ignores indels because PacBio CCS deletion errors are systematic
- The problem is within-isolate sequencing-error removal (9-IQR outlier flagging), not cross-isolate OTU homogeneity assessment

## Inputs

- Cross_match alignments of per-isolate PacBio CCS reads against each isolate's most-abundant full-length 16S sequence (isolates with >=200 aligned reads)
- Per-position substitution frequency tables for each isolate
- Full-length 16S OTU assignments clustered at 99% identity

## Outputs

- Retained set of OTUs with <=3% SNP-position impurity across member isolates
- List of OTUs discarded for exceeding the 3% impurity threshold
- Per-isolate/per-OTU SNP-call table used to count the 205 unique SNP profiles

## How to apply

Cluster the most-abundant full-length 16S sequences of the 381 cultured gut isolates into OTUs at 99% identity; upstream, align each isolate's PacBio CCS reads to its most-abundant sequence with Cross_match and keep only isolates with at least 200 aligned reads. From those alignments compute per-position substitution frequencies, counting substitutions only and ignoring insertions/deletions because PacBio CCS deletion errors are systematic and coincide with homopolymer runs. Remove SNP positions flagged as 9-IQR outliers (systematic sequencing-error artifacts), then assess each OTU's impurity, i.e. the discordance of SNP calls across its member isolates. Discard any OTU whose SNP positions show greater than 3% impurity across member isolates, since high impurity indicates the OTU conflates distinct taxa or residual error rather than genuine intragenomic polymorphism. Tabulate surviving SNP calls per isolate and OTU; the expected end state reproduces 58 putative species, 61 OTUs, 349/381 isolates (54/61 OTUs) with at least one SNP, and 205 unique SNP profiles (Fig. 4a).

## Related tools

- **Cross_match** (Aligns each isolate's PacBio CCS reads to its most-abundant full-length 16S sequence, producing the alignments from which per-position substitution frequencies (and hence OTU impurity) are computed)
- **PacBio** (Sequencing platform whose systematic CCS deletion errors (coincident with homopolymer runs) motivate assessing impurity on substitutions only)
- **ruffus** (CGAT ruffus framework running the Figure 4 pipeline (pipeline_pb_strains, Python 3.6) that implements the OTU impurity filtering) — https://github.com/cgat-developers/ruffus

## Examples

```
python pipeline_pb_strains.py
```

## Evaluation signals

- Recomputed OTU count after impurity filtering equals 61 OTUs (58 putative species), matching Fig. 4a
- 349 of 381 isolates (54 of 61 OTUs) retain at least one SNP call after the filter
- 205 unique SNP profiles are recovered when the 3% impurity filter is combined with 9-IQR sequencing-error removal
- Every retained OTU shows <=3% impurity at SNP positions across member isolates; every discarded OTU exceeds 3%
- Only isolates passing the >=200 Cross_match-aligned-read coverage filter contribute to impurity calculations

## Limitations

- The substitution-only approach is imperfect: a genuine single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes would not be captured
- Systematic PacBio CCS deletion errors coincident with homopolymer runs occur at a given frequency and may not be improved by greater sequencing effort, leaving residual false impurity
- The 99% identity threshold used to define the OTUs being filtered was not validated as separating intragenomic vs. inter-genomic variation for all bacterial species

## Evidence

- [other] Discard OTUs (clustered at 99% identity) whose SNP positions show greater than 3% impurity across member isolates.: [evidence span withheld — anchor status: not_found]
- [discussion] full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
- [discussion] we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [abstract] full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach.: "The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach."
- [readme] The code underlying Figure 4 is written in python 3.6 (pipeline_pb_strains) or 2.7 (pipeline_blast_16s).: "The code underlying Figure 4 is written in python 3.6 (pipeline_pb_strains) or 2.7 (pipeline_blast_16s)."
