---
name: phred-probability-edge-weighting
description: Use when building a variation graph from a long-read alignment pile for
  error correction and you must distinguish error-induced nodes/edges from true (possibly
  low-frequency haplotype) variants using coverage and per-base Phred-quality weights.
when_to_use_negative:
- You need a simple consensus-based polishing step with minimal resources — the article
  notes VeChat does not win in computational resources, showing longer runtimes and
  higher peak memory usage on large datasets
- The input is short accurate reads or already-corrected reads — this weighting/pruning
  scheme is designed for noisy long reads from TGS platforms (PacBio, ONT)
- You require the haplotype-aware consensus round of cycle 2 — that step derives consensus
  sequences via dynamic programming without re-pruning the graph
tools:
- name: VeChat
  role: Implements the frequent-itemset-weighted graph construction, Confidence/Support
    edge pruning (--min-confidence, --min-support) and iterative re-alignment
  repo: https://github.com/HaploKit/vechat
- name: Racon
  role: Modified Racon module bundled with VeChat provides the POA routine used to
    construct the ad-hoc variation graph that is subsequently weighted and pruned
  repo: https://github.com/lbcb-sci/racon
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:dde80d0a9ac304fde6f55adcde296dcb61d39c5f96d0df31b5712a6a0c25a323
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Racon
  - VeChat
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# phred-probability-edge-weighting

## Summary

Weight the nodes and edges of the ad-hoc variation graph built by POA in VeChat's step 4 using read coverage and per-base Phred probabilities, feeding a frequent itemset model that separates sequencing-error artifacts from true sequential phenomena before confidence/support-based edge pruning.

## When to use

Use when building a variation graph from a long-read alignment pile for error correction and you must distinguish error-induced nodes/edges from true (possibly low-frequency haplotype) variants using coverage and per-base Phred-quality weights.

## When NOT to use

- You need a simple consensus-based polishing step with minimal resources — the article notes VeChat does not win in computational resources, showing longer runtimes and higher peak memory usage on large datasets
- The input is short accurate reads or already-corrected reads — this weighting/pruning scheme is designed for noisy long reads from TGS platforms (PacBio, ONT)
- You require the haplotype-aware consensus round of cycle 2 — that step derives consensus sequences via dynamic programming without re-pruning the graph

## Inputs

- Gzip-compressed FASTA/FASTQ raw long-read file (e.g. example/reads.fq.gz) with per-base Phred quality scores
- Per-window read alignment pile of all reads overlapping the selected target read
- Ad-hoc variation graph constructed via POA from the alignment pile

## Outputs

- Pruned variation graph whose nodes/edges are annotated with coverage- and Phred-probability-derived weights
- Directional Confidence(e) and Support(e) values per edge
- Converged graph (3 iterations) retaining only true sequential phenomena, from which the corrected subread is read off as the optimal alignment path

## How to apply

After constructing the per-window ad-hoc variation graph with the partial order alignment (POA) routine from the bundled Racon module, annotate each node and edge with weights derived from read coverage of the alignment pile and the per-base Phred probabilities carried in the FASTQ input. Apply the frequent itemset model — which jointly considers read coverage, sequencing errors, and co-occurrence of characters in reads — to compute a directional Confidence(e) and Support(e) for every edge. Prune edges falling below the minimum-confidence threshold (-d, default 0.2) or the minimum-support threshold (-s, default 0.2). Re-align the target subread against the pruned graph and repeat the prune-and-realign cycle for 3 iterations until convergence, so the surviving graph contains only nodes and edges reflecting true sequential phenomena; the optimal alignment path of the target subread through this converged graph is then read off as the corrected subread. The rationale is that graph-based reference systems, unlike consensus templates, retain low-frequency true variants and do not mistakenly mask them as errors.

## Related tools

- **VeChat** (Implements the frequent-itemset-weighted graph construction, Confidence/Support edge pruning (--min-confidence, --min-support) and iterative re-alignment) — https://github.com/HaploKit/vechat
- **Racon** (Modified Racon module bundled with VeChat provides the POA routine used to construct the ad-hoc variation graph that is subsequently weighted and pruned) — https://github.com/lbcb-sci/racon

## Examples

```
vechat -d 0.2 -s 0.2 example/reads.fq.gz
```

## Evaluation signals

- Pruning converges under the default thresholds (confidence 0.2, support 0.2) within the 3 prune-and-realign iterations
- Every edge retained in the final graph meets both the minimum-confidence (-d 0.2) and minimum-support (-s 0.2) cutoffs
- The surviving graph retains only nodes/edges supported as true sequential phenomena rather than error artifacts
- Node/edge weights are traceable to read-coverage counts and per-base Phred probabilities of the alignment pile
- The corrected subread corresponds to an optimal alignment path through the converged graph

## Limitations

- On large datasets VeChat experiences longer runtimes and higher peak memory usage than competing approaches
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads
- The weighting depends on the quality of the initial ad-hoc graph from POA; computations such as edit-distance-based and sequence-to-graph alignments could be replaced by more efficient routines in the future

## Evidence

- [results] Frequent itemset model over coverage, errors and co-occurrence: "we make use of a frequent itemset model that involves read coverage, sequencing errors and co-occurrence of characters in reads"
- [results] Iterative pruning until convergence: "The procedure of spurious node and edge removal followed by re-alignment is repeated until convergence"
- [methods] Confidence threshold CLI option: "-d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2)"
- [methods] Support threshold CLI option: "-s MIN_SUPPORT, --min-support MIN_SUPPORT minimum support for keeping edges in the graph (default: 0.2)"
- [readme] README: pruning rationale: "nodes and edges that are due to errors are pruned from that graph by way of an iterative procedure that is based on principles from frequent itemset mining"
