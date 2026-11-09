---
name: long-read-error-correction
description: Use when raw PacBio CLR or ONT long reads (FASTA/FASTQ, gzip allowed)
  from diploid, polyploid, or metagenomic mixed samples need error correction that
  preserves low-frequency haplotype variants prior to assembly or QV benchmarking.
when_to_use_negative:
- The task is short-read error correction or polishing of an existing assembly with
  Illumina data — VeChat targets raw third-generation long reads.
- 'Runtime/peak memory are the primary constraints: on large datasets VeChat has longer
  runtimes and higher peak memory usage than competing correctors.'
- Reads are already error-corrected or the goal is consensus polishing of contigs
  rather than haplotype-aware correction of raw reads.
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: VeChat
  role: 'Primary correction tool: two-cycle variation-graph construction, frequent-itemset
    graph pruning, and haplotype-aware consensus derivation'
  repo: https://github.com/HaploKit/vechat
- name: minimap2
  role: Computes minimizer-based all-vs-all read overlaps (cycle 1) and overlaps with
    base-level alignments for haplotype-aware filtration (cycle 2)
  repo: https://github.com/lh3/minimap2
- name: Racon
  role: Modified Racon module bundled with VeChat provides the POA routine used to
    construct ad-hoc variation graphs per window
  repo: https://github.com/lbcb-sci/racon
- name: fpa
  role: Dependency used to filter pairwise all-against-all read alignments
  repo: https://github.com/natir/fpa
- name: yacrd
  role: Dependency for chimera detection on long reads
  repo: https://github.com/natir/yacrd
- name: QUAST
  role: Alignment-based computation of residual substitution and indel error rates
    of corrected reads against reference ground truth
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_007
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:8c1a25658fafd0ef826efeb0eaa2759ddefce5a1d8d0684b61b483489dd0517f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - CONSENT
  - Canu
  - Daccord
  - Merqury
  - Minimap2
  - QUAST
  - Racon
  - VeChat
  - fpa
  - minimap2
  - yacrd
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# long-read-error-correction

## Summary

Haplotype-aware error correction of noisy long reads (PacBio CLR, ONT) using variation graphs via VeChat, which prunes error-induced nodes/edges from POA-constructed graphs using frequent-itemset-based Support/Confidence thresholds instead of consensus-sequence templates that mask low-frequency haplotype variants.

## When to use

Use when raw PacBio CLR or ONT long reads (FASTA/FASTQ, gzip allowed) from diploid, polyploid, or metagenomic mixed samples need error correction that preserves low-frequency haplotype variants prior to assembly or QV benchmarking.

## When NOT to use

- The task is short-read error correction or polishing of an existing assembly with Illumina data — VeChat targets raw third-generation long reads.
- Runtime/peak memory are the primary constraints: on large datasets VeChat has longer runtimes and higher peak memory usage than competing correctors.
- Reads are already error-corrected or the goal is consensus polishing of contigs rather than haplotype-aware correction of raw reads.

## Inputs

- Raw long-read file in FASTA or FASTQ format (gzip-compressed accepted), e.g. PacBio CLR or ONT reads (example/reads.fq.gz)
- Sequencing platform designation (pb or ont) via --platform
- Optional reference haplotypes for downstream QUAST-based error-rate validation

## Outputs

- reads.corrected.fa — final corrected reads in FASTA, one corrected sequence per input read
- Residual error rates / QV per dataset when corrected reads are aligned back to references
- error_rate_table.tsv or pacbio_error_rates.tsv when benchmarking against baselines

## How to apply

