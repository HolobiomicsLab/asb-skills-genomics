---
name: polyploid-haplotype-variant-recovery
description: Use when long reads (PacBio CLR or ONT) come from mixed samples — triploid/tetraploid
  genomes, metagenomes, or virus mixtures — and consensus-template correction would
  mask low-frequency haplotype variants.
when_to_use_negative:
- Reads are from a single-haploid/clonal sample where a simple consensus template
  suffices — the graph-based haplotype machinery is unnecessary overhead (VeChat has
  longer runtimes and higher peak memory on large datasets).
- Input is already-corrected or high-accuracy data (e.g., PacBio HiFi/CCS); VeChat
  is a self-correction method for noisy raw TGS long reads.
- The goal is only assembly polishing of contigs rather than recovering haplotype
  variants in a mixed sample (e.g., Racon-style consensus polishing is the wrong-and-right
  tool mismatch).
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: VeChat
  role: 'Core tool: builds variation graphs from raw long reads, prunes error-derived
    nodes/edges, and outputs haplotype-aware corrected reads'
  repo: https://github.com/HaploKit/vechat
- name: PBSIM2
  role: Simulates PacBio CLR reads from the individual haplotype reference genomes
    at chosen per-haplotype coverage (10x/50x)
- name: minimap2
  role: Aligns raw and corrected reads back to the three reference strains to quantify
    residual sequencing error rates
  repo: https://github.com/lh3/minimap2
- name: racon
  role: State-of-the-art comparison corrector for benchmarking VeChat's error-rate
    reduction on polyploid read sets
  repo: https://github.com/lbcb-sci/racon
- name: yacrd
  role: Chimeric-read detection on all-vs-all read mappings as read-quality control
    in the evaluation pipeline
  repo: https://github.com/natir/yacrd
- name: fpa
  role: Filters all-against-all read overlap outputs (internal matches, containments,
    dovetails) before downstream evaluation
  repo: https://github.com/natir/fpa
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:29e2e8984f0a520852e321126dc82ef5e0ad2e88672ac63789e1f9d59596a241
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - PBSIM2
  - VeChat
  - fpa
  - minimap2
  - racon
  - yacrd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# polyploid-haplotype-variant-recovery

## Summary

Haplotype-aware error correction of long reads from polyploid (>diploid) samples using variation graphs (VeChat), so that true variants of lower-frequency haplotypes are not mistakenly masked as sequencing errors by consensus-induced biases.

## When to use

Use when long reads (PacBio CLR or ONT) come from mixed samples — triploid/tetraploid genomes, metagenomes, or virus mixtures — and consensus-template correction would mask low-frequency haplotype variants.

## When NOT to use

- Reads are from a single-haploid/clonal sample where a simple consensus template suffices — the graph-based haplotype machinery is unnecessary overhead (VeChat has longer runtimes and higher peak memory on large datasets).
- Input is already-corrected or high-accuracy data (e.g., PacBio HiFi/CCS); VeChat is a self-correction method for noisy raw TGS long reads.
- The goal is only assembly polishing of contigs rather than recovering haplotype variants in a mixed sample (e.g., Racon-style consensus polishing is the wrong-and-right tool mismatch).

## Inputs

- Raw noisy long reads from a polyploid/mixed sample in FASTQ/FASTQ.gz (e.g., PacBio CLR reads simulated with PBSIM2 from three E. coli strains)
- Reference genomes of the constituent haplotypes/strains (e.g., the three E. coli strain FASTA files) for aligning raw and corrected reads

## Outputs

- Haplotype-aware corrected full-length reads in FASTA (vechat -o output)
- Raw-vs-corrected read alignments to each haplotype reference (minimap2 SAM/PAF)
- Per-read-set residual sequencing error rates (and QV) allowing comparison across coverage levels and ploidy

## How to apply

