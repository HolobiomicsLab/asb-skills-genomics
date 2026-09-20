---
name: snv-variant-calling-benchmarking
description: Use when comparing single nucleotide variant caller performance on ONT
  long-read data (e.g., Dorado hac/sup-basecalled GIAB trio BAMs or Pbsim3-simulated
  reads) and per-caller precision, recall, and F1-score metrics are needed.
when_to_use_negative:
- Input data are short reads or non-ONT sequencing data, since the benchmarked callers
  (miniSNV's pileup/phasing/consensus pipeline) are optimized for ONT long reads
- The goal is calling indels or structural variants rather than single nucleotide
  variants
- No reference genome or truth set (GIAB or simulated) is available, so caller output
  cannot be scored for precision, recall, or F1
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: miniSNV
  role: Primary SNV caller under benchmark; run with 1000 Genomes index, MAPQ >= 20
    filtering, and default R_alt 0.2 threshold on sorted ONT BAMs
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Comparison caller; run with the latest ONT-trained model downloaded from the
    nanoporetech/rerio clair3_models repository
  repo: https://github.com/nanoporetech/rerio
- name: Longshot
  role: Diploid SNV comparison caller used in simulated and trio (Mendelian consistency)
    benchmarks
- name: Dorado
  role: Basecaller producing the hac/sup BAM inputs used for the benchmark datasets
    (kit v14, R10.4.1 flowcells)
- name: Pbsim3
  role: Generates ONT-like simulated reads at defined depths (e.g., 60x Human chr2,
    30x Mouse chr2) from reference assemblies to create truth-set benchmarks
- name: Minimap2
  role: Aligns simulated and real ONT reads to the reference genome to produce sorted
    BAM inputs for each caller
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:606214771a7ae393889bbe1c563675e9ef5199b87604da4aec1cdfb6e0004d4a
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
  - Dorado
  - Longshot
  - Minimap2
  - Pbsim3
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# snv-variant-calling-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark SNV callers (miniSNV, Clair3, Longshot) on ONT nanopore datasets by running each caller on the same sorted BAMs, evaluating VCFs against GIAB truth sets with v3.3 stratifications, and computing precision, recall, and F1 per sample.

## When to use

Use when comparing single nucleotide variant caller performance on ONT long-read data (e.g., Dorado hac/sup-basecalled GIAB trio BAMs or Pbsim3-simulated reads) and per-caller precision, recall, and F1-score metrics are needed.

## When NOT to use

- Input data are short reads or non-ONT sequencing data, since the benchmarked callers (miniSNV's pileup/phasing/consensus pipeline) are optimized for ONT long reads
- The goal is calling indels or structural variants rather than single nucleotide variants
- No reference genome or truth set (GIAB or simulated) is available, so caller output cannot be scored for precision, recall, or F1

## Inputs

- Sorted, samtools-indexed BAM files of ONT reads aligned to a reference genome (e.g., Dorado hac/sup-basecalled GIAB Ashkenazi Trio BAMs)
- Reference FASTA (samtools-indexed), e.g., GRCh38, GRCm39, TAIR10, or BDGP6
- miniSNV prebuilt index folder of 1000 Genomes common variants (five files)
- Clair3 ONT-trained model from nanoporetech/rerio clair3_models
- GIAB truth VCF sets and GIAB v3.3 stratification BED files
- Simulated truth VCFs (for Pbsim3-derived datasets) and trio VCFs for Mendelian consistency checks

## Outputs

- Per-caller VCF files of SNV calls
- Per-caller x per-sample precision, recall, and F1-score table (e.g., benchmark_table_hac.csv, simulated_benchmark.csv)
- Per-caller Mendelian Consistency Rate table (e.g., mcr_table.csv)

## How to apply

Align ONT reads to GRCh38 (or the appropriate species reference) with Minimap2 and sort/index the BAMs. Run miniSNV with the prebuilt 1000 Genomes common-variants index and default filters (MAPQ >= 20, R_alt threshold 0.2, high-quality locus thresholds R_alt > 0.3 for known variants and > 0.7 with no other candidate within 10 bp); run Clair3 with the latest ONT-trained model from nanoporetech/rerio clair3_models, and run Longshot for diploid SNV calling. For simulated benchmarks, generate ONT-like reads with Pbsim3 v3.0.4 at defined depths (e.g., 60x Human chr2, 30x Mouse chr2, 60x Arabidopsis, 30x Drosophila) and score each caller's VCF against the simulated truth variants. For real benchmarks, evaluate against GIAB truth sets while applying GIAB v3.3 stratifications that exclude difficult-to-map and low-complexity regions, computing precision, recall, and F1 per sample. For trio data, additionally compute the Mendelian Consistency Rate by classifying each child biallelic SNV genotype (0/0, 0/1, 1/1) against parental genotypes.

## Related tools

- **miniSNV** (Primary SNV caller under benchmark; run with 1000 Genomes index, MAPQ >= 20 filtering, and default R_alt 0.2 threshold on sorted ONT BAMs) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Comparison caller; run with the latest ONT-trained model downloaded from the nanoporetech/rerio clair3_models repository) — https://github.com/nanoporetech/rerio
- **Longshot** (Diploid SNV comparison caller used in simulated and trio (Mendelian consistency) benchmarks)
- **Dorado** (Basecaller producing the hac/sup BAM inputs used for the benchmark datasets (kit v14, R10.4.1 flowcells))
- **Pbsim3** (Generates ONT-like simulated reads at defined depths (e.g., 60x Human chr2, 30x Mouse chr2) from reference assemblies to create truth-set benchmarks)
- **Minimap2** (Aligns simulated and real ONT reads to the reference genome to produce sorted BAM inputs for each caller)

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG002_hac.sorted.bam --workDir work_hg002 --useindex -i miniSNV_index --human --sample HG002
```

## Evaluation signals

- Reproduced F1 scores exceed 99.7% for both miniSNV and Clair3 on all three GIAB hac-dataset samples (HG002, HG003, HG004)
- Simulated-data F1 values match reported numbers, e.g., miniSNV F1 ~99.90% on Arabidopsis and ~99.58% on Human chr2 at 60x
- Mendelian Consistency Rates for miniSNV and Longshot exceed Clair3's by roughly 3 percentage points on the sup-dataset trio
- All input BAMs and reference FASTAs are samtools-indexed, and caller VCFs contain only biallelic SNVs when parsed for trio genotyping
- GIAB v3.3 stratifications (difficult-to-map and low-complexity regions) are applied before computing precision, recall, and F1

## Limitations

- Clair3 showed unexpectedly low recalls on non-human simulated species (Mouse, Drosophila, Arabidopsis), suggesting model generalization limits across species
- Rerio basecalling/variant models are research releases not validated or supported by Oxford Nanopore, and rerio Clair3 models are compatible with Clair3 v1.x but not v2.x without conversion
- The R_alt high-quality threshold range of 0.3-0.7 was chosen via in-house experiments, so callers' filter settings are tuning-sensitive

## Evidence

- [methods] R_alt threshold selection: [evidence span withheld — anchor status: not_found]
