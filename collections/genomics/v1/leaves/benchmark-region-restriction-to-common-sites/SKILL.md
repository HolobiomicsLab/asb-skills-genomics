---
name: benchmark-region-restriction-to-common-sites
description: Use when benchmarking a variant caller on a defined benchmark region
  such as 1KGp3 common variant sites, where whole-genome call sets must be filtered
  to a common-site subset before truth-set comparison.
when_to_use_negative:
- The goal is whole-genome (not common-variant-restricted) performance; the paper
  reports a separate whole-genome F1 (87.26% for ONT) that should not be mixed with
  the 1KP common-variant F1 (90.53% for ONT).
- The truth set or benchmark region is not shared between the callers being compared
  — filtering only one caller's calls to the common sites would bias the comparison.
- The task is de novo discovery of rare or novel variants; restricting to 1KGp3 common
  sites discards exactly those calls.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: Clairvoyante
  role: DNN variant caller whose chr1 call set is restricted to 1KGp3 common sites
    before truth-set comparison
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Nanopolish
  role: Baseline ONT caller run in whole-genome mode; its calls are restricted to
    the same 1KGp3 common sites for the benchmark
- name: NGMLR
  role: Realigns ONT rel3 reads to GRCh37 decoy version 5 (v0.2.3) to produce the
    BAM the calls are made from
- name: Tensorflow
  role: Framework hosting the trained Clairvoyante checkpoint used for calling prior
    to region restriction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:c2758e4ec59445258afc22e4993e7798d11d2a49220d4919f346a9ffdefb93c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - NGMLR
  - Nanopolish
  - Oxford Nanopore
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# benchmark-region-restriction-to-common-sites

## Summary

Restricts variant-caller output (e.g., Clairvoyante and Nanopolish calls on ONT data) to 1KGp3 common variant sites before computing benchmark metrics (F1, FPR, FNR, precision) against the GIAB HG001 v3.3.2 truth set, enabling a like-for-like 'common variants' comparison as in the paper's Table 4.

## When to use

Use when benchmarking a variant caller on a defined benchmark region such as 1KGp3 common variant sites, where whole-genome call sets must be filtered to a common-site subset before truth-set comparison.

## When NOT to use

- The goal is whole-genome (not common-variant-restricted) performance; the paper reports a separate whole-genome F1 (87.26% for ONT) that should not be mixed with the 1KP common-variant F1 (90.53% for ONT).
- The truth set or benchmark region is not shared between the callers being compared — filtering only one caller's calls to the common sites would bias the comparison.
- The task is de novo discovery of rare or novel variants; restricting to 1KGp3 common sites discards exactly those calls.

## Inputs

- Full-chromosome variant call VCF from Clairvoyante (TensorFlow model checkpoint trained on HG001 with chr1 excluded, e.g., 1999 epochs)
- Whole-genome variant call VCF from Nanopolish on the same ONT alignments
- 1KGp3 common variant site list (BED or VCF of common sites)
- GIAB HG001 v3.3.2 truth set (chr1)
- ONI ONT rel3 alignments realigned to GRCh37 decoy v5 with NGMLR v0.2.3

## Outputs

- Common-site-restricted Clairvoyante call set (VCF)
- Common-site-restricted Nanopolish call set (VCF)
- Benchmark metrics per caller (F1, FPR, FNR, precision), e.g., chr1_benchmark_metrics.csv

## How to apply

Run the caller over the full chromosome (e.g., Clairvoyante with a 1999-epoch HG001 model on chr1, and Nanopolish in whole-genome mode on the same NGMLR-realigned ONT BAM), then intersect each VCF with the 1KGp3 common variant site list so that only calls at common sites are retained. Compare the restricted call set against the GIAB HG001 v3.3.2 truth set on the same chromosome, computing F1-score, FPR, FNR, and precision per caller. Restricting both callers to the same site list is essential because callers with different false-positive profiles (e.g., Nanopolish FPR 0.04% vs Clairvoyante FPR 2.96% on whole-genome-style evaluation) are only fairly compared on a shared region. In the paper this yielded Clairvoyante F1 90.53% (FPR 2.96%, FNR 14.78%, precision 96.55%) and Nanopolish F1 86.93% (FPR 0.04%, FNR 21.57%, precision 97.51%) on HG001 chr1 ONT data, written to chr1_benchmark_metrics.csv.

## Related tools

- **Clairvoyante** (DNN variant caller whose chr1 call set is restricted to 1KGp3 common sites before truth-set comparison) — https://github.com/aquaskyline/Clairvoyante
- **Nanopolish** (Baseline ONT caller run in whole-genome mode; its calls are restricted to the same 1KGp3 common sites for the benchmark)
- **NGMLR** (Realigns ONT rel3 reads to GRCh37 decoy version 5 (v0.2.3) to produce the BAM the calls are made from)
- **Tensorflow** (Framework hosting the trained Clairvoyante checkpoint used for calling prior to region restriction)

## Examples

```
aws s3 cp s3://nanopore-human-wgs/rel3-nanopore-wgs-288418386-FAB39088.fastq.gz . && ngmlr -r GRCh37_decoy_v5.fa -q rel3.fastq.gz -o ont_rel3.sam && callVarBamParallel.py --chkfn ckpt/1999 --bamFn ont_rel3.bam --refFn GRCh37_decoy_v5.fa --outFn chr1_clairvoyante.vcf && bedtools intersect -a chr1_clairvoyante.vcf -b 1kgp3_common_sites.bed > chr1_clairvoyante_common.vcf
```

## Evaluation signals

- Reproduced Clairvoyante chr1 ONT metrics at 1KGp3 common sites match F1 90.53%, FPR 2.96%, FNR 14.78%, precision 96.55% (1999-epoch model).
- Reproduced Nanopolish chr1 ONT metrics at the same sites match F1 86.93%, FPR 0.04%, FNR 21.57%, precision 97.51%.
- Restricted call sets contain no positions outside the 1KGp3 common site list (set-difference check against the site BED).
- Metrics are computed against GIAB HG001 v3.3.2 truth on the same chromosome for both callers, and results are serialized to chr1_benchmark_metrics.csv.
- Both callers' call sets were derived from the same NGMLR v0.2.3 realigned GRCh37 decoy v5 BAM, so region restriction — not input differences — drives metric differences.

## Limitations

- Restricting to 1KGp3 common variants inflates apparent accuracy relative to whole-genome analysis (ONT F1 rises from 87.26% to 90.53%); the two numbers are not comparable.
- Clairvoyante's design ignores multi-allelic variants and can only output one of two possible alternative alleles per position, which affects which calls survive at common sites.
- Indel calling at common sites remains weak for ONT data (indel F1-score ~50%), so overall common-site F1 is dominated by SNP performance.

## Evidence

- [abstract] Clairvoyante achieves 99.67, 95.78, and 90.53% F1-score on 1KP common variants using Illumina, PacBio, and Oxford Nanopore data: [evidence span withheld — anchor status: not_found]
- [methods] Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] we used as our truth dataset the latest dataset version 3.3.2 for HG001: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [abstract] 98.65, 92.57, and 87.26% F1-score for whole-genome analysis, using Illumina, PacBio, and Oxford Nanopore data, respectively: [evidence span withheld — anchor status: not_found]
- [discussion] we used the ‘rel3’ release sequenced on the Oxford Nanopore MinION using 1D ligation kits (450 bp/s) and R9.4 chemistry: "we used the ‘rel3’ release sequenced on the Oxford Nanopore MinION using 1D ligation kits (450 bp/s) and R9.4 chemistry"
