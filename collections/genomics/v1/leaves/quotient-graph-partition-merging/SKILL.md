---
name: quotient-graph-partition-merging
description: Use when SV-signature nodes derived from paired-end, PacBio or ONT BAM/CRAM
  alignments must be linked by reciprocal-overlap and span-position-distance criteria
  and collapsed into quotient-graph partitions, with soft-clipped reads clustered
  via minimizer sketches.
when_to_use_negative:
- Input is a plain BAM of all reads rather than the SV-associated subset — dysgu expects
  reads pre-filtered by the 'fetch' stage (split/discordant reads and soft-clips >=
  15 bp)
- The goal is full-length insertion sequence resolution in PE data — dysgu has no
  de novo assembly stage, so multi-allelic sites and full-length insertions in PE
  data remain unresolved
- Evaluation of small-variant (SNV/indel) callers rather than SVs — tools like CHM-eval/syndip
  target small variant calling, not the SV-signature clustering covered here
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: dysgu
  role: Runs the fetch/call pipeline whose clustering stage enumerates SV-signature
    nodes, links them with black edges, and clusters soft-clipped reads before interval
    merging; exposes the python-API used for post-calling model switching
  repo: https://github.com/kcleal/dysgu
- name: merge_intervals
  role: Dysgu interval-merging utility used to collapse connected components into
    quotient-graph partitions
  repo: https://github.com/kcleal/dysgu
- name: merge_dysgu_df
  role: Dysgu dataframe-merging utility used alongside merge_intervals to collapse
    the clustered signature table into quotient-graph partitions
  repo: https://github.com/kcleal/dysgu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:3e1c794137425218c54ede352ca26549e41f64a23c680e2901a708bd5a8aae3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - DysguSV
  - dysgu
  - merge_dysgu_df
  - merge_intervals
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# quotient-graph-partition-merging

## Summary

Dysgu's clustering stage converts enumerated SV-signature nodes ('discordant', 'split', 'deletion', 'insertion', 'breakend') into linked graph components and then collapses connected components into quotient-graph partitions using dysgu's interval-merging utilities (merge_intervals / merge_dysgu_df), clustering soft-clipped reads via a unique-set minimizer sketch instead of pairwise alignment.

## When to use

Use when SV-signature nodes derived from paired-end, PacBio or ONT BAM/CRAM alignments must be linked by reciprocal-overlap and span-position-distance criteria and collapsed into quotient-graph partitions, with soft-clipped reads clustered via minimizer sketches.

## When NOT to use

- Input is a plain BAM of all reads rather than the SV-associated subset — dysgu expects reads pre-filtered by the 'fetch' stage (split/discordant reads and soft-clips >= 15 bp)
- The goal is full-length insertion sequence resolution in PE data — dysgu has no de novo assembly stage, so multi-allelic sites and full-length insertions in PE data remain unresolved
- Evaluation of small-variant (SNV/indel) callers rather than SVs — tools like CHM-eval/syndip target small variant calling, not the SV-signature clustering covered here

## Inputs

- Sorted/indexed BAM or CRAM of SV-associated reads (e.g. temp_dir/input.dysgu_reads.bam produced by dysgu fetch)
- Indexed reference genome FASTA (e.g. reference.fa)
- Insert-size metrics from the main alignment file (via dysgu call --ibam) for paired-end data

## Outputs

- Clustered signature table with one row per enumerated SV-signature and a non-empty quotient-graph partition assignment
- SV calls emitted as VCF (svs.vcf) for downstream machine-learning quality scoring

## How to apply

First enumerate SV-signature nodes from the candidate read alignments, each typed as 'discordant', 'split', 'deletion', 'insertion' or 'breakend' and annotated with genomic start/end coordinates. Add black edges between signature nodes only when they pass a reciprocal-overlap threshold of 0.1 and a span-position-distance (SPD) threshold t<0.3, using the read-type-specific normalization constant N=100 for paired-end, 600 for PacBio and 900 for ONT reads. Cluster soft-clipped reads lacking edges with the unique-set minimizer sketch (kmer length 6, window length 12) compared through hashmap queries instead of pairwise alignment, accepting clusters with support (a/2+b)>=2 and evicting hashmap entries after 150 bp. Finally collapse connected components into quotient-graph partitions using dysgu's merge_intervals / merge_dysgu_df interval-merging utilities so every signature row receives a non-empty partition assignment.

## Related tools

- **dysgu** (Runs the fetch/call pipeline whose clustering stage enumerates SV-signature nodes, links them with black edges, and clusters soft-clipped reads before interval merging; exposes the python-API used for post-calling model switching) — https://github.com/kcleal/dysgu
- **merge_intervals** (Dysgu interval-merging utility used to collapse connected components into quotient-graph partitions) — https://github.com/kcleal/dysgu
- **merge_dysgu_df** (Dysgu dataframe-merging utility used alongside merge_intervals to collapse the clustered signature table into quotient-graph partitions) — https://github.com/kcleal/dysgu

## Examples

```
dysgu run reference.fa temp_dir input.bam > svs.vcf
```

## Evaluation signals

- Clustered signature table contains one row per enumerated signature with a non-empty partition assignment
- No signature node is left unassigned to a quotient-graph partition
- Black edges exist only between signature pairs passing reciprocal overlap >= 0.1 and SPD threshold t < 0.3 (with N=100 PE, 600 PacBio, 900 ONT)
- Every soft-clip cluster accepted via the minimizer sketch satisfies support (a/2+b) >= 2
- Downstream calls carry probability values from the gradient-boosting classifier and survive filtering at max model probability >= 0.5 (PacBio) or >= 0.35 (ONT)

## Limitations

- Dysgu relies on several heuristics — span position distance, minimizer support and others — that are unoptimized and could potentially be tuned using a large test set
- No de novo assembly stage is incorporated, so multi-allelic sites and full-length insertion sequences in PE data remain unresolved and true-positive events can be duplicated
- For low-accuracy long reads the default --divergence 0.02 can be too stringent, filtering more reads and lowering sensitivity; use '--divergence auto'

## Evidence

- [full_text] Dysgu converts candidate alignments into SV-signature nodes ... and links compatible signatures only if they pass a reciprocal overlap of 0.1 and a separation-distance threshold: [evidence span withheld — anchor status: not_found]
- [full_text] soft-clipped reads lacking edges are clustered via a unique-set minimizer sketch (kmer length 6, window length 12) compared through hashmap queries instead of pairwise alignment: [evidence span withheld — anchor status: not_found]
- [discussion] Dysgu relies on a several heuristics such as span position distance, minimizer support and others that could potentially be optimized using a large test set: "Dysgu relies on a several heuristics such as span position distance, minimizer support and others that could potentially be optimized using a large test set"
- [readme] This will first run the `fetch` command that will create a temporary bam file plus other analysis files: [evidence span withheld — anchor status: external_doc]
