---
name: reference-database-mapping
description: Use when long nanopore/PacBio reads need strain- or species-level taxonomic
  assignment against a large RefSeq-scale reference database, and per-read mapping
  locations, identities, and mapping qualities are required.
when_to_use_negative:
- 'Samples with a high proportion of short reads: reads below the chosen minimum length
  get minimizer sets too small for reliable mapping and remain unassigned'
- Complex samples dominated by non-characterized entities (database-sample incongruity/mismatch),
  where assignments against the reference database will miss or misplace novelty
- Analyses requiring real-time/streaming processing during sequencing, which MetaMaps
  does not support
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: MetaMaps
  role: 'Primary tool: minimizer-based approximate mapping of long reads to the reference
    database, probabilistic mapping-quality scoring, and EM-based sample composition
    estimation'
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying approximate mapping algorithm (minimizer/minhash-based local alignment
    boundaries) on which MetaMaps' mapping stage is built
  repo: https://github.com/marbl/MashMap
- name: downloadRefSeq.pl
  role: Perl script to download and assemble custom RefSeq/GenBank classification
    databases (--targetBranches archaea,bacteria,fungi)
  repo: https://github.com/DiltheyLab/MetaMaps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:8f85ebe9d469ba43d95343aa0733af8bb5950face0a373bcfb573303ff39b636
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Kraken
  - Kraken 2
  - MEGAN
  - MashMap
  - MetaMaps
  - downloadRefSeq.pl
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# reference-database-mapping

## Summary

Assign long PacBio/Oxford Nanopore metagenomic reads to a comprehensive reference database (e.g. the miniSeq+H/RefSeq database of >12,000 microbial genomes) using MetaMaps, which combines minimizer-based approximate mapping (built on MashMap) with probabilistic mapping-quality scoring and EM-based sample composition estimation.

## When to use

Use when long nanopore/PacBio reads need strain- or species-level taxonomic assignment against a large RefSeq-scale reference database, and per-read mapping locations, identities, and mapping qualities are required.

## When NOT to use

- Samples with a high proportion of short reads: reads below the chosen minimum length get minimizer sets too small for reliable mapping and remain unassigned
- Complex samples dominated by non-characterized entities (database-sample incongruity/mismatch), where assignments against the reference database will miss or misplace novelty
- Analyses requiring real-time/streaming processing during sequencing, which MetaMaps does not support

## Inputs

- Long-read metagenomic reads in FASTQ (e.g. ONT GridION/PromethION Zymo mock community reads)
- Reference classification database (e.g. miniSeq+H: miniSeq+H/DB.fa FASTA plus taxonomy subdirectory, built from RefSeq/GenBank via downloadRefSeq.pl)

## Outputs

- classification_results.EM.reads2Taxon (per-read taxon assignments; x-prefixed MetaMaps-internal taxon IDs resolvable via the database taxonomy subdirectory)
- classification_results.EM.WIMP (sample composition: Absolute, EMFrequency, PotFrequency per taxonomic entity)
- classification_results.EM (approximate mappings: coordinates, Poisson/binomial identity estimates, MinHash intersection/union, mapping quality summing to 1 per read)
- classification_results.EM.contigCoverage (1000 bp window read coverage per contig)
- classification_results.EM.lengthAndIdentitiesPerMappingUnit (read length and estimated alignment identity per read)

## How to apply

Build MetaMaps from source (./bootstrap.sh; ./configure --with-boost=/path/to/boost; make metamaps) and download or construct a classification database such as miniSeq+H (>12,000 microbial genomes plus the human reference) into the databases/ directory. Run the two-step pipeline: mapDirectly generates candidate mapping locations for each long read via minimizer-based approximate mapping (based on MashMap), then classify scores each location with a probabilistic mapping-quality model and applies the EM algorithm to estimate sample composition and disambiguate closely related genomes. Use --maxmemory (e.g. 20G on a 32G machine) to constrain RAM during mapping, and -t for multithreading (unset MALLOC_ARENA_MAX if threading is inefficient). The workflow maps reads against the full database in <16 GB of RAM on a laptop, and EM 'borrowing of information across reads' is what enables strain-level discrimination, so run classification at the 'definedGenomes' level for strain resolution.

## Related tools

- **MetaMaps** (Primary tool: minimizer-based approximate mapping of long reads to the reference database, probabilistic mapping-quality scoring, and EM-based sample composition estimation) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying approximate mapping algorithm (minimizer/minhash-based local alignment boundaries) on which MetaMaps' mapping stage is built) — https://github.com/marbl/MashMap
- **downloadRefSeq.pl** (Perl script to download and assemble custom RefSeq/GenBank classification databases (--targetBranches archaea,bacteria,fungi)) — https://github.com/DiltheyLab/MetaMaps

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results --maxmemory 20; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Mapping qualities of all mappings for each read sum to 1 (stated property of classification_results.EM output)
- Species-level read assignment accuracy exceeds 94% and sample composition r2 > 0.97 on benchmarks where sample genomes or close relatives are in the database (e.g. i100, p25)
- Strain-level recall/precision approximates published benchmarks: 94%/89% on i100/p25 simulated sets, 65%/61% on real HMP7
- reads2Taxon output contains one line per read and every 'x'-prefixed taxon ID resolves against the database's extended taxonomy directory
- WIMP file reports monotone ordering Absolute/EMFrequency/PotFrequency with PotFrequency reflecting correction for unmapped reads

## Limitations

- Reads below the minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- No explicit term for genomic divergence is integrated into the statistical models, limiting explicit detection of novel strains and species
- Sample-database mismatches are a concern: microbial reference databases comprise only a fraction of microbial genome diversity; visual inspection of coverage/identity patterns should take precedence over the limited evidenceUnknownSpecies statistics
- MetaMaps is robust against large contaminant genomes that cause false positives in k-mer-based methods, but exact time/memory can exceed --maxmemory targets since the memory limit is heuristic

## Evidence

- [abstract] capable of mapping a long-read metagenome to a comprehensive RefSeq database with >12,000 genomes in <16 GB or RAM on a laptop computer: "capable of mapping a long-read metagenome to a comprehensive RefSeq database with >12,000 genomes in <16 GB or RAM on a laptop computer"
- [intro] each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm.: "each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [readme] The 'miniSeq+H' database is a good place to start. It contains >12000 microbial genomes and the human reference genome.: [evidence span withheld — anchor status: not_found]
- [readme] The mapping qualities of all mappings for one read sum up to 1.: [evidence span withheld — anchor status: not_found]
- [discussion] Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations.: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