Run the two-cycle VeChat workflow: in cycle 1, compute minimizer-based all-vs-all overlaps with Minimap2, build read alignment piles per target read, split piles into windows, construct variation graphs via partial order alignment (POA) in the bundled modified Racon, and iteratively prune error-induced edges falling below minimum confidence (-d, default 0.2) or minimum support (-s, default 0.2), repeating prune-and-realign until convergence (3 iterations) before concatenating corrected target subreads into full-length pre-corrected reads. In cycle 2, recompute all-vs-all overlaps with base-level alignments enabling haplotype-aware read overlap filtration (minimum overlap identity 0.99 via --min-identity-cns) and derive haplotype-aware consensus sequences from the variation graphs with dynamic programming, without re-pruning. Set --platform to pb or ont to match the input chemistry, use --split for FASTQ > 20G or FASTA > 10G to reduce memory, and optionally --scrub to remove chimeric reads. Judge success by residual error rates on benchmark datasets (e.g. 0.236% yeast pseudo-diploid, 0.101% NWC metagenome, 0.089% Microbial 10-plex on PacBio) and by the presence of a valid FASTA with one corrected sequence per input read.

## Related tools

- **VeChat** (Primary correction tool: two-cycle variation-graph construction, frequent-itemset graph pruning, and haplotype-aware consensus derivation) — https://github.com/HaploKit/vechat
- **minimap2** (Computes minimizer-based all-vs-all read overlaps (cycle 1) and overlaps with base-level alignments for haplotype-aware filtration (cycle 2)) — https://github.com/lh3/minimap2
- **Racon** (Modified Racon module bundled with VeChat provides the POA routine used to construct ad-hoc variation graphs per window) — https://github.com/lbcb-sci/racon
- **fpa** (Dependency used to filter pairwise all-against-all read alignments) — https://github.com/natir/fpa
- **yacrd** (Dependency for chimera detection on long reads) — https://github.com/natir/yacrd
- **QUAST** (Alignment-based computation of residual substitution and indel error rates of corrected reads against reference ground truth)

## Examples

```
vechat reads.fq.gz -t 8 --platform pb -o reads.corrected.fa
```

## Evaluation signals

- reads.corrected.fa exists for each dataset and is a valid FASTA containing one corrected sequence per input read
- Pruning converges under default thresholds (confidence 0.2, support 0.2) within the 3-iteration cycle, retaining only nodes/edges reflecting true sequential phenomena
- Reproduced error rates match reported values: 0.236% (yeast pseudo-diploid), 0.101% (NWC metagenome), 0.089% (Microbial 10-plex) on PacBio datasets
- Simulated-diploid benchmark shows VeChat ≈ 0.014% vs Racon 0.276% on PacBio, within the reported 14–30x improvement range over baselines
- Comparison table rows for each dataset/corrector contain finite positive error-rate metrics (Merqury QV and QUAST alignment-based rate)

## Limitations

- VeChat does not win in computational resources: on large datasets it experiences longer runtimes and higher peak memory usage.
- Off-the-shelf components are used without exploiting all their features, causing unnecessary overheads; edit-distance-based and sequence-to-graph alignments could be replaced by more efficient routines.
- Earlier haplotype-aware approaches only handle diploid scenarios; polyploid/cancer/metagenomic pipeline combinations remain under-explored and require further efforts.

## Evidence

- [readme] VeChat distinguishes errors from haplotype-specific true variants based on variation graphs: "VeChat distinguishes errors from haplotype-specific true variants based on variation graphs, which reflect a popular type of data structure for pangenome reference systems."
- [results] the construction of a variation graph using the partial order alignment (POA) algorithm: "the construction of a variation graph using the partial order alignment (POA) algorithm30, and pruning this graph in an iterative manner from nodes and edges that are spurious"
- [methods] minimum confidence for keeping edges in the graph (default: 0.2): "-d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2)"
- [results] base-level alignments are computed, which enables haplotype aware read overlap filtration: "base-level alignments are computed, which enables haplotype aware read overlap filtration."
- [abstract] long reads corrected by VeChat contain 4 to 15 (Pacific Biosciences) and 1 to 10 times (Oxford Nanopore Technologies) less errors: "long reads corrected by VeChat contain 4 to 15 (Pacific Biosciences) and 1 to 10 times (Oxford Nanopore Technologies) less errors than when being corrected by state of the art approaches"
- [discussion] VeChat does not win the competition in terms of computational resources: "VeChat does not win the competition in terms of computational resources. In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage"
