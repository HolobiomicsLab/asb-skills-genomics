---
name: f1-score-computation-variant-calling
description: 'Use when evaluating SNV callers (e.g. miniSNV, Longshot, Clair3) on
  real or simulated ONT data: you have caller VCFs and truth variant sets and need
  per-caller, per-condition precision, recall, and F1 values.'
when_to_use_negative:
- The task is variant calling itself rather than benchmarking caller output — use
  the callers' pipelines instead
- You have no truth/reference variant set to compare against, so precision, recall,
  and F1 cannot be defined
- The comparison is structural variant or indel calling; this skill covers single
  nucleotide variant (SNV) benchmarking as described in the article
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: miniSNV
  role: SNV caller under benchmark; its F1, recall, and precision values are compared
    against other callers per species and depth
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Longshot
  role: Baseline diploid SNV caller benchmarked in the F1 comparison
- name: Clair3
  role: Deep-learning SNV caller benchmarked using the ONT model from the nanoporetech/rerio
    clair3_models repository
  repo: https://github.com/nanoporetech/rerio
- name: Pbsim3
  role: Generates the ONT-like simulated reads and truth variants (version 3.0.4)
    that define the denominator for recall in the F1 computation
- name: Minimap2
  role: Aligns simulated FASTQ reads to each reference genome to produce the BAMs
    consumed by all callers before scoring
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:a30a7be339adf401c3cc3aad84b15fb51c2f90809f986a968fb8ff11083bbda5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Clair3
  - Longshot
  - Minimap2
  - Pbsim3
  - miniSNV
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# f1-score-computation-variant-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark SNV caller accuracy by comparing each caller's output VCF against simulated truth variants to compute precision, recall, and F1-score per caller per species/depth, as done for miniSNV, Longshot, and Clair3 on Pbsim3-simulated ONT-like data.

## When to use

Use when evaluating SNV callers (e.g. miniSNV, Longshot, Clair3) on real or simulated ONT data: you have caller VCFs and truth variant sets and need per-caller, per-condition precision, recall, and F1 values.

## When NOT to use

- The task is variant calling itself rather than benchmarking caller output — use the callers' pipelines instead
- You have no truth/reference variant set to compare against, so precision, recall, and F1 cannot be defined
- The comparison is structural variant or indel calling; this skill covers single nucleotide variant (SNV) benchmarking as described in the article

## Inputs

- Caller output VCF files (miniSNV, Longshot, Clair3)
- Simulated truth variant set per condition
- Pbsim3-simulated ONT-like FASTQ files (Human, Mouse, Drosophila, Arabidopsis)
- Reference assemblies: GRCh38 chr2, GRCm39 chr2, TAIR10, BDGP6
- Sorted, indexed BAM alignments from Minimap2

## Outputs

- Per-caller, per-species precision, recall, and F1-score table (e.g. simulated_benchmark.csv)
- Caller ranking (miniSNV > Longshot > Clair3 on the simulated datasets)

## How to apply

Generate truth variants and reads with Pbsim3 v3.0.4 (e.g. 60x for Human chr2 GRCh38 and Arabidopsis TAIR10, 30x for Mouse chr2 GRCm39 and Drosophila BDGP6), align each simulated FASTQ to its reference with Minimap2, then run all callers under test — miniSNV, Longshot, and Clair3 (with the ONT model from nanoporetech/rerio clair3_models) — on every sorted BAM. For each caller-by-species combination, compare the output VCF against the simulated truth variants and compute precision (true positives / called variants), recall (true positives / truth variants), and F1 as the harmonic mean of precision and recall. Record the full caller-by-species F1 table (e.g. to simulated_benchmark.csv) and check it against reported values such as miniSNV F1 ~99.90% on Arabidopsis and ~99.58% on Human chr2 at 60x. The same metric computation applies to real-data benchmarks, e.g. HG002/HG003/HG004 GIAB sup-datasets where miniSNV achieved F1 and recall greater than 99.7%.

## Related tools

- **miniSNV** (SNV caller under benchmark; its F1, recall, and precision values are compared against other callers per species and depth) — https://github.com/CuiMiao-HIT/miniSNV
- **Longshot** (Baseline diploid SNV caller benchmarked in the F1 comparison)
- **Clair3** (Deep-learning SNV caller benchmarked using the ONT model from the nanoporetech/rerio clair3_models repository) — https://github.com/nanoporetech/rerio
- **Pbsim3** (Generates the ONT-like simulated reads and truth variants (version 3.0.4) that define the denominator for recall in the F1 computation)
- **Minimap2** (Aligns simulated FASTQ reads to each reference genome to produce the BAMs consumed by all callers before scoring)

## Evaluation signals

- Reproduced F1 table matches reported simulated values: miniSNV ~99.90% on Arabidopsis and ~99.58% on Human chr2 at 60x
- Caller ordering reproduces miniSNV best overall, followed by Longshot and Clair3 on the simulated datasets
- In the two 60x datasets (Human and Arabidopsis), miniSNV F1 and sensitivity exceed Longshot's
- Clair3 shows unexpectedly low recall on non-human species — a known signature of a correctly reproduced benchmark
- F1 values equal the harmonic mean of independently computed precision and recall per caller-by-species cell

## Limitations

- Clair3 showed unexpectedly low recalls on non-human species in the simulated benchmark, so cross-species generalization of DL-based callers should not be assumed
- Reported F1 values are specific to Pbsim3 v3.0.4 error profiles and the stated depths (60x/30x); other simulators or depths will not necessarily reproduce the numbers
- Clair3 models from nanoporetech/rerio are compatible with Clair3 v1.x but not v2.x without conversion, which affects reproducibility of the caller configuration

## Evidence

- [results] F1 and recall on real data: "miniSNV simultaneously achieved the highest F1-score and recall, all of which were greater than 99.7%, and closer precision on HG002, HG003, and HG004 sup-datasets"
- [intro] F1 as headline metric vs state of the art: "The results demonstrate that miniSNV has higher or competitive F1-scores for SNV calling compared to the state-of-the-art approaches."
- [results] Simulated benchmark setup: "We selected four species, i.e. Human, Mouse, Drosophila, and Arabidopsis, and generated ONT-like datasets using Pbsim3 (version 3.0.4)"
- [readme] Clair3 model source for benchmarking: "Each model will be downloaded to the folder ``clair3_models/{config}``.

All models are compatible with Clair3 v1.x but not v2.x."