Simulate or obtain noisy long reads from the polyploid sample (e.g., PBSIM2 PacBio CLR reads from three E. coli strains at 10x or 50x coverage per haplotype, in FASTQ). Run VeChat, which builds an ad-hoc variation graph from the raw reads via partial order alignment, iteratively prunes error-derived nodes/edges with a frequent itemset model, and derives haplotype-aware consensus sequences by dynamic programming — keep the default graph-pruning parameters (-d 0.2 min-confidence, -s 0.2 min-support) so low-frequency haplotype variants are retained rather than pruned as errors. Then align raw and corrected reads back to the constituent reference strains with minimap2 (e.g., preset map-pb for PacBio CLR) and quantify residual sequencing error rates per read set. Correct application is confirmed when error rates drop substantially with coverage (e.g., 0.311% at 10x to 0.017% at 50x on triploid PacBio CLR) and corrected reads are several-fold (9–26x on triploid genomes) less error-prone than with consensus-based state-of-the-art correctors.

## Related tools

- **VeChat** (Core tool: builds variation graphs from raw long reads, prunes error-derived nodes/edges, and outputs haplotype-aware corrected reads) — https://github.com/HaploKit/vechat
- **PBSIM2** (Simulates PacBio CLR reads from the individual haplotype reference genomes at chosen per-haplotype coverage (10x/50x))
- **minimap2** (Aligns raw and corrected reads back to the three reference strains to quantify residual sequencing error rates) — https://github.com/lh3/minimap2
- **racon** (State-of-the-art comparison corrector for benchmarking VeChat's error-rate reduction on polyploid read sets) — https://github.com/lbcb-sci/racon
- **yacrd** (Chimeric-read detection on all-vs-all read mappings as read-quality control in the evaluation pipeline) — https://github.com/natir/yacrd
- **fpa** (Filters all-against-all read overlap outputs (internal matches, containments, dovetails) before downstream evaluation) — https://github.com/natir/fpa

## Examples

```
vechat reads.fq.gz --platform pb -o reads.corrected.fa
```

## Evaluation signals

- Residual corrected-read error rate on triploid PacBio CLR must fall from ~0.311% at 10x per-haplotype coverage to ~0.017% at 50x.
- Corrected-read error rates should be ~9–26x lower than consensus-based state-of-the-art correctors on triploid genomes (14–30x diploid, 4–11x tetraploid).
- Corrected reads align cleanly to all three constituent reference strains with minimap2 (map-pb preset), with variants from low-frequency haplotypes retained rather than masked.
- VeChat run used default pruning thresholds (-d 0.2 min-confidence, -s 0.2 min-support); changing them would flag retention/loss of low-frequency haplotype variants.
- Output is a single full-length corrected FASTA per input read set (corrected target subreads concatenated per target read).

## Limitations

- VeChat has longer runtimes and higher peak memory usage than competing correctors, especially on large datasets.
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads; edit-distance and sequence-to-graph alignments could be replaced by more efficient routines.
- Optimal combinations of preprocessing, error correction, and assembly tools for polyploid, cancer, or metagenomic settings were beyond the study's scope and need further exploration.
- Haplotype-aware extension beyond diploid is the novelty — earlier approaches only handled diploid scenarios, so comparisons on >2-ploidy data may lack directly comparable baselines.

## Evidence

- [abstract] Unlike consensus sequence templates, graph-based reference systems are not affected by such biases, so do not mistakenly mask true variants as errors: "Unlike consensus sequence templates, graph-based reference systems are not affected by such biases, so do not mistakenly mask true variants as errors"
- [results] VeChat achieves approximately 14–30, 9–26 and 4–11 times lower error rates on diploid, triploid and tetraploid genomes, respectively.: "VeChat achieves approximately 14–30, 9–26 and 4–11 times lower error rates on diploid, triploid and tetraploid genomes, respectively."
- [results] we focused on the triploid genome, consisting of three E. coli strains as described before: "we focused on the triploid genome, consisting of three E. coli strains as described before"
- [methods] -d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2): "-d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2)"
- [readme] VeChat distinguishes errors from haplotype-specific true variants based on variation graphs: "VeChat distinguishes errors from haplotype-specific true variants based on variation graphs, which reflect a popular type of data structure for pangenome reference systems."
- [intro] This reflects a novelty for ploidies larger than two, because earlier approaches only deal with diploid scenarios21: "This reflects a novelty for ploidies larger than two, because earlier approaches only deal with diploid scenarios21"
