---
name: descending-kmer-size-parameter-selection
description: Use when running the ntEdit+Sealer finishing protocol on a draft long-read
  assembly with short reads and you must choose k-mer lengths and Bloom filter size
  (b) parameters for the Makefile 'finish' target.
when_to_use_negative:
- The protocol is alignment-based rather than k-mer/Bloom-filter-based — hybrid polishing
  tools like Pilon or Racon that rely on read alignments are a different approach
  and this parameter selection does not apply to them
- Only a single k value can be used with no descending k cascade, undermining the
  multi-k Bloom filter design of ntEdit+Sealer
tools:
- name: ntHits
  role: Counts/filters short-read k-mers used to build the Bloom filters at the selected
    k values and filter size b
  repo: https://github.com/BirolLab/nthits
- name: ntEdit
  role: Performs alignment-free, k-mer-based correction of base errors and small indels
    in the draft assembly at the chosen k cascade
  repo: https://github.com/BirolLab/ntEdit
- name: ABySS-Bloom
  role: Creates Bloom filters from short-read k-mers at the requested filter size
  repo: https://github.com/BirolLab/abyss
- name: Sealer
  role: Fills assembly gaps and ntEdit-flagged problematic regions using the same
    Bloom filter cascade built from the selected k and b parameters
  repo: https://github.com/BirolLab/abyss
- name: ntEdit+Sealer
  role: The Makefile pipeline whose 'finish' target consumes the k and b parameters
    to orchestrate polishing and gap-filling
  repo: https://github.com/BirolLab/ntedit_sealer_protocol
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:5c04aee94da0df72679a9de888474f3c5defd9cd19f8c12489bc6a7cceab7e5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/nthits
  tools:
  - ABySS
  - ABySS-Bloom
  - GNU Make
  - Sealer
  - ntEdit
  - ntEdit+Sealer
  - ntHits
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# descending-kmer-size-parameter-selection

## Summary

Selecting and passing optimal k-mer lengths (k) and Bloom filter size (b) to the ntEdit+Sealer Makefile finishing pipeline, with k values supplied in strictly descending order (e.g. '80 65 50') so the Bloom filter cascade supports ntEdit's error correction at long k before Sealer gap-fills at shorter k.

## When to use

Use when running the ntEdit+Sealer finishing protocol on a draft long-read assembly with short reads and you must choose k-mer lengths and Bloom filter size (b) parameters for the Makefile 'finish' target.

## When NOT to use

- The protocol is alignment-based rather than k-mer/Bloom-filter-based — hybrid polishing tools like Pilon or Racon that rely on read alignments are a different approach and this parameter selection does not apply to them
- Only a single k value can be used with no descending k cascade, undermining the multi-k Bloom filter design of ntEdit+Sealer

## Inputs

- Draft long-read assembly FASTA (e.g. ecoli_shasta.fa, draft-assembly.fa)
- Paired short reads as .fq.gz files (e.g. SRR158595208_1.fq.gz / _2.fq.gz)
- Chosen k-mer lengths supplied in descending order (e.g. '80 65 50')
- Chosen Bloom filter size (e.g. b=200M for E. coli demo, b=5G in the generic methods example)

## Outputs

- Finished, polished, gap-filled assembly FASTA with suffix .ntedit_edited.prepd.sealer_scaffold.fa (e.g. ecoli_shasta.ntedit_edited.prepd.sealer_scaffold.fa)
- Bloom filter cascade built at the requested size by ntHits and ABySS-Bloom from short-read k-mers

## How to apply

Before invoking 'ntedit-sealer finish', choose k-mer lengths suited to the read data and genome size and supply them in descending order, as the demo run uses k='80 65 50'. Set the Bloom filter size b to a value scaled to genome/reads size — the E. coli NDM5 demonstration uses b=200M for the SRR158595208 short-read Bloom filters built by ntHits and ABySS-Bloom, while the methods show b=5G for the generic draft-assembly example. Pass these as Makefile variables (seqs=<draft.fa> reads='<read1.fq.gz read2.fq.gz>' k='<k values>' b=<size>). The descending k ordering ensures the alignment-free, k-mer-based polishing (ntEdit correction of base errors and small indels) operates across the k cascade before Sealer fills gaps and ntEdit-flagged problematic regions using the same Bloom filter cascade.

## Related tools

- **ntHits** (Counts/filters short-read k-mers used to build the Bloom filters at the selected k values and filter size b) — https://github.com/BirolLab/nthits
- **ntEdit** (Performs alignment-free, k-mer-based correction of base errors and small indels in the draft assembly at the chosen k cascade) — https://github.com/BirolLab/ntEdit
- **ABySS-Bloom** (Creates Bloom filters from short-read k-mers at the requested filter size) — https://github.com/BirolLab/abyss
- **Sealer** (Fills assembly gaps and ntEdit-flagged problematic regions using the same Bloom filter cascade built from the selected k and b parameters) — https://github.com/BirolLab/abyss
- **ntEdit+Sealer** (The Makefile pipeline whose 'finish' target consumes the k and b parameters to orchestrate polishing and gap-filling) — https://github.com/BirolLab/ntedit_sealer_protocol

## Examples

```
ntedit-sealer finish seqs=ecoli_shasta.fa reads='SRR158595208_1.fq.gz SRR158595208_2.fq.gz' k='80 65 50' b=200M
```

## Evaluation signals

- The k variable is passed as a space-separated list in strictly descending order, e.g. k='80 65 50' — an ascending or unsorted list signals incorrect application
- The Bloom filter size b is explicit and scaled to the dataset (200M for the E. coli NDM5 demo; 5G in the generic example), and Bloom filters are actually built by ntHits and ABySS-Bloom at that size
- On success the pipeline emits the Makefile message 'ntEdit and Sealer polishing steps complete!'
- The expected output file <assembly>.ntedit_edited.prepd.sealer_scaffold.fa (e.g. ecoli_shasta.ntedit_edited.prepd.sealer_scaffold.fa) exists in the working directory

## Limitations

- Bloom filters are probabilistic data structures, so parameter choice trades memory footprint against false-positive rates; b must be selected carefully per genome/read set
- k values must be descending for the cascade; incorrect ordering or unsuitable k/b values can compromise error correction and gap-filling quality
- Alignment-free k-mer-based polishing may not catch errors that alignment-based hybrid polishers (Pilon, Racon) would detect

## Evidence

- [abstract] Selecting optimal values for k-mer lengths (k) and Bloom filter size (b): "Selecting optimal values for k-mer lengths (k) and Bloom filter size (b)"
- [methods] methods example invocation with descending k: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"
- [intro] Bloom filter rationale: "This protocol relies on Bloom filters, a probabilistic data structure that stores and tests for elements in a set in constant time (Bloom, 1970)."
- [readme] README description of two-stage finishing: "An automated protocol for finishing long-read genome assemblies using short reads."
