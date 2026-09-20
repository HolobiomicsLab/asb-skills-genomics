---
name: homozygous-discovery-rate-analysis
description: Use when benchmarking SNV-caller sensitivity on ONT Dorado sup-basecalled
  data, specifically when computing homozygous discovery rate (HDR) against GIAB truth
  sets for HG002/HG003/HG004 and comparing miniSNV with Clair3 and Longshot.
when_to_use_negative:
- The goal is indel or structural variant calling — miniSNV and this benchmark target
  single nucleotide variants (SNVs) only
- Input data is short-read or PacBio — the benchmark protocol and caller models (rerio
  r1041 ONT model, Longshot diploid model) are specific to ONT long reads
- Samples are not diploid (e.g. haploid or polyploid) — the HDR metric and Longshot's
  pair hidden Markov model assume diploid genotypes
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: miniSNV
  role: Primary SNV caller leveraging known common variants, read pileup, read-based
    phasing, and consensus generation on ONT long reads
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Deep-learning comparison caller (v1.0.4) run with the ONT-trained model from
    the rerio clair3_models repository
  repo: https://github.com/nanoporetech/rerio
- name: Longshot
  role: Diploid SNV comparison caller (v0.4.5) using a pair hidden Markov model
- name: Oxford Nanopore Technologies (ONT)
  role: Source of the long-read sequencing data (kit v14, R10.4.1, Dorado sup-basecalled)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:40c3f5d265a3f901142f26a7e99176a93ea576834165fdd413cb5b18d6ce9047
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
  - Oxford Nanopore Technologies (ONT)
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# homozygous-discovery-rate-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the miniSNV paper's sup-dataset benchmark on the GIAB Ashkenazi trio by calling SNVs with miniSNV, Clair3 (v1.0.4, rerio ONT model), and Longshot (0.4.5) and computing per-caller homozygous discovery rate (HDR) for the paternal sample HG003.

## When to use

Use when benchmarking SNV-caller sensitivity on ONT Dorado sup-basecalled data, specifically when computing homozygous discovery rate (HDR) against GIAB truth sets for HG002/HG003/HG004 and comparing miniSNV with Clair3 and Longshot.

## When NOT to use

- The goal is indel or structural variant calling — miniSNV and this benchmark target single nucleotide variants (SNVs) only
- Input data is short-read or PacBio — the benchmark protocol and caller models (rerio r1041 ONT model, Longshot diploid model) are specific to ONT long reads
- Samples are not diploid (e.g. haploid or polyploid) — the HDR metric and Longshot's pair hidden Markov model assume diploid genotypes

## Inputs

- Sorted, samtools-indexed BAM files of GIAB Ashkenazi trio ONT reads (HG002, HG003, HG004; kit v14, R10.4.1, Dorado sup-basecalled) aligned to GRCh38 (GCA_000001405.15)
- GRCh38 reference FASTA (samtools indexed)
- miniSNV prebuilt 1000 Genomes known-common-variants index (folder with five files)
- Clair3 ONT-trained model from nanoporetech/rerio clair3_models (e.g. r1041_e82_400bps_sup_v410)
- GIAB truth VCFs and GIAB v3.3 stratification regions

## Outputs

- Per-sample SNV VCFs from miniSNV, Clair3 1.0.4, and Longshot 0.4.5
- Per-caller x per-sample precision, recall, and F1 metrics computed against GIAB truth within stratification regions
- Per-caller homozygous discovery rate (HDR) for HG003, assembled into benchmark_table_sup.csv

## How to apply

Download the GIAB AshkenazimTrio ONT deposits (kit v14 chemistry, R10.4.1 flowcells, Dorado sup-basecalled, ~200 Gbases per sample) and map reads to GRCh38 (GCA_000001405.15) into samtools-indexed sorted BAMs. Run miniSNV per sample using its prebuilt 1000 Genomes known-common-variants index (window length L=300 bp, half-length overlaps, >5% allele frequency), run Clair3 1.0.4 with the ONT-trained model from nanoporetech/rerio clair3_models, and run Longshot 0.4.5 for diploid SNV calls. Compare each caller's VCF against GIAB truth VCFs within GIAB stratification regions to compute precision, recall, and F1 per caller per sample, then compute HDR per caller for HG003. The reproduction is validated when miniSNV's F1 and recall exceed 99.7% on all three sup-datasets and its HG003 HDR exceeds Clair3 by ~1.8% and Longshot by ~1.6%.

## Related tools

- **miniSNV** (Primary SNV caller leveraging known common variants, read pileup, read-based phasing, and consensus generation on ONT long reads) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Deep-learning comparison caller (v1.0.4) run with the ONT-trained model from the rerio clair3_models repository) — https://github.com/nanoporetech/rerio
- **Longshot** (Diploid SNV comparison caller (v0.4.5) using a pair hidden Markov model)
- **Oxford Nanopore Technologies (ONT)** (Source of the long-read sequencing data (kit v14, R10.4.1, Dorado sup-basecalled))

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG003.sorted.bam --workDir ./hg003_miniSNV --useindex -i miniSNV_index --human --sample HG003
```

## Evaluation signals

- miniSNV F1-score and recall are each greater than 99.7% on HG002, HG003, and HG004 sup-datasets
- Reproduced HG003 HDR advantage is ~1.8% over Clair3 and ~1.6% over Longshot, matching reported values
- Per-caller metrics are computed only within GIAB stratification regions against GIAB truth VCFs, not genome-wide
- miniSNV is run with the prebuilt index flag (--useindex) and the index folder contains exactly five files
- All input BAMs are samtools indexed before caller execution

## Limitations

- HDR results are specific to Dorado sup-basecalled kit v14 R10.4.1 GIAB trio data; other error profiles or chemistries are not covered by this benchmark
- miniSNV's known-variant background relies on population VCF input (1000 Genomes by default); performance on populations poorly represented in known-variant panels is not established here
- Haplotype-based filtering (preGT '0/1' with hapGT '1/1' variants are filtered out) can remove true variants where phasing is confounded by systematic errors

## Evidence

- [results] Benchmark dataset description: [evidence span withheld — anchor status: not_found]
