---
name: database-sample-mismatch-detection
description: Use when analyzing long-read (PacBio/ONT) metagenomic samples against
  RefSeq and you suspect out-of-database contaminants (e.g. eukaryotic genomes) or
  database-sample mismatches detectable via identity and coverage patterns.
when_to_use_negative:
- Samples dominated by short reads, which fall below the minimum length needed for
  reliable minimizer-based mapping and remain unassigned
- Complex samples comprising many non-characterized entities where no single contaminant
  or mismatch signal dominates
- Analyses requiring real-time/streaming classification during sequencing, which MetaMaps
  does not support
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Maps long reads to a comprehensive RefSeq database and produces per-read mapping
    locations, estimated alignment identities, mapping qualities, and contig coverage
    used to flag out-of-database reads and database-sample mismatches
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Approximate mapping algorithm underlying MetaMaps' minimizer-based mapping
    stage
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:0528fd77f8fbfb165d38e5edc27a9704f7bf937e738f4392877b65b6b2d3859d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# database-sample-mismatch-detection

## Summary

Detect incongruities between a metagenomic sample and the reference database using MetaMaps' per-read mapping output: out-of-database large genomes remain unclassified with high precision, while in-database but mismatched samples reveal themselves via shifted estimated-alignment-identity distributions and uneven read-depth coverage along reference contigs.

## When to use

Use when analyzing long-read (PacBio/ONT) metagenomic samples against RefSeq and you suspect out-of-database contaminants (e.g. eukaryotic genomes) or database-sample mismatches detectable via identity and coverage patterns.

## When NOT to use

- Samples dominated by short reads, which fall below the minimum length needed for reliable minimizer-based mapping and remain unassigned
- Complex samples comprising many non-characterized entities where no single contaminant or mismatch signal dominates
- Analyses requiring real-time/streaming classification during sequencing, which MetaMaps does not support

## Inputs

- Long-read (PacBio/ONT) metagenomic FASTQ files, e.g. simulated e2 eukaryotic read sets or HMP7 Actinomyces reads
- MetaMaps classification database (e.g. miniSeq+H / comprehensive RefSeq with >12,000 genomes) in databases/ directory

## Outputs

- classification_results.EM.contigCoverage: per-1000bp-window read coverage along reference contigs
- classification_results.EM.lengthAndIdentitiesPerMappingUnit: per-read estimated alignment identity and read length
- Precision/recall statistics of leaving out-of-database reads unclassified (>99% at species, genus, family levels)
- Coverage and identity-distribution plots evidencing database-sample mismatch (Supplementary Data 4)

## How to apply

Build MetaMaps from source (./bootstrap.sh; ./configure --with-boost=/path/to/boost; make metamaps) and map long reads against a comprehensive RefSeq database (e.g. miniSeq+H with >12,000 genomes) using 'metamaps mapDirectly' followed by 'metamaps classify'. For out-of-database detection, run on samples containing large contaminant genomes (e.g. simulated yellow fever mosquito or Toxoplasma gondii reads in experiment e2) and compute precision/recall of leaving those reads unclassified, expecting both >99%, whereas k-mer/alignment tools misclassify up to ~45% of such reads. For in-database mismatch detection, extract per-read mapping locations and estimated alignment identities from classification_results.EM.lengthAndIdentitiesPerMappingUnit and classification_results.EM.contigCoverage; visualize read-depth coverage along the reference (e.g. Actinomyces meyeri in HMP7) to expose uneven spatial coverage, and plot the identity distribution to reveal a shift away from the ~0.88 identity mode. On CAMI, summarize identity statistics to confirm the ~82% median-identity mismatch signal. Use visual analysis or automatic thresholding on these coverage/identity patterns to decide the sample contains entities poorly represented in the database.

## Related tools

- **MetaMaps** (Maps long reads to a comprehensive RefSeq database and produces per-read mapping locations, estimated alignment identities, mapping qualities, and contig coverage used to flag out-of-database reads and database-sample mismatches) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Approximate mapping algorithm underlying MetaMaps' minimizer-based mapping stage) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results && ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Precision and recall of leaving out-of-database eukaryotic reads (yellow fever mosquito, Toxoplasma gondii) unclassified both exceed 99% at species, genus, and family levels
- Coverage plot along the Actinomyces meyeri reference shows uneven spatial read-depth coverage
- Alignment-identity distribution shows a shift away from the ~0.88 mode for mismatched in-database samples (reproducing Supplementary Data 4)
- CAMI sample identity statistics yield a ~82% median-identity mismatch signal
- Mapping qualities of all mappings for one read sum to 1 in classification_results.EM output

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Analysis of samples with a high proportion of short reads is not well addressed
- Complex samples comprising many non-characterized entities remain challenging
- No explicit term for genomic divergence is integrated into the statistical models, limiting explicit detection of novel strains and species
- The classification_results.EM.evidenceUnknownSpecies statistics file 'are not particularly useful in their current state; visual inspection of coverage and identity patterns should take precedence'

## Evidence

- [abstract] MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence) and detection of incongruities between sample and reference genomes: "MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence) and detection of incongruities between sample and reference genomes"
- [methods] bootstrap and configure build commands: "./bootstrap.sh
   ./configure --with-boost=/path/to/boost [--prefix=...] 
   make metamaps"
- [discussion] Reads below minimum length have minimizer sets too small for reliable mapping: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
- [discussion] HMP7 and CAMI visual/threshold analysis of mismatch: "As we have shown in the HMP7 and CAMI experiments, visual analysis or automatic thresholding can contribute to detecting and dealing with database-sample mismatches."
- [readme] README usage of mapDirectly and classify: [evidence span withheld — anchor status: not_found]
- [readme] README output description of contig coverage file: [evidence span withheld — anchor status: not_found]
