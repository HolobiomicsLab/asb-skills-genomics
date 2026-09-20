---
name: soft-clip-sequence-clustering
description: Use when building or reproducing a graph-based SV-calling clustering
  stage (as in dysgu) where soft-clipped reads must be grouped by sequence similarity
  cheaply, or when enumerating SV-signature nodes from discordant/split/deletion/insertion/breakend
  evidence.
when_to_use_negative:
- Input is a VCF of already-called SVs — clustering of raw soft-clipped signatures
  is a pre-call stage, not a post-call merge/filter step
- 'You need full break-end sequence resolution or multi-allelic site resolution: the
  paper notes no de novo assembly stage is incorporated, so full-length insertions
  in PE data remain unresolved'
- Reads are not SV-associated (no soft-clip >= 15 bp, no split/discordant/supplementary
  mapping) — dysgu's fetch stage will simply not emit them
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: dysgu
  role: 'Implements the alignment-graph clustering: fetches SV-associated reads, enumerates
    signature nodes, links edges via SPD, and clusters soft clips via minimizer sketch'
  repo: https://github.com/kcleal/dysgu
- name: merge_intervals
  role: Dysgu interval-merging utility used to collapse connected components into
    quotient-graph partitions
  repo: https://github.com/kcleal/dysgu
- name: merge_dysgu_df
  role: Dysgu dataframe-merging utility used to collapse connected components into
    quotient-graph partitions
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:b025b78f81576f3c88c18d11a0826b28838a12b686e22dd55283ba65ee07a17d
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
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# soft-clip-sequence-clustering

## Summary

Clusters soft-clipped read sequences from SV candidate alignments without pairwise sequence alignment, using dysgu's unique-set minimizer sketch (kmer length 6, window 12) compared through hashmap queries, alongside graph-based SV-signature clustering with reciprocal-overlap and span-position-distance thresholds.

## When to use

Use when building or reproducing a graph-based SV-calling clustering stage (as in dysgu) where soft-clipped reads must be grouped by sequence similarity cheaply, or when enumerating SV-signature nodes from discordant/split/deletion/insertion/breakend evidence.

## When NOT to use

- Input is a VCF of already-called SVs — clustering of raw soft-clipped signatures is a pre-call stage, not a post-call merge/filter step
- You need full break-end sequence resolution or multi-allelic site resolution: the paper notes no de novo assembly stage is incorporated, so full-length insertions in PE data remain unresolved
- Reads are not SV-associated (no soft-clip >= 15 bp, no split/discordant/supplementary mapping) — dysgu's fetch stage will simply not emit them

## Inputs

- Candidate read alignments (BAM/CRAM) from the dysgu fetch stage, including split, discordant and supplementary-mapping reads
- Soft-clipped read sequences (soft-clip >= 15 bp by default for paired-end reads)
- Insert-size metrics from the main alignment file (--ibam) for paired-end data
- Read-type metadata indicating paired-end, PacBio or ONT provenance (selects SPD normalizer)

## Outputs

- Enumerated SV-signature node table, one row per signature, typed as discordant/split/deletion/insertion/breakend with genomic start/end coordinates
- Edge set of compatible signature pairs passing reciprocal-overlap 0.1 and SPD t<0.3
- Soft-clip clusters with support (a/2+b)>=2
- Quotient-graph partitions per connected component (non-empty partition assignment for every signature node)

## How to apply

First enumerate SV-signature nodes from candidate read alignments, each typed as 'discordant', 'split', 'deletion', 'insertion' or 'breakend' and annotated with genomic start/end coordinates. Link compatible signature nodes with edges only when they pass both a reciprocal-overlap threshold of 0.1 and a span position distance (SPD) threshold t<0.3, where SPD uses the read-type-specific normalization constant N=100 for paired-end, N=600 for PacBio and N=900 for ONT reads. For soft-clipped reads that lack graph edges, compute a unique-set minimizer sketch with kmer length 6 and window length 12 and cluster reads by comparing sketches through hashmap queries rather than pairwise sequence alignment; accept clusters only when support (a/2+b)>=2 and evict hashmap entries after 150 bp. Finally collapse connected components into quotient-graph partitions using dysgu's interval-merging utilities (merge_intervals / merge_dysgu_df). The rationale is that minimizer sketching avoids the cost of all-against-all soft-clip alignment while still grouping reads that support the same break site.

## Related tools

- **dysgu** (Implements the alignment-graph clustering: fetches SV-associated reads, enumerates signature nodes, links edges via SPD, and clusters soft clips via minimizer sketch) — https://github.com/kcleal/dysgu
- **merge_intervals** (Dysgu interval-merging utility used to collapse connected components into quotient-graph partitions) — https://github.com/kcleal/dysgu
- **merge_dysgu_df** (Dysgu dataframe-merging utility used to collapse connected components into quotient-graph partitions) — https://github.com/kcleal/dysgu

## Examples

```
dysgu run -p4 reference.fa temp_dir input.bam > svs.vcf
```

## Evaluation signals

- The clustered signature table contains exactly one row per enumerated signature, each with a non-empty quotient-graph partition assignment
- No signature node is left unassigned to a quotient-graph partition
- Every retained soft-clip cluster has support meeting (a/2+b)>=2
- All signature-pair edges satisfy both reciprocal-overlap >= 0.1 and SPD t<0.3 with the correct read-type normalizer (N=100 PE / 600 PacBio / 900 ONT)
- Soft-clip sketching is performed via hashmap queries on unique minimizers (k=6, w=12), with hashmap entries evicted after 150 bp

## Limitations

- Span position distance and minimizer-support heuristics are unoptimized; the authors note they 'could potentially be optimized using a large test set'
- No de novo assembly stage is incorporated, so multi-allelic sites and full-length insertions in PE data remain unresolved
- Soft-clip length threshold defaults to 15 bp for paired-end reads; shorter clips will not enter the clustering pool

## Evidence

- [readme] Fetch separates soft-clipped reads feeding the clustering stage: [evidence span withheld — anchor status: external_doc]
- [discussion] Heuristics are unoptimized per the authors: "Dysgu relies on a several heuristics such as span position distance, minimizer support and others that could potentially be optimized using a large test set"
- [intro] Downstream consensus/remapping context for anomalous sequences: "Dysgu employs a fast consensus sequence algorithm, inspired by the positional de Brujin graph, followed by remapping of anomalous sequences to discover additional small SVs"
- [abstract] Signal sources that feed signature enumeration: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs, before classifying events using machine learning"
