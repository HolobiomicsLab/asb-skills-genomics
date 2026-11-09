---
name: variant-caller-benchmark-evaluation
description: Use when comparing the accuracy of variant callers (e.g., DeepVariant
  vs Clairvoyante) on Illumina or ONT HG001/HG001-like WGS data, with reported metrics
  like precision 0.9995 and ONT F1 scores on chr1/chr22.
when_to_use_negative:
- Input data is from a completely novel sequencing technology/chemistry for which
  no high-quality GIAB-style training dataset exists — Clairvoyante cannot be applied
  in that setting
- The goal is indel calling on ONT data — the paper reports the Indel F1-score remains
  around 50%, so benchmark results will not be representative of production indel
  accuracy
- Variants of interest have two or more alternative alleles — the current Clairvoyante
  design ignores such variants, biasing comparisons
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: DeepVariant
  role: Variant caller under evaluation; v0.6.1 run with transfer learning from pre-trained
    Illumina and chr21 ONT models
- name: Clairvoyante
  role: Baseline deep-neural-network variant caller run on the same Illumina and ONT
    inputs for comparison
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Clair
  role: Successor of Clairvoyante; noted for users needing an updated caller
  repo: https://github.com/HKU-BAL/Clair
- name: Oxford Nanopore
  role: Source of rel5 long-read HG001 sequencing data benchmarked on chr1 and chr22
  repo: https://github.com/nanopore-wgs-consortium/NA12878
- name: Illumina
  role: Source of short-read HG001 sequencing data used for the precision/recall benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:24aca5cf737baceb4c969f056e96de4e6f022acc92e54f8f2ee87deb4677e81f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clair
  - Clairvoyante
  - DeepVariant
  - Illumina
  - Oxford Nanopore
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# variant-caller-benchmark-evaluation

## Summary

Benchmark DeepVariant (with transfer learning) against Clairvoyante and other variant callers on HG001 Illumina and Oxford Nanopore rel5 data, computing precision, recall, and F1-score against the Genome-in-a-Bottle v3.3.2 HG001 truth set.

## When to use

Use when comparing the accuracy of variant callers (e.g., DeepVariant vs Clairvoyante) on Illumina or ONT HG001/HG001-like WGS data, with reported metrics like precision 0.9995 and ONT F1 scores on chr1/chr22.

## When NOT to use

- Input data is from a completely novel sequencing technology/chemistry for which no high-quality GIAB-style training dataset exists — Clairvoyante cannot be applied in that setting
- The goal is indel calling on ONT data — the paper reports the Indel F1-score remains around 50%, so benchmark results will not be representative of production indel accuracy
- Variants of interest have two or more alternative alleles — the current Clairvoyante design ignores such variants, biasing comparisons

## Inputs

- Illumina HG001 BAM alignments
- Oxford Nanopore rel5 HG001 alignments (chr1 and chr22)
- Genome-in-a-Bottle (GIAB) v3.3.2 HG001 truth VCF
- GRCh37 decoy version 5 reference genome
- Pre-trained DeepVariant Illumina model and chr21-trained ONT model for transfer learning

## Outputs

- deepvariant_clairvoyante_benchmark.csv with per-caller, per-dataset/chromosome precision, recall, and F1-score
- Per-sample VCF variant calls from DeepVariant and Clairvoyante

## How to apply

Install DeepVariant v0.6.1 and run it on Illumina HG001 alignments applying transfer learning from the pre-trained Illumina model per the transfer-learning guide, then evaluate calls against the Genome-in-a-Bottle v3.3.2 HG001 truth set to compute precision and recall (expected: 0.9995 and 0.9991). For Oxford Nanopore rel5 data, run DeepVariant v0.6.1 with transfer learning from the chromosome 21-trained model on chromosomes 1 and 22, and run Clairvoyante on the same Illumina and ONT inputs as the baseline. Compute per-benchmark precision, recall, and F1-score for both callers on each dataset/chromosome, record them in a summary CSV, and verify they match reported values (Illumina precision 0.9995, recall 0.9991; ONT F1 83.05% chr1, 77.89% chr22) within rounding tolerance. When preparing Clairvoyante inputs, follow the paper's protocol: re-align PacBio/ONT reads to the GRCh37 decoy version 5 reference using NGMLR v0.2.3, and do not remove alignments by mapping quality.

## Related tools

- **DeepVariant** (Variant caller under evaluation; v0.6.1 run with transfer learning from pre-trained Illumina and chr21 ONT models)
- **Clairvoyante** (Baseline deep-neural-network variant caller run on the same Illumina and ONT inputs for comparison) — https://github.com/aquaskyline/Clairvoyante
- **Clair** (Successor of Clairvoyante; noted for users needing an updated caller) — https://github.com/HKU-BAL/Clair
- **Oxford Nanopore** (Source of rel5 long-read HG001 sequencing data benchmarked on chr1 and chr22) — https://github.com/nanopore-wgs-consortium/NA12878
- **Illumina** (Source of short-read HG001 sequencing data used for the precision/recall benchmark)

## Examples

```
git clone --depth=1 https://github.com/aquaskyline/Clairvoyante.git && cd Clairvoyante && curl http://www.bio8.cs.hku.hk/trainedModels.tbz | tar -jxf - && python callVarBam.py --thread 4 --ref GRCh37.fa --bam HG001_ont.bam --outPath calls --chkfn trainedModels/recv
```

## Evaluation signals

- DeepVariant v0.6.1 on HG001 Illumina reproduces precision 0.9995 and recall 0.9991 against GIAB v3.3.2 within rounding tolerance
- DeepVariant ONT F1-scores match reported values: 83.05% on chr1 and 77.89% on chr22 with the chr21-transferred model
- The summary CSV contains precision, recall, and F1 rows for both callers across all evaluated datasets/chromosomes
- ONT inputs were re-aligned to GRCh37 decoy v5 with NGMLR v0.2.3 and no alignments were removed by mapping quality, per the paper's protocol

## Limitations

- Clairvoyante ONT Indel F1-score remains around 50%, so indel-level comparisons on ONT data are unreliable
- Clairvoyante requires a high-quality GIAB-style training dataset per technology; it cannot be applied to completely novel sequencing technologies or chemistries
- Clairvoyante ignores variants with two or more alternative alleles and only outputs one of two possible alternative alleles per position (network version 3)
- Clairvoyante does not consider base quality of sequencing reads, and provides exact alternative alleles only for indels ≤4 bp
- Only HG001 ONT data was available, limiting cross-sample variant calling evaluation and combined-sample training on ONT

## Evidence

- [other] DeepVariant exceeds Clairvoyante on Illumina HG001: "Clairvoyante is on par with GATK UnifiedGenotyper on Illumina data, and substantially outperforms Nanopolish and DeepVariant on PacBio and ONT data"
- [discussion] GIAB v3.3.2 HG001 truth set used for evaluation: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [discussion] ONT realignment protocol for benchmark inputs: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] No mapping-quality filtering of alignments: "We do not suggest removing any alignment by their mapping quality"
- [discussion] ONT indel F1 limitation: "it still has a large room to be improved in detecting Indels, especially for ONT data, in which the Indel F1-score remains around 50%"
- [readme] Clairvoyante installation and trained models: "git clone --depth=1 https://github.com/aquaskyline/Clairvoyante.git"
- [abstract] Whole-genome F1 targets by technology: "98.65, 92.57, 87.26% F1-score for whole-genome analysis, using Illumina, PacBio, and Oxford Nanopore data, respectively"
