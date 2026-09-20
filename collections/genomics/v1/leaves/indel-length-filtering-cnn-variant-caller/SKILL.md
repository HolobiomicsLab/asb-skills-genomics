---
name: indel-length-filtering-cnn-variant-caller
description: Use when emitting a VCF from Clairvoyante variant calling on single-molecule
  sequencing (PacBio or ONT) data and you must remove long-indel predictions that
  the network cannot resolve to an exact alternative allele.
when_to_use_negative:
- You need to detect long indels (>4 bp) — the model was designed to provide exact
  alternative alleles only for indels ≤4 bp, so filtering would silently discard true
  calls rather than correct errors
- Variant calling with tools that resolve full ALT sequences for long indels (e.g.,
  structural variant callers); this filter is specific to Clairvoyante network version
  3's output layers
- Multi-allelic sites or variants with two or more alternative alleles — Clairvoyante
  already excludes multi-allelic SNPs from training and outputs only one of two possible
  alternative alleles per position
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: Clairvoyante
  role: CNN variant caller whose indel-length output drives the ≤4 bp retention filter
    during VCF emission
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Tensorflow
  role: Machine-learning framework on which the Clairvoyante model performing the
    indel-length prediction is built
- name: RTG vcfeval
  role: Benchmark of the filtered VCF against GIAB v3.3.2 HG001 truth data to compute
    FPR, FNR, precision, and F1-score
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:905dfa37aafc4a495791759a420fc2810e46a28ba47a4d7ed18c83cd760cd8cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - PacBio
  - RTG vcfeval
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# indel-length-filtering-cnn-variant-caller

## Summary

Discard CNN-predicted indel calls longer than 4 bp, because Clairvoyante's multi-task network can only output an exact alternative allele for indel variants ≤4 bp due to the scarcity of long indel training samples. This filter keeps the emitted VCF restricted to calls for which the model can supply a precise ALT allele.

## When to use

Use when emitting a VCF from Clairvoyante variant calling on single-molecule sequencing (PacBio or ONT) data and you must remove long-indel predictions that the network cannot resolve to an exact alternative allele.

## When NOT to use

- You need to detect long indels (>4 bp) — the model was designed to provide exact alternative alleles only for indels ≤4 bp, so filtering would silently discard true calls rather than correct errors
- Variant calling with tools that resolve full ALT sequences for long indels (e.g., structural variant callers); this filter is specific to Clairvoyante network version 3's output layers
- Multi-allelic sites or variants with two or more alternative alleles — Clairvoyante already excludes multi-allelic SNPs from training and outputs only one of two possible alternative alleles per position

## Inputs

- Clairvoyante per-site predictions (variant type, zygosity, alternative allele, indel length) from PacBio/ONT BAM tensors
- Candidate variant site list from 1KGp3 common variant sites or genome-wide scan

## Outputs

- Filtered VCF containing only SNPs and indels ≤4 bp with exact alternative alleles
- vcfeval confusion counts convertible to FPR, FNR, precision, F1-score metrics

## How to apply

Run Clairvoyante (Tensorflow-based, five-layer CNN, network version 3) to predict variant type (SNP or indel), zygosity, alternative allele, and indel length for each candidate site. During call-to-VCF conversion, check the predicted indel length at each candidate variant site: retain the record only if the deletion/insertion is ≤4 bp, where the model provides an exact alternative allele. Drop any predicted indel longer than 4 bp — for these the model cannot output a precise ALT, so keeping them would corrupt the VCF and bias downstream benchmarking (e.g., RTG vcfeval against the GIAB v3.3.2 HG001 truth set). Apply the filter symmetrically to training-time candidate generation and calling so metrics such as FPR, FNR, precision, and F1-score remain comparable to published benchmarks.

## Related tools

- **Clairvoyante** (CNN variant caller whose indel-length output drives the ≤4 bp retention filter during VCF emission) — https://github.com/aquaskyline/Clairvoyante
- **Tensorflow** (Machine-learning framework on which the Clairvoyante model performing the indel-length prediction is built)
- **RTG vcfeval** (Benchmark of the filtered VCF against GIAB v3.3.2 HG001 truth data to compute FPR, FNR, precision, and F1-score)

## Examples

```
pypy callVarBam.py --inputBam HG001_pacbio.bam --ref GRCh37_decoy_v5.fa --trainedModel hg001_1499epoch --callVarVcf out.vcf  # then drop records with |len(ref)-len(alt)| > 4
```

## Evaluation signals

- No record in the output VCF has an indel longer than 4 bp
- Every emitted indel record carries an explicit, exact alternative allele sequence
- Recomputing RTG vcfeval metrics on the PacBio HG001 / 1KGp3 common-variant benchmark reproduces FPR 2.17%, FNR 6.06%, precision 97.70%, F1-score 95.78%
- Indel F1-score on ONT data remains around 50% if filtering is applied unchanged to Nanopore data — a large deviation from the paper's reported range suggests the filter or calling setup diverged

## Limitations

- Exact alternative allele is only provided for indel variants ≤4 bp due to the rareness of long indel variants in training data; longer indels cannot be precisely represented
- Multi-allelic SNPs are excluded from training, and the current design ignores variants with two or more alternative alleles
- Base quality of sequencing reads is not considered in the current Clairvoyante implementation
- Indel detection, especially for ONT data where the Indel F1-score remains around 50%, still needs substantial improvement

## Evidence

- [discussion] Exact alternative allele only for indels ≤4 bp: "Due to the rareness of long indel variants for model training, Clairvoyante was set to provide the exact alternative allele only for indel variants ≤4 bp"
- [other] Discarding indel calls longer than 4 bp (workflow): [evidence span withheld — anchor status: not_found]
- [discussion] Multi-allelic SNPs excluded: "In the current Clairvoyante implementation, (1) multi-allelic SNPs are excluded from training"
- [discussion] Indel F1-score low on ONT: "it still has a large room to be improved in detecting Indels, especially for ONT data, in which the Indel F1-score remains around 50%"
- [abstract] PacBio HG001 benchmark metrics: "Clairvoyante achieves 99.67, 95.78, 90.53% F1-score on 1KP common variants, and 98.65, 92.57, 87.26% F1-score for whole-genome analysis"
