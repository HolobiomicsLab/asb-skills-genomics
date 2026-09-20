---
name: unique-minimizer-sketching
description: 'Use when reconstructing dysgu''s alignment-graph clustering: soft-clipped
  SV-supporting reads (clip length >= 15 bp by default for PE) have no black edges
  linking their SV-signature nodes, so their sequences must be clustered by minimizer-sketch
  similarity instead of pairwise alignment.'
when_to_use_negative:
- Signature nodes already share black edges satisfying reciprocal overlap >= 0.1 and
  SPD < 0.3 — they are clustered by graph connectivity, not the minimizer sketch
- The task is benchmarking or comparing called SVs against a truth set — use svbench
  / GIAB-style evaluation instead of this clustering stage
- The variant caller is not dysgu; the SPD normalizers (100/600/900), k=6/w=12 sketch,
  and (a/2+b)>=2 support rule are dysgu-internal heuristics
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: 'Implements the graph-clustering stage: signature-node enumeration, black-edge
    creation, minimizer-sketch clustering of soft-clipped reads, and quotient-graph
    partitioning'
  repo: https://github.com/kcleal/dysgu
- name: merge_intervals
  role: Dysgu utility used to collapse connected components into quotient-graph partitions
  repo: https://github.com/kcleal/dysgu
- name: merge_dysgu_df
  role: Dysgu utility for merging dysgu dataframes during interval-based partitioning
    of clustered signatures
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:c78cccff9c58c21be2b5882d7948cc7de4741d801757252da4fbe38cf48974bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - DysguSV
  - dysgu
  - merge_dysgu_df
  - merge_intervals
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# unique-minimizer-sketching

## Summary

Clusters soft-clipped read sequences in dysgu's SV-calling graph stage using a unique-set minimizer sketch (kmer length 6, window length 12) compared via hashmap queries, replacing costly pairwise sequence alignment between reads that lack black signature edges.

## When to use

Use when reconstructing dysgu's alignment-graph clustering: soft-clipped SV-supporting reads (clip length >= 15 bp by default for PE) have no black edges linking their SV-signature nodes, so their sequences must be clustered by minimizer-sketch similarity instead of pairwise alignment.

## When NOT to use

- Signature nodes already share black edges satisfying reciprocal overlap >= 0.1 and SPD < 0.3 — they are clustered by graph connectivity, not the minimizer sketch
- The task is benchmarking or comparing called SVs against a truth set — use svbench / GIAB-style evaluation instead of this clustering stage
- The variant caller is not dysgu; the SPD normalizers (100/600/900), k=6/w=12 sketch, and (a/2+b)>=2 support rule are dysgu-internal heuristics

## Inputs

- Sorted/indexed .bam/.cram of SV-associated reads produced by 'dysgu fetch' (input.dysgu_reads.bam), including reads with soft-clips >= 15 bp (default clip_length for paired-end)
- Enumerated SV-signature nodes typed as 'discordant', 'split', 'deletion', 'insertion' or 'breakend' with genomic start/end positions
- Black-edge adjacency satisfying reciprocal overlap >= 0.1 and SPD t < 0.3 (N=100 PE, 600 PacBio, 900 ONT)
- Read type annotation (paired-end vs PacBio vs ONT) to select the SPD normalizer

## Outputs

- Clusters of soft-clipped reads with support (a/2 + b) >= 2 assigned via minimizer-sketch hashmap matches
- Quotient-graph partitions of the signature table (one non-empty partition assignment per enumerated signature)
- Merged interval partitions via dysgu's merge_intervals / merge_dysgu_df utilities

## How to apply

First enumerate SV-signature nodes (typed 'discordant', 'split', 'deletion', 'insertion' or 'breakend' with genomic start/end coordinates) from candidate read alignments, and add black edges only between signatures passing reciprocal overlap >= 0.1 and span position distance t < 0.3, with the read-type-specific normalizer N=100 for paired-end, 600 for PacBio and 900 for ONT reads. For soft-clipped reads whose signatures remain edge-less, compute a unique-set minimizer sketch of each sequence with kmer length 6 and window length 12, then compare sketches through hashmap queries rather than pairwise sequence alignment. Accept a cluster when support (a/2 + b) >= 2, where a and b count the sketch-matched reads, and evict entries from the hashmap after 150 bp to bound memory. Finally collapse connected components into quotient-graph partitions using dysgu's interval-merging utilities (merge_intervals / merge_dysgu_df), so every signature node lands in exactly one partition.

## Related tools

- **dysgu** (Implements the graph-clustering stage: signature-node enumeration, black-edge creation, minimizer-sketch clustering of soft-clipped reads, and quotient-graph partitioning) — https://github.com/kcleal/dysgu
- **merge_intervals** (Dysgu utility used to collapse connected components into quotient-graph partitions) — https://github.com/kcleal/dysgu
- **merge_dysgu_df** (Dysgu utility for merging dysgu dataframes during interval-based partitioning of clustered signatures) — https://github.com/kcleal/dysgu

## Examples

```
dysgu fetch samp1_temp input.bam  # produce input.dysgu_reads.bam with soft-clipped reads for the clustering stage
```

## Evaluation signals

- The clustered signature table contains exactly one row per enumerated signature, each with a non-empty partition assignment
- No signature node is left unassigned to a quotient-graph partition after merge_intervals / merge_dysgu_df
- Every accepted minimizer-sketch cluster satisfies support (a/2 + b) >= 2
- Hashmap entries are evicted after 150 bp, so no cluster links reads more than 150 bp beyond the sketch window
- Downstream call quality: on HG002 Tier 1 at 20x, dysgu reached 95.7% deletion precision and 62.2% deletion / 23.7% insertion recall, so a correct implementation should preserve high precision-recall behaviour

## Limitations

- Dysgu's heuristics — including span position distance and minimizer support — are unoptimized and could potentially be tuned with a larger test set
- No de novo assembly stage is incorporated, so multi-allelic sites and full-length insertion sequences in paired-end data remain unresolved and true-positive duplication can occur
- The default divergence setting (0.02) can be too stringent for lower-accuracy reads, filtering more reads and lowering sensitivity; '--divergence auto' is recommended for higher-error data

## Evidence

- [methods] soft-clipped reads lacking edges are clustered via a unique-set minimizer sketch (kmer length 6, window length 12) compared through hashmap queries instead of pairwise alignment: [evidence span withheld — anchor status: not_found]
- [discussion] Dysgu relies on a several heuristics such as span position distance, minimizer support and others that could potentially be optimized using a large test set.: "Dysgu relies on a several heuristics such as span position distance, minimizer support and others that could potentially be optimized using a large test set."
- [readme] The first stage of the "run" pipeline is to separate SV-associated reads - split/discordant reads, and reads with a soft-clip >= clip_length (15 bp by default for paired-end reads).: [evidence span withheld — anchor status: external_doc]
