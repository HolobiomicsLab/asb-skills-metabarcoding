---
name: phylogenetic-tree-construction-from-otu-sequences
description: Use when a filtered, dereplicated, chimera-checked set of representative
  OTU sequences (e.g. result/otus.fa) must be converted into a phylogenetic tree to
  enable phylogenetically aware beta diversity (UniFrac) and PCoA ordination.
when_to_use_negative:
- Input is still raw reads or unfiltered sequences — chimera removal (uchime_ref/cluster_otus)
  must precede tree building
- The analysis plan needs only composition-based distances such as Bray-Curtis; the
  tree is unnecessary overhead
- The user requires full open-source tooling — USEARCH is free but not open-source
  (VSEARCH is its open-source alternative, though the tree-building step here is described
  with USEARCH)
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0637
tools:
- name: USEARCH
  role: Performs hierarchical average-linkage agglomerative clustering of representative
    OTU sequences to output the phylogenetic tree (cluster_agg), and consumes the
    tree in -beta_div to compute UniFrac distances
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyAmplicon
  role: Hosts the pipeline context (pipeline.sh, scripts such as beta_pcoa.R) that
    wires the tree into beta-diversity computation and PCoA ordination
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: VSEARCH
  role: Open-source alternative used earlier in the same pipeline for dereplication
    and reference-based chimera removal that prepares the otus.fa input to tree building
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:8610bf603e2112cab24ae208366a3df0c30e26163d880666114e00f3123c581b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# phylogenetic-tree-construction-from-otu-sequences

## Summary

Builds a phylogenetic tree from representative OTU sequences (result/otus.fa) with USEARCH's hierarchical average-linkage agglomerative clustering, producing result/otus.tree that is required downstream for UniFrac-style beta-diversity distance matrices in the EasyAmplicon pipeline.

## When to use

Use when a filtered, dereplicated, chimera-checked set of representative OTU sequences (e.g. result/otus.fa) must be converted into a phylogenetic tree to enable phylogenetically aware beta diversity (UniFrac) and PCoA ordination.

## When NOT to use

- Input is still raw reads or unfiltered sequences — chimera removal (uchime_ref/cluster_otus) must precede tree building
- The analysis plan needs only composition-based distances such as Bray-Curtis; the tree is unnecessary overhead
- The user requires full open-source tooling — USEARCH is free but not open-source (VSEARCH is its open-source alternative, though the tree-building step here is described with USEARCH)

## Inputs

- Representative OTU sequences FASTA (result/otus.fa) produced by OTU clustering and chimera filtering
- Rarefied OTU table (result/otutab_rare.txt) for the downstream -beta_div step that consumes the tree

## Outputs

- Phylogenetic tree in Newick format (result/otus.tree)
- Downstream inter-sample distance matrices under result/beta/ (Bray-Curtis, Euclidean, UniFrac)
- PCoA ordination plot (beta/pcoa_bray_curtis.pdf) and pairwise group comparison p-values (result/beta/beta_pcoa_stat.txt)

## How to apply

After OTU picking and chimera removal (e.g. vsearch --uchime_ref against rdp_16s_v18.fa or de novo cluster_otus), run USEARCH's cluster_agg command on the representative-sequences FASTA to perform hierarchical average-linkage clustering and write the tree: usearch -cluster_agg result/otus.fa -treeout result/otus.tree. The resulting Newick tree captures evolutionary relatedness among OTUs so that USEARCH's -beta_div step can compute UniFrac alongside Bray-Curtis and Euclidean distance matrices from the rarefied table result/otutab_rare.txt. Decision point: this tree is only needed if phylogenetic distances (UniFrac) are part of the beta-diversity plan; Bray-Curtis alone does not require it. The tree, distance matrices, and PCoA plot (beta_pcoa_bray_curtis.pdf via beta_pcoa.R) plus pairwise group p-values (beta_pcoa_stat.txt) are all written under result/beta/.

## Related tools

- **USEARCH** (Performs hierarchical average-linkage agglomerative clustering of representative OTU sequences to output the phylogenetic tree (cluster_agg), and consumes the tree in -beta_div to compute UniFrac distances) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyAmplicon** (Hosts the pipeline context (pipeline.sh, scripts such as beta_pcoa.R) that wires the tree into beta-diversity computation and PCoA ordination) — https://github.com/YongxinLiu/EasyAmplicon
- **VSEARCH** (Open-source alternative used earlier in the same pipeline for dereplication and reference-based chimera removal that prepares the otus.fa input to tree building) — https://github.com/torognes/vsearch

## Examples

```
usearch -cluster_agg result/otus.fa -treeout result/otus.tree
```

## Evaluation signals

- result/otus.tree exists and is a parseable Newick file with leaf count matching the number of OTUs in result/otus.fa
- The subsequent usearch -beta_div run on result/otutab_rare.txt with -tree result/otus.tree completes and emits UniFrac alongside Bray-Curtis/Euclidean matrices under result/beta/
- Each generated distance matrix's column count equals the number of samples in result/metadata.txt
- beta_pcoa_stat.txt contains p-values for every pairwise group comparison defined by the Group column of the metadata

## Limitations

- The tree is a hierarchical average-linkage clustering approximation of phylogeny from amplicon OTUs, not a full multiple-alignment-based phylogeny
- USEARCH is not open-source; its default heuristic seed-and-extend aligner may be less sensitive than VSEARCH's Needleman-Wunsch global alignment (per the VSEARCH README)
- Tree quality depends on upstream chimera removal; de novo dechimerization via cluster_otus is the default, reference-based (uchime_ref) is optional

## Evidence

- [methods] Build the phylogenetic tree from representative sequences with USEARCH hierarchical average-linkage clustering: `usearch -cluster_agg result/otus.fa -treeout result/otus.tree`.: [evidence span withheld — anchor status: not_found]
- [methods] Generate inter-sample distance matrices from the rarefied feature table using the tree: `usearch -beta_div result/otutab_rare.txt -tree result/otus.tree -filename_prefix result/beta/`: "usearch -beta_div result/otutab_rare.txt -tree result/otus.tree -filename_prefix result/beta/"
- [intro] USEARCH is a versatile tool for high‐throughput sequence analysis, offering capabilities such as clustering, chimera detection, and OTU picking.: "USEARCH is a versatile tool for high‐throughput sequence analysis, offering capabilities such as clustering, chimera detection, and OTU picking."
- [readme] The new tool should: have open source code with an appropriate open source license ... In contrast to USEARCH which by default uses a heuristic seed and extend aligner.: [evidence span withheld — anchor status: external_doc]
