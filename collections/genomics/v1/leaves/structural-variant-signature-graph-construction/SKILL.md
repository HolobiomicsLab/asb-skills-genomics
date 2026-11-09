---
name: structural-variant-signature-graph-construction
description: 'Use when reproducing or re-implementing dysgu''s SV clustering: you
  have per-read SV signatures from split/discordant/supplementary alignments plus
  soft-clipped sequences and need to build a signature graph and quotient-graph partitions.'
when_to_use_negative:
- Input is a finalized VCF of SV calls rather than per-read signature data — downstream
  filtering (e.g. model probability >= 0.5 PacBio / >= 0.35 ONT) applies instead
- The analysis requires de novo assembly to resolve multi-allelic sites or full-length
  insertion sequences in PE data, which the signature graph does not address
- Pairwise sequence alignment of soft-clip sequences is explicitly required; dysgu's
  design goal is to avoid it via minimizer sketches
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Performs the signature enumeration, black-edge linking, minimizer-sketch clustering
    and quotient-graph partitioning; installed via pip/conda or docker
  repo: https://github.com/kcleal/dysgu
- name: merge_intervals
  role: Dysgu interval-merging utility used to collapse connected components into
    quotient-graph partitions
  repo: https://github.com/kcleal/dysgu
- name: merge_dysgu_df
  role: Dysgu dataframe-merging utility used alongside merge_intervals for partitioning
    clustered signatures
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:8803cc8f5c848c2040352150f2975053868375078cf690bf923b11dcac761559
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# structural-variant-signature-graph-construction

## Summary

Reconstructs dysgu's graph-based SV clustering stage: candidate read alignments are converted into enumerated SV-signature nodes ('discordant', 'split', 'deletion', 'insertion', 'breakend'), linked with black edges under reciprocal-overlap and span-position-distance thresholds, and unlinked soft-clipped reads are clustered via a unique-set minimizer sketch instead of pairwise alignment.

## When to use

Use when reproducing or re-implementing dysgu's SV clustering: you have per-read SV signatures from split/discordant/supplementary alignments plus soft-clipped sequences and need to build a signature graph and quotient-graph partitions.

## When NOT to use

- Input is a finalized VCF of SV calls rather than per-read signature data — downstream filtering (e.g. model probability >= 0.5 PacBio / >= 0.35 ONT) applies instead
- The analysis requires de novo assembly to resolve multi-allelic sites or full-length insertion sequences in PE data, which the signature graph does not address
- Pairwise sequence alignment of soft-clip sequences is explicitly required; dysgu's design goal is to avoid it via minimizer sketches

## Inputs

- Candidate read alignments (sorted/indexed .bam or .cram) containing split, discordant and supplementary mappings and soft-clipped reads
- SV-signature calls from dysgu's fetch stage (input.dysgu_reads.bam)
- Reference genome in FASTA format

## Outputs

- Enumerated SV-signature node table with type and genomic start/end coordinates per signature
- Black-edge links between compatible signatures passing reciprocal-overlap 0.1 and SPD < 0.3
- Minimizer-sketch clusters of soft-clipped reads (k=6, window=12) with support (a/2+b) >= 2
- Quotient-graph partitions of connected components via merge_intervals / merge_dysgu_df

## How to apply

Enumerate each candidate signature as a typed node ('discordant', 'split', 'deletion', 'insertion' or 'breakend') annotated with genomic start/end coordinates. Add black edges only between signature pairs passing both a reciprocal-overlap threshold of 0.1 and the span position distance (SPD) threshold t < 0.3, where SPD uses a read-type-specific normalization constant N = 100 for paired-end, 600 for PacBio, and 900 for ONT reads. For soft-clipped reads lacking edges, cluster sequences with a unique-set minimizer sketch (kmer length 6, window length 12) compared through hashmap queries rather than pairwise alignment, accepting clusters with support (a/2 + b) >= 2 and evicting hashmap entries after 150 bp. Finally collapse connected components into quotient-graph partitions using dysgu's interval-merging utilities (merge_intervals / merge_dysgu_df).

## Related tools

- **dysgu** (Performs the signature enumeration, black-edge linking, minimizer-sketch clustering and quotient-graph partitioning; installed via pip/conda or docker) — https://github.com/kcleal/dysgu
- **merge_intervals** (Dysgu interval-merging utility used to collapse connected components into quotient-graph partitions) — https://github.com/kcleal/dysgu
- **merge_dysgu_df** (Dysgu dataframe-merging utility used alongside merge_intervals for partitioning clustered signatures) — https://github.com/kcleal/dysgu

## Examples

```
dysgu fetch samp1_temp input.bam
```

## Evaluation signals

- Every enumerated signature appears as exactly one row in the clustered signature table with a non-empty partition assignment
- No signature node is left unassigned to a quotient-graph partition
- Every black edge satisfies both reciprocal overlap >= 0.1 and SPD < 0.3 using N = 100 (PE), 600 (PacBio) or 900 (ONT)
- Each accepted soft-clip cluster has support (a/2 + b) >= 2 with minimizer k=6, window=12
- Hashmap entries are evicted after 150 bp, so no stale soft-clip cluster survives beyond that span

## Limitations

- Heuristics such as span position distance and minimizer support are unoptimized and could potentially be tuned with a larger test set
- No de novo assembly stage is incorporated, so multi-allelic sites and full-length insertion sequences in PE data remain unresolved
- The heuristics were calibrated per read type; using the wrong normalizer (100/600/900) for a read type will mis-link signatures

## Evidence

- [abstract] Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs"
- [intro] Signature detection scope: "Conceptually, dysgu identifies SVs from alignment cigar information as well as discordant and split-read mappings."
- [discussion] Unoptimized heuristics caveat: "Dysgu relies on a several heuristics such as span position distance, minimizer support and others that could potentially be optimized using a large test set."
- [readme] Soft-clip read separation in fetch: [evidence span withheld — anchor status: external_doc]
