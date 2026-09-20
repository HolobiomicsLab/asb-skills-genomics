---
name: variation-graph-construction-and-pruning
description: Use when correcting errors in noisy long reads (PacBio CLR or ONT) from
  mixed samples such as metagenomes or polyploid genomes, where consensus-template
  correction would mask low-frequency haplotype variants as errors.
when_to_use_negative:
- Input is already error-corrected or high-accuracy (e.g. PacBio HiFi/CCS) reads —
  graph pruning targets raw noisy CLR/ONT reads
- The goal is a consensus sequence template rather than haplotype-aware correction
  — VeChat's graph approach is designed to avoid consensus-induced biases
- Computational resources are tightly constrained on very large datasets — VeChat
  experiences longer runtimes and higher peak memory usage than competing tools
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: VeChat
  role: Constructs variation graphs from alignment piles via POA and performs iterative
    frequent-itemset-based pruning with min-confidence 0.2 and min-support 0.2
  repo: https://github.com/HaploKit/vechat
- name: minimap2
  role: Computes minimizer-based all-vs-all read overlaps that seed the alignment
    piles
  repo: https://github.com/lh3/minimap2
- name: racon
  role: Modified Racon is built alongside VeChat (conda/bioconda install) for consensus/alignment
    routines
  repo: https://github.com/lbcb-sci/racon
- name: fpa
  role: Filters pairwise alignments from all-vs-all read mapping as part of the preprocessing
    pipeline
  repo: https://github.com/natir/fpa
- name: yacrd
  role: Chimeric-read detection over all-vs-all mappings, paired with the --scrub
    option
  repo: https://github.com/natir/yacrd
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:492b200d20e54db46e8421866e4872728e81083856ae9d5b98155ad7b237530d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Racon
  - VeChat
  - fpa
  - minimap2
  - racon
  - yacrd
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# variation-graph-construction-and-pruning

## Summary

Constructs ad-hoc variation graphs from raw long-read alignment piles via partial order alignment (POA) and iteratively prunes error-induced nodes and edges — the core graph-building step of VeChat's haplotype-aware error correction for PacBio/ONT reads.

## When to use

Use when correcting errors in noisy long reads (PacBio CLR or ONT) from mixed samples such as metagenomes or polyploid genomes, where consensus-template correction would mask low-frequency haplotype variants as errors.

## When NOT to use

- Input is already error-corrected or high-accuracy (e.g. PacBio HiFi/CCS) reads — graph pruning targets raw noisy CLR/ONT reads
- The goal is a consensus sequence template rather than haplotype-aware correction — VeChat's graph approach is designed to avoid consensus-induced biases
- Computational resources are tightly constrained on very large datasets — VeChat experiences longer runtimes and higher peak memory usage than competing tools

## Inputs

- Raw long-read FASTA/FASTQ file (gzip-compressed allowed, e.g. example/reads.fq.gz)
- Sequencing platform flag (--platform pb or ont, default pb)
- All-vs-all read overlaps computed by Minimap2

## Outputs

- Pruned variation graph per window with error-induced nodes/edges removed
- Corrected target subreads (optimal alignment paths), concatenated into pre-corrected full-length reads (reads.corrected.fa)

## How to apply

Compute minimizer-based all-vs-all overlaps with Minimap2, select a target read, and build a read alignment pile of all overlapping reads; divide the pile into small window segments, where the target read part is the 'target subread'. For each window, construct a variation graph with the partial order alignment (POA) algorithm, then iteratively remove nodes and edges deemed spurious using a frequent itemset model over read coverage, sequencing errors and character co-occurrence, with minimum confidence 0.2 (-d) and minimum support 0.2 (-s). Re-align after each pruning round and repeat until convergence; the optimal alignment path through the graph yields the corrected target subread. Rationale: unlike sequence-based consensus templates, graph-based reference systems capture variants of lower frequencies and do not mistakenly mask them as errors.

## Related tools

- **VeChat** (Constructs variation graphs from alignment piles via POA and performs iterative frequent-itemset-based pruning with min-confidence 0.2 and min-support 0.2) — https://github.com/HaploKit/vechat
- **minimap2** (Computes minimizer-based all-vs-all read overlaps that seed the alignment piles) — https://github.com/lh3/minimap2
- **racon** (Modified Racon is built alongside VeChat (conda/bioconda install) for consensus/alignment routines) — https://github.com/lbcb-sci/racon
- **fpa** (Filters pairwise alignments from all-vs-all read mapping as part of the preprocessing pipeline) — https://github.com/natir/fpa
- **yacrd** (Chimeric-read detection over all-vs-all mappings, paired with the --scrub option) — https://github.com/natir/yacrd

## Examples

```
vechat reads.fq.gz -t 8 --platform pb -d 0.2 -s 0.2 --min-identity-cns 0.99 -o reads.corrected.fa
```

## Evaluation signals

- Corrected reads on real PacBio datasets reach residual error rates of 0.089% (Microbial 10-plex), 0.101% (NWC metagenome) and 0.236% (yeast pseudo-diploid) with default parameters
- Pruning parameters remain at defaults: -d/--min-confidence 0.2, -s/--min-support 0.2, --min-identity-cns 0.99
- The iterative prune-and-re-align procedure converges rather than terminating at a fixed iteration count
- Output reads.corrected.fa exists and contains one corrected sequence per input read in valid FASTA
- Low-frequency true variants are retained rather than pruned: corrected reads show 4–15x fewer errors (PacBio) and 1–10x fewer errors (ONT) than state-of-the-art tools

## Limitations

- VeChat does not win in terms of computational resources: on large datasets it shows longer runtimes and higher peak memory usage than competing correction tools
- VeChat uses off-the-shelf approaches (e.g. edit-distance and sequence-to-graph alignments) without exploiting all their features, causing avoidable overheads
- Optimal combinations of preprocessing, error correction and assembly for polyploid, cancer or metagenomic settings remain to be explored

## Evidence

- [results] POA construction with iterative pruning: "the construction of a variation graph using the partial order alignment (POA) algorithm30, and pruning this graph in an iterative manner from nodes and edges that are spurious"
- [results] Frequent itemset pruning model: "we make use of a frequent itemset model that involves read coverage, sequencing errors and co-occurrence of characters in reads"
- [results] Convergence criterion: "The procedure of spurious node and edge removal followed by re-alignment is repeated until convergence"
- [methods] Default pruning thresholds: "-d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2)"
- [readme] Graph-based template rationale: "graph based reference systems also capture variants of lower frequencies, so do not mistakenly mask them as errors"
- [discussion] Memory/runtime limitation: "VeChat does not win the competition in terms of computational resources. In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage"
