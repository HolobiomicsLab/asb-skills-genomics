---
name: genome-assembly-finishing-parameter-selection
description: 'Use when configuring the ntEdit+Sealer Makefile pipeline: ntCard k-mer
  frequency histograms are available for the protocol''s k-mer sizes (e.g. k=80, k=65,
  k=50) and a Bloom filter size (b) must be chosen for ABySS-Bloom.'
when_to_use_negative:
- No short-read data or ntCard histograms are available — n cannot be estimated and
  the Bloom filter would be sized arbitrarily
- The task is hybrid polishing with alignment-based tools (e.g. Pilon, Racon) where
  memory-intensive read alignments, not Bloom filters, are used
- A draft long-read assembly for gap filling/polishing does not exist, since k and
  b are parameters of the ntEdit+Sealer finishing pipeline specifically
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntCard
  role: Produces the k-mer frequency count histograms from short reads used to estimate
    n per k-mer size
  repo: https://github.com/BirolLab/ntCard
- name: ntHits
  role: Required for creating Bloom filters from short-read k-mers in the ntEdit+Sealer
    protocol
  repo: https://github.com/bcgsc/nthits
- name: ABySS-Bloom
  role: Consumes the computed b parameter (suffix notation, e.g. 200M) to build the
    Bloom filter used by Sealer for gap filling
  repo: https://github.com/bcgsc/abyss
- name: ntEdit+Sealer
  role: The Makefile pipeline whose k and b parameters are selected by this skill
  repo: https://github.com/bcgsc/ntedit_sealer_protocol
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:2bdf3e87a2f679f25ad5fb968fffc3b0e453a3a3862bb4d2938ff65c16dec72c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/ntCard
  tools:
  - ABySS-Bloom
  - ntCard
  - ntEdit
  - ntEdit+Sealer
  - ntHits
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# genome-assembly-finishing-parameter-selection

## Summary

Selects optimal k-mer lengths (k) and Bloom filter size (b) for the ntEdit+Sealer alignment-free finishing protocol by computing k-mer cardinality from ntCard histograms and applying the Bloom filter sizing formula m = Ceil(−n/ln(1−f)) at a target false-positive rate (f ≈ 0.005).

## When to use

Use when configuring the ntEdit+Sealer Makefile pipeline: ntCard k-mer frequency histograms are available for the protocol's k-mer sizes (e.g. k=80, k=65, k=50) and a Bloom filter size (b) must be chosen for ABySS-Bloom.

## When NOT to use

- No short-read data or ntCard histograms are available — n cannot be estimated and the Bloom filter would be sized arbitrarily
- The task is hybrid polishing with alignment-based tools (e.g. Pilon, Racon) where memory-intensive read alignments, not Bloom filters, are used
- A draft long-read assembly for gap filling/polishing does not exist, since k and b are parameters of the ntEdit+Sealer finishing pipeline specifically

## Inputs

- ntCard k-mer frequency count histograms (.hist) for each protocol k-mer size (k=80, k=65, k=50)
- Short-read sequencing data (e.g. paired FASTQ reads_1.fq.gz / reads_2.fq.gz) from which histograms are derived
- Target false-positive rate (f ≈ 0.005)

## Outputs

- bloom_filter_size.txt recording the computed b parameter (e.g. b=200M) for the ntEdit-Sealer Makefile pipeline
- Per-k distinct k-mer cardinalities (n = F0 − singletons) used for the sizing decision

## How to apply

Load the ntCard k-mer count histograms produced for each k-mer size used by the protocol (here k=80, k=65, k=50). For each k, compute n as F0 (the estimated total distinct k-mers) minus the number of k-mers appearing exactly once, removing erroneous singleton k-mers that should not be stored in the filter. Confirm which k yields the largest n — with the E. coli NDM5 dataset (SRA SAMN21398207) k=65 gave the largest n (6,720,950 k-mers appearing ≥2 times), consistent with the protocol's descending k-mer ordering. Apply m = Ceil(−n/ln(1−f)) with f = 0.005 to get m = 1,340,826,718 bits (~168 MB), then round up to a practical memory setting accepted by ABySS-Bloom suffix notation (b=200M) and write it to bloom_filter_size.txt for use as the b parameter alongside k='80 65 50' in the Makefile invocation.

## Related tools

- **ntCard** (Produces the k-mer frequency count histograms from short reads used to estimate n per k-mer size) — https://github.com/BirolLab/ntCard
- **ntHits** (Required for creating Bloom filters from short-read k-mers in the ntEdit+Sealer protocol) — https://github.com/bcgsc/nthits
- **ABySS-Bloom** (Consumes the computed b parameter (suffix notation, e.g. 200M) to build the Bloom filter used by Sealer for gap filling) — https://github.com/bcgsc/abyss
- **ntEdit+Sealer** (The Makefile pipeline whose k and b parameters are selected by this skill) — https://github.com/bcgsc/ntedit_sealer_protocol

## Examples

```
ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=200M
```

## Evaluation signals

- n computed per k matches F0 minus the count of k-mers appearing exactly once in the .hist file (singletons excluded as erroneous)
- The k yielding the largest n (k=65 here, n=6,720,950) is used for sizing, consistent with the protocol's descending k-mer ordering
- m = Ceil(−n/ln(1−0.005)) reproduces 1,340,826,718 bits (~168 MB) for n=6,720,950
- The rounded value written to bloom_filter_size.txt is in ABySS-Bloom suffix notation (e.g. b=200M) and is ≥ the computed m
- The Makefile invocation accepts the parameters, e.g. k='80 65 50' b=200M, without error

## Limitations

- FPR 0.005 was chosen for this E. coli dataset; larger genomes may require revisiting f and the rounding-up margin (~168 MB computed vs 200 MB used)
- Singleton removal assumes low-frequency k-mers are erroneous; real biological singletons (genuine novel sequence) would be excluded from the filter
- Sizing depends on the k-mer sizes tested (80, 65, 50); a different k set changes which histogram yields the largest n

## Evidence

- [abstract] Selecting optimal values for k-mer lengths (k) and Bloom filter size (b): "Selecting optimal values for k-mer lengths (k) and Bloom filter size (b)"
- [intro] ntHits and ABySS-Bloom are required for creating Bloom filters from short read k-mers: "Additionally, ntHits (Mohamadi, Chu, Coombe, Warren, & Birol, 2020) and ABySS-Bloom (Jackman et al., 2017) are required for creating Bloom filters from short read k-mers"
- [methods] Example pipeline invocation with k and b parameters: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"
- [readme] ntHits description from README: [evidence span withheld — anchor status: external_doc]
