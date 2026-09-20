---
name: small-variant-calling-clair3-nanopore-bams
description: Use when aligned nanopore BAMs (Dorado-basecalled simulated or experimental
  reads) need SNP/small-indel calling and F1 benchmarking against GIAB v3.3.2 truth
  sets.
when_to_use_negative:
- The goal is structural variant calling from DNA BAMs — use Sniffles2 and benchmark
  with Truvari instead
- Input is raw POD5 signal rather than basecalled, aligned BAMs — basecall and align
  first
- The sequencing chemistry is not ONT R10.4.1 DNA or RNA004 direct-RNA, since Clair3
  model selection and the reported F1 ranges are chemistry-specific
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: Clair3
  role: Small-variant (SNP and small indel) caller run on aligned DNA R10.4.1 sample
    BAMs
- name: Clair3-RNA
  role: Variant caller run on the RNA004 direct-RNA sample BAM
- name: RTG tools
  role: vcfeval benchmarking of called SNP/indel VCFs against GIAB v3.3.2 truth sets
- name: Dorado
  role: Basecalls simulated signals into FASTQ/BAM that are aligned and passed to
    the variant caller
  repo: github.com/nanoporetech/dorado
- name: NanoSimFormer
  role: Produces simulated POD5 signals with ont_r1041_dna_5khz or ont_rna004_4khz
    presets that feed the calling pipeline
  repo: https://github.com/BioinfoSZU/NanoSimFormer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:952c8861a39f61fa703afaeba6d8d8243892d36fea5b19ac991c9c84db791193
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clair3
  - Clair3-RNA
  - Dorado
  - NanoSimFormer
  - RTG tools
  - Squigulator
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Small variant calling with Clair3 on nanopore BAMs

## Summary

Run Clair3 (DNA R10.4.1) and Clair3-RNA (RNA004) on aligned Dorado-basecalled nanopore BAMs to call SNPs and small indels, then benchmark the resulting VCFs against GIAB truth sets with RTG tools vcfeval to compute precision, recall and F1 per sample and variant class.

## When to use

Use when aligned nanopore BAMs (Dorado-basecalled simulated or experimental reads) need SNP/small-indel calling and F1 benchmarking against GIAB v3.3.2 truth sets.

## When NOT to use

- The goal is structural variant calling from DNA BAMs — use Sniffles2 and benchmark with Truvari instead
- Input is raw POD5 signal rather than basecalled, aligned BAMs — basecall and align first
- The sequencing chemistry is not ONT R10.4.1 DNA or RNA004 direct-RNA, since Clair3 model selection and the reported F1 ranges are chemistry-specific

## Inputs

- Aligned BAM of Dorado-basecalled nanopore DNA R10.4.1 reads (e.g., from NanoSimFormer --basecall --emit-bam)
- Aligned BAM of Dorado-basecalled direct-RNA (RNA004) reads
- Reference genome/transcriptome FASTA used for simulation and alignment
- GIAB v3.3.2 truth-set VCFs for the corresponding samples

## Outputs

- SNP and small-indel VCF calls per sample
- Precision, recall and F1-score table per sample and variant class (variant_calling_f1_scores.csv)

## How to apply

Align basecalled reads (FASTQ/BAM produced via Dorado basecalling of NanoSimFormer-simulated POD5 signals) to their reference genomes, then run Clair3 on the DNA R10.4.1 sample BAMs and Clair3-RNA on the RNA004 direct-RNA sample BAM. Benchmark the called SNP/indel VCFs against GIAB v3.3.2 truth sets using RTG tools vcfeval, and compute precision, recall and F1 per sample and variant class. For NanoSimFormer-simulated reads, healthy F1 ranges are DNA SNP 0.9953–0.9973 and small indel 0.7862–0.8612 across HG001–HG005, with RNA004 SNP F1 ~0.92 and indel F1 ~0.57; write results to variant_calling_f1_scores.csv.

## Related tools

- **Clair3** (Small-variant (SNP and small indel) caller run on aligned DNA R10.4.1 sample BAMs)
- **Clair3-RNA** (Variant caller run on the RNA004 direct-RNA sample BAM)
- **RTG tools** (vcfeval benchmarking of called SNP/indel VCFs against GIAB v3.3.2 truth sets)
- **Dorado** (Basecalls simulated signals into FASTQ/BAM that are aligned and passed to the variant caller) — github.com/nanoporetech/dorado
- **NanoSimFormer** (Produces simulated POD5 signals with ont_r1041_dna_5khz or ont_rna004_4khz presets that feed the calling pipeline) — https://github.com/BioinfoSZU/NanoSimFormer

## Examples

```
python -m nano_signal_simulator simulate --input chr22.fasta --output out --mode Reference --preset ont_r1041_dna_5khz --basecall --emit-bam --seed 42; clair3 --bam sample.bam --ref ref.fasta --outvcf sample.vcf; rtg vcfeval -b truth.vcf -c calls.vcf -t ref.sdf
```

## Evaluation signals

- DNA SNP F1-scores fall within 0.9953–0.9973 across the five human HG001–HG005 datasets
- Small indel F1-scores fall within 0.7862–0.8612 for NanoSimFormer-simulated reads, comparable to experimental baselines of 0.8209–0.8963
- RNA004 direct-RNA calling reaches approximately 0.92 SNP F1 and 0.57 indel F1
- Baseline separation: seq2squiggle indel F1 lands at 0.4761–0.5135 and Squigulator at 0.1960–0.2419, far below NanoSimFormer
- All per-sample, per-class precision/recall/F1 rows are written to variant_calling_f1_scores.csv

## Limitations

- Small-indel F1 (~0.79–0.86) remains notably below SNP F1 (~0.995+), reflecting residual indel error even with high-fidelity simulated signals
- F1 validity ranges are defined only for the five human R10.4.1 datasets and the HG002 RNA004 sample; other genomes or chemistries are unvalidated
- Direct-RNA small-indel detection is weak (F1 ~0.57), so RNA-derived indel calls should be interpreted cautiously

## Evidence

- [methods] Clair3 on DNA BAMs vs Clair3-RNA: [evidence span withheld — anchor status: not_found]
- [methods] vcfeval benchmarking: [evidence span withheld — anchor status: not_found]
- [abstract] reported F1 ranges: "It faithfully recapitulates experimental variant calling performance across the five human samples, achieving F1-scores of 0.9953–0.9973 and 0.7862–0.8612 for single-nucleotide polymorphisms and"
- [results] baseline comparator scores: [evidence span withheld — anchor status: not_found]
- [readme] basecalling of simulated reads: [evidence span withheld — anchor status: not_found]
