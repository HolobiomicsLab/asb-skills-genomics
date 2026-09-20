---
name: reciprocal-overlap-edge-annotation
description: Use when a structural-variant caller must cluster SV candidate reads
  (discordant pairs, split reads, cigar-derived deletions/insertions, breakends) into
  signature nodes on an alignment graph and connect compatible signatures without
  pairwise sequence alignment.
when_to_use_negative:
- Input is raw unaligned reads or an unfiltered VCF of finished calls rather than
  candidate read alignments — the skill operates on the clustering stage between read
  fetching and SV calling
- The goal is genotype refinement or ML quality scoring of already-called SVs; dysgu
  applies a gradient boosting classifier downstream for that purpose
- A de novo assembly stage is required to resolve multi-allelic sites or full-length
  insertion sequences, which the authors state dysgu's clustering stage does not do
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Implements SV-signature node enumeration, black-edge linking via reciprocal
    overlap and span position distance, and minimizer-sketch clustering of soft-clipped
    reads
  repo: https://github.com/kcleal/dysgu
- name: merge_intervals
  role: Interval-merging utility used to collapse connected components into quotient-graph
    partitions
  repo: https://github.com/kcleal/dysgu
- name: merge_dysgu_df
  role: DataFrame-merging utility used to collapse connected components into quotient-graph
    partitions
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:2a1299d34bcf33443315529299193f30682623b7f6adc79af1a1855e197bcda3
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
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# reciprocal-overlap-edge-annotation

## Summary

A graph-clustering skill from dysgu that converts SV candidate reads into enumerated SV-signature nodes ('discordant', 'split', 'deletion', 'insertion', 'breakend') and links compatible signatures with black edges using a reciprocal-overlap threshold of 0.1 plus a span position distance (SPD) threshold of 0.3, so soft-clipped reads can be clustered without pairwise sequence alignment.

## When to use

Use when a structural-variant caller must cluster SV candidate reads (discordant pairs, split reads, cigar-derived deletions/insertions, breakends) into signature nodes on an alignment graph and connect compatible signatures without pairwise sequence alignment.

## When NOT to use

- Input is raw unaligned reads or an unfiltered VCF of finished calls rather than candidate read alignments — the skill operates on the clustering stage between read fetching and SV calling
- The goal is genotype refinement or ML quality scoring of already-called SVs; dysgu applies a gradient boosting classifier downstream for that purpose
- A de novo assembly stage is required to resolve multi-allelic sites or full-length insertion sequences, which the authors state dysgu's clustering stage does not do

## Inputs

- Sorted and indexed BAM/CRAM of candidate SV-associated read alignments (split, discordant, supplementary/soft-clipped reads), e.g. temp_dir/input.dysgu_reads.bam produced by dysgu fetch
- Indexed reference genome FASTA
- Insert-size metrics for paired-end data (via --ibam on the main alignment file or manual -I option)

## Outputs

- Enumerated SV-signature node table with type ('discordant', 'split', 'deletion', 'insertion', 'breakend') and genomic start/end coordinates
- Edge-annotated alignment graph with black edges between signatures passing reciprocal-overlap 0.1 and SPD t<0.3
- Minimizer-sketch clusters of soft-clipped reads with support (a/2+b)>=2
- Collapsed quotient-graph partitions of connected components via merge_intervals / merge_dysgu_df

## How to apply

Load candidate read alignments and enumerate SV-signature nodes, each typed as 'discordant', 'split', 'deletion', 'insertion' or 'breakend' and annotated with genomic start and end coordinates. Add black edges between signature nodes only when they satisfy both a reciprocal-overlap threshold of 0.1 and the span position distance threshold t<0.3, computing SPD with the read-type-specific normalizer N=100 for paired-end, 600 for PacBio, and 900 for ONT data. For soft-clipped reads lacking edges, cluster via a unique-set minimizer sketch (kmer length 6, window length 12) queried through a hashmap instead of pairwise alignment, accepting clusters with support (a/2+b)>=2 and evicting hashmap entries after 150 bp. Finally collapse connected components into quotient-graph partitions using dysgu's interval-merging utilities (merge_intervals / merge_dysgu_df). The rationale is that typed signature nodes with reciprocal-overlap/SPD edge rules exploit alignment metadata rather than sequence comparison, which the paper identifies as a key reason dysgu can call SVs rapidly from both PE and LR data.

## Related tools

- **dysgu** (Implements SV-signature node enumeration, black-edge linking via reciprocal overlap and span position distance, and minimizer-sketch clustering of soft-clipped reads) — https://github.com/kcleal/dysgu
- **merge_intervals** (Interval-merging utility used to collapse connected components into quotient-graph partitions) — https://github.com/kcleal/dysgu
- **merge_dysgu_df** (DataFrame-merging utility used to collapse connected components into quotient-graph partitions) — https://github.com/kcleal/dysgu

## Examples

```
dysgu run reference.fa temp_dir input.bam > svs.vcf
```

## Evaluation signals

- The clustered signature table contains exactly one row per enumerated SV-signature node, each with a non-empty partition assignment
- No signature node is left unassigned to a quotient-graph partition after collapsing connected components
- Every black edge in the graph satisfies both reciprocal overlap >= 0.1 and SPD < 0.3 using the correct read-type normalizer (100 PE, 600 PacBio, 900 ONT)
- Soft-clip clusters are derived from minimizer-sketch hashmap queries (k=6, w=12) and each accepted cluster has support (a/2+b) >= 2
- Hashmap eviction occurs after 150 bp, so no stale soft-clip entries persist across distant loci

## Limitations

- The span position distance and minimizer-support heuristics are unoptimized; the authors note they 'could potentially be optimized using a large test set'
- Read-type-specific SPD normalization (100/600/900) must match the input technology; misclassification of PE vs PacBio vs ONT reads degrades edge linking
- Without a de novo assembly stage, multi-allelic sites and full-length insertion sequences in PE data remain unresolved by this clustering stage

## Evidence

- [methods] links compatible signatures only if they pass a reciprocal overlap of 0.1 and a separation-distance threshold, with span position distance using a normalization constant set at 100 for PE reads, 600 for PacBio and 900 for ONT reads and an SPD threshold of 0.3: [evidence span withheld — anchor status: not_found]
- [methods] soft-clipped reads lacking edges are clustered via a unique-set minimizer sketch (kmer length 6, window length 12) compared through hashmap queries instead of pairwise alignment: [evidence span withheld — anchor status: not_found]
- [discussion] Dysgu relies on a several heuristics such as span position distance, minimizer support and others that could potentially be optimized using a large test set.: "Dysgu relies on a several heuristics such as span position distance, minimizer support and others that could potentially be optimized"
- [readme] The first stage of the "run" pipeline is to separate SV-associated reads - split/discordant reads, and reads with a soft-clip >= clip_length (15 bp by default for paired-end reads).: [evidence span withheld — anchor status: external_doc]
