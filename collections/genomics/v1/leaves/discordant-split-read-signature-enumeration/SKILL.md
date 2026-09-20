---
name: discordant-split-read-signature-enumeration
description: 'Use when implementing or reproducing SV-candidate clustering from BAM/CRAM
  alignments: converting split/discordant/soft-clip signals into signature nodes and
  clustering them without pairwise sequence alignment.'
when_to_use_negative:
- Input is raw reads or a VCF of already-called SVs — this skill operates on per-read
  alignment signatures, not raw FASTQ or finished call sets
- You need pairwise sequence-level clustering of soft-clipped reads — dysgu deliberately
  replaces pairwise alignment with minimizer-sketch hashmap queries
- Events require de novo assembly resolution (e.g. full-length insertions in PE data
  or multi-allelic sites) — the article notes dysgu has no assembly stage
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: dysgu
  role: Implements the signature-enumeration, black-edge linking, minimizer-sketch
    clustering and interval-merging stages (fetch/call pipeline)
  repo: https://github.com/kcleal/dysgu
- name: merge_intervals
  role: Collapses connected components of the signature graph into quotient-graph
    partitions
  repo: https://github.com/kcleal/dysgu
- name: merge_dysgu_df
  role: Interval-merging utility applied to the clustered signature table during partition
    collapsing
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:649c79c931d40c816e50efa497e07c594550f7f6e4027beb8b8c82cf8d3cd0c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - DysguSV
  - dysgu
  - merge_dysgu_df
  - merge_intervals
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# discordant-split-read-signature-enumeration

## Summary

Reconstructs dysgu's graph-based clustering stage: candidate read alignments are converted into enumerated SV-signature nodes (discordant, split, deletion, insertion, breakend) linked by edges under reciprocal-overlap and span-position-distance thresholds, with unlinked soft-clipped reads clustered via a unique-minimizer sketch instead of pairwise sequence alignment.

## When to use

Use when implementing or reproducing SV-candidate clustering from BAM/CRAM alignments: converting split/discordant/soft-clip signals into signature nodes and clustering them without pairwise sequence alignment.

## When NOT to use

- Input is raw reads or a VCF of already-called SVs — this skill operates on per-read alignment signatures, not raw FASTQ or finished call sets
- You need pairwise sequence-level clustering of soft-clipped reads — dysgu deliberately replaces pairwise alignment with minimizer-sketch hashmap queries
- Events require de novo assembly resolution (e.g. full-length insertions in PE data or multi-allelic sites) — the article notes dysgu has no assembly stage

## Inputs

- Sorted, indexed BAM/CRAM of SV-associated reads (dysgu fetch output, e.g. temp_dir/input.dysgu_reads.bam)
- Indexed reference genome FASTA
- Candidate read alignments with CIGAR-derived split, discordant and soft-clip (>=15 bp) signals

## Outputs

- Enumerated SV-signature node table (one row per signature: type, genomic start/end, partition assignment)
- Edge list of compatible signature pairs passing reciprocal-overlap 0.1 and SPD < 0.3
- Quotient-graph partitions of clustered signatures (merged via merge_intervals / merge_dysgu_df)

## How to apply

Load candidate read alignments (from dysgu's 'fetch' output, e.g. input.dysgu_reads.bam, containing split reads, discordant pairs, and soft-clipped reads with clip length >= 15 bp) and enumerate SV-signature nodes, each typed as 'discordant', 'split', 'deletion', 'insertion' or 'breakend' with genomic start/end coordinates. Add black edges between signature nodes only when they pass a reciprocal-overlap threshold of 0.1 and a span position distance (SPD) threshold t < 0.3, where the SPD normalization constant N is read-type-specific: 100 for paired-end, 600 for PacBio and 900 for ONT reads. Soft-clipped reads lacking edges are clustered using a unique-set minimizer sketch (kmer length 6, window length 12) compared through hashmap queries rather than pairwise alignment, accepting clusters with support (a/2 + b) >= 2 and evicting hashmap entries after 150 bp. Finally collapse connected components into quotient-graph partitions using dysgu's merge_intervals / merge_dysgu_df interval-merging utilities.

## Related tools

- **dysgu** (Implements the signature-enumeration, black-edge linking, minimizer-sketch clustering and interval-merging stages (fetch/call pipeline)) — https://github.com/kcleal/dysgu
- **merge_intervals** (Collapses connected components of the signature graph into quotient-graph partitions) — https://github.com/kcleal/dysgu
- **merge_dysgu_df** (Interval-merging utility applied to the clustered signature table during partition collapsing) — https://github.com/kcleal/dysgu

## Examples

```
dysgu call --ibam all_reads.bam reference.fa temp_dir temp_dir/input.dysgu_reads.bam > svs.vcf
```

## Evaluation signals

- Clustered signature table contains one row per enumerated signature (typed discordant/split/deletion/insertion/breakend) with a non-empty partition assignment
- No signature node is left unassigned to a quotient-graph partition after merge_intervals/merge_dysgu_df collapsing
- Every black edge satisfies both reciprocal overlap >= 0.1 and SPD < 0.3 with the read-type-specific normalizer (N=100 PE, 600 PacBio, 900 ONT)
- Accepted soft-clip clusters meet support (a/2 + b) >= 2 and the minimizer sketch uses k=6, window=12 with 150 bp hashmap eviction
- downstream 'dysgu test' passes after a full run pipeline on sample data

## Limitations

- Heuristics such as span position distance and minimizer support are unoptimized and could be tuned with a larger test set
- Soft-clip clustering depends on clip_length threshold (15 bp default for paired-end reads); shorter clips are not fetched
- No de novo assembly stage, so multi-allelic sites and full-length insertions in PE data remain unresolved
- Default sequence divergence of 0.02 can be too stringent for lower-accuracy ONT reads, filtering more reads and reducing sensitivity

## Evidence

- [methods] Signature enumeration with types and coordinates: [evidence span withheld — anchor status: not_found]
- [methods] Edge thresholds: [evidence span withheld — anchor status: not_found]
- [methods] Minimizer sketch clustering: [evidence span withheld — anchor status: not_found]
- [discussion] Heuristics unoptimized caveat: "Dysgu relies on a several heuristics such as span position distance, minimizer support and others that could potentially be optimized using a large test set."
- [readme] Readme fetch stage producing the clustering input: [evidence span withheld — anchor status: external_doc]
