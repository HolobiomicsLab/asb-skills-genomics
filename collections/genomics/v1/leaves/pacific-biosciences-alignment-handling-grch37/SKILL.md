---
name: pacific-biosciences-alignment-handling-grch37
description: 'Use when preparing PacBio (or other long-read) alignments for deep-learning-based
  small variant calling: the raw vendor BAM must be extracted and realigned to GRCh37
  decoy v5 with NGMLR v0.2.3 before tensor generation and calling against GIAB HG001
  v3.3.2 truth.'
when_to_use_negative:
- The input is Illumina short-read data — Clairvoyante's Illumina pipeline does not
  use the NGMLR realignment step, and GATK UnifiedGenotyper is the comparator on Illumina.
- Multi-allelic variants or indels >4 bp must be reported with exact alternative alleles
  — Clairvoyante's network version 3 cannot provide these.
- The task is somatic or trio-based variant calling — these modes are not yet supported
  by Clairvoyante.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NGMLR
  role: Long-read aligner (v0.2.3) used to re-map PacBio reads onto the GRCh37 decoy
    version 5 reference
- name: Clairvoyante
  role: Downstream multi-task five-layer CNN that consumes the realigned BAM to predict
    variant type, zygosity, alternative allele and indel length
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Tensorflow
  role: Machine-learning framework on which Clairvoyante (≥1.0.0 required) runs the
    variant calling over the realigned alignment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:00e8a9f1ea2b0d7af6974b91392a417c8a1f41ea950b667347e7c6e3a8cf8b21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - NGMLR
  - PacBio
  - Tensorflow
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pacific-biosciences-alignment-handling-grch37

## Summary

Re-align long PacBio sequencing reads (e.g. the 44x HG001/NA12878 BAM) to the GRCh37 decoy version 5 reference using NGMLR v0.2.3, producing the alignment that Clairvoyante tensor extraction and variant calling consume for single-molecule sequencing variant calling.

## When to use

Use when preparing PacBio (or other long-read) alignments for deep-learning-based small variant calling: the raw vendor BAM must be extracted and realigned to GRCh37 decoy v5 with NGMLR v0.2.3 before tensor generation and calling against GIAB HG001 v3.3.2 truth.

## When NOT to use

- The input is Illumina short-read data — Clairvoyante's Illumina pipeline does not use the NGMLR realignment step, and GATK UnifiedGenotyper is the comparator on Illumina.
- Multi-allelic variants or indels >4 bp must be reported with exact alternative alleles — Clairvoyante's network version 3 cannot provide these.
- The task is somatic or trio-based variant calling — these modes are not yet supported by Clairvoyante.

## Inputs

- PacBio sequencing alignment (BAM) for HG001/NA12878 at ~44x coverage
- GRCh37 decoy version 5 reference genome
- Genome-in-a-Bottle (GIAB) version 3.3.2 HG001 truth dataset (for downstream benchmarking)
- 1KGp3 common variant site BED/region list (for stratified evaluation)

## Outputs

- BAM of PacBio reads realigned to GRCh37 decoy v5 with NGMLR v0.2.3
- Candidate variant tensors from the realigned alignment
- Called VCF (SNP/indel type, zygosity, alternative allele) restricted to 1KGp3 common variant sites
- vcfeval_performance_metrics.tsv (FPR, FNR, precision, F1-score)

## How to apply

Extract reads from the downloaded PacBio alignment and align them again to the GRCh37 decoy version 5 reference using NGMLR version 0.2.3, since the paper found NGMLR handles the indel-enriched error profile of single-molecule sequencing better than the original mapping. Do not remove any alignment by mapping quality — the Clairvoyante authors explicitly advise keeping all alignments regardless of MAPQ. Feed the realigned BAM into the Clairvoyante data-prep scripts (ExtractVariantCandidates.py, CreateTensor.py, ideally under PyPy for 5–10x speedup) and then callVarBam.py, which uses 4 threads by default and Tensorflow ≥ 1.0.0 for prediction. Discard predicted indel calls longer than 4 bp, for which the model cannot supply an exact alternative allele, before writing the VCF for RTG vcfeval benchmarking against the GIAB v3.3.2 HG001 truth set.

## Related tools

- **NGMLR** (Long-read aligner (v0.2.3) used to re-map PacBio reads onto the GRCh37 decoy version 5 reference)
- **Clairvoyante** (Downstream multi-task five-layer CNN that consumes the realigned BAM to predict variant type, zygosity, alternative allele and indel length) — https://github.com/aquaskyline/Clairvoyante
- **Tensorflow** (Machine-learning framework on which Clairvoyante (≥1.0.0 required) runs the variant calling over the realigned alignment)

## Evaluation signals

- The realignment must use NGMLR version 0.2.3 against GRCh37 decoy version 5 — a different reference build or aligner version changes tensor content and downstream metrics.
- No alignment filtering by mapping quality should be applied between realignment and tensor extraction.
- Indel calls >4 bp must be absent from the final VCF (no exact alternative allele available).
- RTG vcfeval against GIAB v3.3.2 HG001, stratified to 1KGp3 common variant regions, should reproduce FPR 2.17%, FNR 6.06%, precision 97.70% and F1-score 95.78% (HG001-trained 1499-epoch model).
- Whole-genome calling on the 28-core machine should complete in under 2 hours; callVarBam.py runs with 4 threads by default and PyPy is required for acceptable speed.

## Limitations

- Clairvoyante does not consider the base quality of the sequencing reads, so alignment choices and error profile dominate performance.
- Exact alternative alleles are only provided for indel variants ≤4 bp; longer indel predictions lack an exact allele.
- PacBio- and ONT-specific high-confidence region lists were not yet produced, complicating stratified benchmarking.
- blosc installation may fail without AVX2 support; PyPy is incompatible with the Cython-optimized Tensorflow modules.

## Evidence

- [discussion] Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] We do not suggest removing any alignment by their mapping quality: "We do not suggest removing any alignment by their mapping quality"
- [readme] To guarantee a good user experience (good speed), pypy must be installed to run `callVarBam.py` (call variants from BAM): "To guarantee a good user experience (good speed), pypy must be installed to run `callVarBam.py` (call variants from BAM)"
- [intro] Clairvoyante is on par with GATK UnifiedGenotyper on Illumina data, and substantially outperforms Nanopolish and DeepVariant on PacBio and ONT data: "Clairvoyante is on par with GATK UnifiedGenotyper on Illumina data, and substantially outperforms Nanopolish and DeepVariant on PacBio and ONT data on accuracy and speed"
- [discussion] Due to the rareness of long indel variants for model training, Clairvoyante was set to provide the exact alternative allele only for indel variants ≤4 bp: "Due to the rareness of long indel variants for model training, Clairvoyante was set to provide the exact alternative allele only for indel variants ≤4 bp"
