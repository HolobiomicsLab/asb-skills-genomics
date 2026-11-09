---
name: variation-graph-construction-partial-order-alignment
description: 'Use when correcting noisy long reads (PacBio/ONT) read-by-read: after
  computing a read alignment pile for a target read, each window of the pile must
  be turned into a variation graph whose optimal path yields the corrected subread.'
when_to_use_negative:
- 'Cycle-2 consensus derivation: here haplotype-aware consensus sequences are derived
  from the variation graphs by a dynamic programming algorithm without re-pruning
  the graph, so the iterative POA-construct-and-prune step should not be repeated'
- If the goal is conventional consensus-template error correction rather than haplotype-aware
  correction — VeChat's rationale is that graph-based references capture variants
  of lower frequency that consensus templates mask as errors
- When a curated reference/pangenome graph already exists and alignment-to-reference
  is the intended task, rather than constructing an ad-hoc graph from raw reads
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: VeChat
  role: Host tool that orchestrates variation-graph construction from the read alignment
    pile and drives iterative pruning of error-induced nodes and edges
  repo: https://github.com/HaploKit/vechat
- name: Racon
  role: Modified Racon module bundled with VeChat supplies the partial order alignment
    (POA) routine used to construct the ad-hoc variation graph per window
  repo: https://github.com/lbcb-sci/racon
- name: Minimap2
  role: Computes minimizer-based all-vs-all overlaps (upstream of graph construction)
    that define the read alignment pile for each target read
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:00834f751f94b2ada2dfb2effaebcd5ccdf3b1b4695de06560fd55ce05d5b46c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Minimap2
  - Racon
  - VeChat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# variation-graph-construction-partial-order-alignment

## Summary

Builds an ad-hoc variation graph for a window of a long-read alignment pile using the partial order alignment (POA) algorithm from the modified Racon module bundled with VeChat, as the substrate for haplotype-aware error correction and pruning.

## When to use

Use when correcting noisy long reads (PacBio/ONT) read-by-read: after computing a read alignment pile for a target read, each window of the pile must be turned into a variation graph whose optimal path yields the corrected subread.

## When NOT to use

- Cycle-2 consensus derivation: here haplotype-aware consensus sequences are derived from the variation graphs by a dynamic programming algorithm without re-pruning the graph, so the iterative POA-construct-and-prune step should not be repeated
- If the goal is conventional consensus-template error correction rather than haplotype-aware correction — VeChat's rationale is that graph-based references capture variants of lower frequency that consensus templates mask as errors
- When a curated reference/pangenome graph already exists and alignment-to-reference is the intended task, rather than constructing an ad-hoc graph from raw reads

## Inputs

- Raw long-read file in FASTA/FASTQ format (gzip-compressed accepted; e.g. example/reads.fq.gz)
- All-vs-all overlaps computed with Minimap2 defining the read alignment pile for each target read
- Windowed segments of the read alignment pile (the target-read portion of each window is the 'target subread')

## Outputs

- Ad-hoc variation graph per window with nodes/edges weighted by read coverage and per-base Phred probabilities
- Pruned variation graph retaining only nodes/edges supported as true sequential phenomena
- Corrected target subread read off as the optimal alignment path through the converged graph

## How to apply

Take the per-window read alignment pile (produced after Minimap2-based all-vs-all overlaps) and construct an ad-hoc variation graph for that window using the partial order alignment (POA) routine from the modified Racon module bundled with VeChat. Annotate nodes and edges with weights derived from read coverage and per-base Phred probabilities, applying a frequent-itemset model over read coverage, sequencing errors, and character co-occurrence to distinguish error-induced elements from true sequential phenomena. Then iteratively prune edges below the confidence threshold (-d, default 0.2) or support threshold (-s, default 0.2) and re-align the target subread, repeating the prune-and-realign cycle until convergence (3 iterations). The optimal alignment path of the target subread through the converged graph is the corrected subread; this construction is repeated per window and corrected subreads are concatenated into a pre-corrected full-length read.

## Related tools

- **VeChat** (Host tool that orchestrates variation-graph construction from the read alignment pile and drives iterative pruning of error-induced nodes and edges) — https://github.com/HaploKit/vechat
- **Racon** (Modified Racon module bundled with VeChat supplies the partial order alignment (POA) routine used to construct the ad-hoc variation graph per window) — https://github.com/lbcb-sci/racon
- **Minimap2** (Computes minimizer-based all-vs-all overlaps (upstream of graph construction) that define the read alignment pile for each target read) — https://github.com/lh3/minimap2

## Examples

```
git clone https://github.com/HaploKit/vechat.git && vechat example/reads.fq.gz -d 0.2 -s 0.2
```

## Evaluation signals

- Prune-and-realign cycle converges within the expected number of iterations (3) for each window
- Surviving graph edges all meet the default minimum-confidence (0.2) and minimum-support (0.2) thresholds
- Converged graph retains only nodes/edges reflecting true sequential phenomena rather than sequencing-error artifacts
- Optimal alignment path of the target subread through the converged graph yields a corrected subread; concatenated target subreads reconstruct the read at full original length
- Smoke-test run succeeds on the bundled small read file example/reads.fq.gz

## Limitations

- VeChat does not win the competition in terms of computational resources; on large datasets it experiences longer runtimes and higher peak memory usage
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads (e.g. edit-distance-based alignments and sequence-to-graph alignments could be replaced by more efficient routines)
- Exploration of optimal preprocessing/error-correction/assembly pipelines for polyploid, cancer, or metagenomic settings is beyond the scope of the study and requires further efforts

## Evidence

- [results] Graph construction via POA in step 4: "the construction of a variation graph using the partial order alignment (POA) algorithm30, and pruning this graph in an iterative manner from nodes and edges that are spurious"
- [results] Frequent-itemset model underlying pruning: "we make use of a frequent itemset model that involves read coverage, sequencing errors and co-occurrence of characters in reads"
- [readme] Ad-hoc graph construction and iterative pruning from raw reads (README): "Upon initial construction of an ad-hoc variation graph from the raw input reads, nodes and edges that are due to errors are pruned from that graph"
- [methods] Default confidence threshold for keeping edges: "-d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2)"
- [methods] Test input file for the program: "One can test the program using the small sequencing read file `example/reads.fq.gz`"
