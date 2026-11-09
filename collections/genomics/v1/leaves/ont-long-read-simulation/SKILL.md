---
name: ont-long-read-simulation
description: Use when evaluating or comparing SNV/variant callers on ONT-style long
  reads where ground-truth variants are needed, by simulating reads from reference
  assemblies at specified depths.
when_to_use_negative:
- Real ONT sequencing data with GIAB truth sets is already available — simulation
  is unnecessary and less representative
- The analysis goal is caller computational performance (runtime/memory) on real data
  error profiles rather than accuracy against known truth variants
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Pbsim3
  role: Simulates ONT-like sequencing reads from each reference genome at specified
    depths
- name: Minimap2
  role: Aligns each simulated FASTQ to its reference genome to produce sorted BAM
    files
- name: miniSNV
  role: Statistical SNV caller benchmarked on the simulated datasets; best overall
    caller in the benchmark
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Longshot
  role: Pair-HMM-based diploid SNV caller used as a comparison caller on the simulated
    datasets
- name: Clair3
  role: Deep-learning SNV caller using the ONT model from nanoporetech/rerio, run
    as a comparison caller
  repo: https://github.com/nanoporetech/rerio
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:73eee9b66dd2f4233224821158a2ac3fd41e61f7960f1b7f2c6e1b9554e72081
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# ont-long-read-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates ONT-like long-read sequencing datasets with Pbsim3 v3.0.4 from reference genomes at controlled depths (e.g. 60x Human chr2, 30x Mouse chr2) to benchmark SNV callers such as miniSNV, Longshot, and Clair3 against known truth variants.

## When to use

Use when evaluating or comparing SNV/variant callers on ONT-style long reads where ground-truth variants are needed, by simulating reads from reference assemblies at specified depths.

## When NOT to use

- Real ONT sequencing data with GIAB truth sets is already available — simulation is unnecessary and less representative
- The analysis goal is caller computational performance (runtime/memory) on real data error profiles rather than accuracy against known truth variants

## Inputs

- Reference genome FASTA assemblies (e.g. GRCh38 chromosome 2, GRCm39 chromosome 2, TAIR10, BDGP6)
- Target sequencing depth per condition (e.g. 60x, 30x)
- Pbsim3 v3.0.4 with an ONT-like error profile

## Outputs

- Simulated ONT-like FASTQ per species/depth condition
- Sorted BAM files from Minimap2 alignment of simulated reads
- Per-caller VCFs and a caller-by-species F1 table (e.g. simulated_benchmark.csv)

## How to apply

Obtain the public reference assemblies used in the study (GRCh38 chr2, GRCm39 chr2, TAIR10 for Arabidopsis, BDGP6 for Drosophila) and simulate ONT-like reads with Pbsim3 v3.0.4 at the reported depths (60x for Human chr2 and Arabidopsis, 30x for Mouse chr2 and Drosophila), producing one FASTQ per condition. Align each simulated FASTQ to its reference with Minimap2 to produce sorted BAM files. Run the callers under comparison (miniSNV, Longshot, and Clair3 with the ONT model from nanoporetech/rerio clair3_models) on every simulated dataset. The rationale is that simulation provides exact truth variants, so precision, recall, and F1 can be computed per caller per species by comparing output VCFs against the simulated truth. This design also exposes caller-specific failure modes, e.g. Clair3's unexpectedly low recalls on non-human species.

## Related tools

- **Pbsim3** (Simulates ONT-like sequencing reads from each reference genome at specified depths)
- **Minimap2** (Aligns each simulated FASTQ to its reference genome to produce sorted BAM files)
- **miniSNV** (Statistical SNV caller benchmarked on the simulated datasets; best overall caller in the benchmark) — https://github.com/CuiMiao-HIT/miniSNV
- **Longshot** (Pair-HMM-based diploid SNV caller used as a comparison caller on the simulated datasets)
- **Clair3** (Deep-learning SNV caller using the ONT model from nanoporetech/rerio, run as a comparison caller) — https://github.com/nanoporetech/rerio

## Examples

```
pbsim3 --depth 60 --length-min 100 --accuracy-min 0.90 --diffratio 0.08 GRCh38_chr2.fa; minimap2 -ax map-ont ref.fa sim.fastq | samtools sort -o sim.bam; samtools index sim.bam
```

## Evaluation signals

- Reproduced F1 table matches reported simulated-data values, e.g. miniSNV F1 ~99.90% on Arabidopsis (vs 99.56% for the paired caller)
- miniSNV F1 ~99.58% on Human chr2 at 60x
- Caller ranking reproduced: miniSNV best overall, followed by Longshot then Clair3
- Clair3 shows unexpectedly low recalls on the non-human species (Mouse, Drosophila, Arabidopsis)
- Each simulated condition yields one FASTQ and one sorted BAM; BAM is samtools-indexed before caller input

## Limitations

- Simulated reads from Pbsim3 may not capture all real ONT error profiles; results must be corroborated on real data (e.g. GIAB Ashkenazi Trio kit v14 Dorado datasets)
- Clair3's low recall on non-human species suggests DL-based callers generalize poorly across species, so conclusions from simulated benchmarks are species-dependent
- Pbsim3 version matters: the benchmark is reproducible specifically with Pbsim3 v3.0.4

## Evidence

- [results] Benchmark ranking of callers on simulated data: [evidence span withheld — anchor status: not_found]
- [results] Validation threshold for the reproduced table: [evidence span withheld — anchor status: not_found]
