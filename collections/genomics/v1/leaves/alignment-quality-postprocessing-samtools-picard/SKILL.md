---
name: alignment-quality-postprocessing-samtools-picard
description: Use when raw FASTQ WES/WGS reads must be converted to quality-filtered,
  analysis-ready BAM files as a prerequisite for per-sample variant calling with GATK
  HaplotypeCaller.
when_to_use_negative:
- Input is already an aligned, processed BAM or a variant VCF — this skill covers
  FASTQ-to-BAM alignment and post-processing only
- CNV analysis is required — CNV analysis is not yet available in eDiVA and is currently
  being integrated
- Samples are non-human WGS without an exome kit BED — the EXOME parameter assumes
  exome capture region definitions
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: bwa-mem
  role: Aligns reads in FASTQ format to the hg19 reference genome (v0.7.10)
  repo: http://bio-bwa.sourceforge.net/
- name: samtools
  role: Post-processes alignments (v1.3+/v1.5) as part of GATK best-practice processing
  repo: http://samtools.sourceforge.net/
- name: Picard
  role: Post-processes alignments (v1.119) as part of GATK best-practice processing
  repo: https://broadinstitute.github.io/picard/
- name: GATK
  role: Best-practice processing and per-sample variant calling with HaplotypeCaller
    (v3.3+)
  repo: https://software.broadinstitute.org/gatk/
- name: FastQC
  role: Produces quality metrics in the per-sample Quality_control folder (v0.11.5)
  repo: http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- name: Nextflow
  role: Workflow engine running eDiVA-Predict.nf per sample
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:35bc2489947fc32b2980be3173ad196d9d6bb2c9cb24c211252af00b1c46223a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastQC
  - GATK
  - Nextflow
  - Picard
  - bwa-mem
  - eDiVA
  - samtools
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# Alignment quality post-processing with samtools and Picard (GATK best practices)

## Summary

Post-processes bwa-mem alignments of WES/WGS FASTQ reads into analysis-ready BAMs using samtools, Picard, and GATK per GATK best practices, as implemented in the eDiVA-Predict module prior to HaplotypeCaller variant calling.

## When to use

Use when raw FASTQ WES/WGS reads must be converted to quality-filtered, analysis-ready BAM files as a prerequisite for per-sample variant calling with GATK HaplotypeCaller.

## When NOT to use

- Input is already an aligned, processed BAM or a variant VCF — this skill covers FASTQ-to-BAM alignment and post-processing only
- CNV analysis is required — CNV analysis is not yet available in eDiVA and is currently being integrated
- Samples are non-human WGS without an exome kit BED — the EXOME parameter assumes exome capture region definitions

## Inputs

- Paired-end raw sequencing reads in FASTQ format (gzipped read1/read2 files)
- hg19 reference genome (bwa-indexed FASTA)
- Exome capture regions BED file (EXOME parameter)
- dbSNP VCFs split by SNPs and indels for GATK processing

## Outputs

- BAM file of aligned and quality-filtered reads per sample
- Per-sample all_variants.vcf with SNPs and INDELs from GATK HaplotypeCaller
- sample_info.txt per sample (includes AFFECTED column; required for joint multisample calling)
- Quality_control folder with fastqc results and other quality metrics

## How to apply

Align raw paired-end FASTQ reads sample-wise against the hg19 reference using bwa-mem (v0.7.10), then post-process the alignments with samtools (v1.3+/v1.5), Picard (v1.119), and GATK (v3.3+) following GATK best practices while applying the pipeline's custom quality filters. Run the eDiVA-Predict NextFlow workflow per sample, assigning 2 CPUs per task and setting the AFFECTED flag (1 for affected individuals, default 0). Constrain processing to the exome capture regions by supplying a BED file (EXOME parameter), which the authors generate by extending factory kit specifications 150 bp on each side, then sorting and merging intervals. The step yields, per sample, an aligned and filtered BAM, the all_variants.vcf callset from GATK HaplotypeCaller, a sample_info.txt for downstream joint calling, and a Quality_control folder with fastqc results and other quality metrics.

## Related tools

- **bwa-mem** (Aligns reads in FASTQ format to the hg19 reference genome (v0.7.10)) — http://bio-bwa.sourceforge.net/
- **samtools** (Post-processes alignments (v1.3+/v1.5) as part of GATK best-practice processing) — http://samtools.sourceforge.net/
- **Picard** (Post-processes alignments (v1.119) as part of GATK best-practice processing) — https://broadinstitute.github.io/picard/
- **GATK** (Best-practice processing and per-sample variant calling with HaplotypeCaller (v3.3+)) — https://software.broadinstitute.org/gatk/
- **FastQC** (Produces quality metrics in the per-sample Quality_control folder (v0.11.5)) — http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- **Nextflow** (Workflow engine running eDiVA-Predict.nf per sample) — https://www.nextflow.io/

## Examples

```
nextflow run ~/ediva/edivatools-code/Predict/eDiVA-Predict.nf --NAME sample_name --READ1 fastq.read1.gz --READ2 fastq.read2.gz --AFFECTED 1 --CPU 2 --OUTF output_folder/ -w work_folder/
```

## Evaluation signals

- Each sample produces an output folder containing an aligned and filtered BAM, all_variants.vcf, sample_info.txt, and a Quality_control folder
- sample_info.txt AFFECTED column is correct: 0 for non-affected samples, 1 for affected ones
- Per-sample VCF genotype counts are consistent with the ~138,705 variants reported for the CEPH trio (NA12878, NA12891, NA12892)
- NextFlow work directory (-w) is specified to enable resuming of interrupted jobs and manual cleanup
- Quality_control folder contains fastqc results and other per-sample quality metrics

## Limitations

- Sample-wise processing only — joint multisample calling is a separate later step (eDiVA_prepare_for_annotation.nf)
- AFFECTED defaults to 0 and must be verified in sample_info.txt before downstream joint calling
- Output VCF always has the same name 'all_variants.vcf', so one folder per sample is recommended
- CNV analysis is not available in eDiVA; the module addresses SNVs and small indels

## Evidence

- [methods] eDiVA-Predict performs sample-wise variant calling following the recent GATK best practices: "The eDiVA‐Predict module performs sample‐wise variant calling according to the recent GATK (McKenna et al., 2010) best practices"
- [methods] Reads in FASTQ format aligned with bwa-mem: "Reads in fastq format are aligned using bwa‐mem (H. Li, 2013)"
- [intro] eDiVA-Predict processes sequencing results to predict genomic variants: "(a) eDiVA‐Predict, where sequencing results are processed to predict the presence of genomic variants"
- [readme] Per-sample outputs include BAM, all_variants.vcf, sample_info.txt, and Quality_control folder: "Bam file of aligned and filtered reads, all_variants.vcf with the SNPs and INDELs calls, sample_info.txt [required for the next steps], Quality_control folder"
- [readme] Exome BED generation by extending kit intervals 150 bp each side: "We normally extend the factory specifications by 150bp on each side, then we sort and merge the intervals."
- [readme] eDiVA-Predict launched per sample via nextflow with 2 CPUs and AFFECTED flag: "nextflow run ~/ediva/edivatools-code/Predict/eDiVA-Predict.nf --NAME sample_name --READ1 fastq.read1.gz --READ2 fastq.read2.gz --AFFECTED 1 --CPU 2"
