---
name: ngs-read-alignment-bwa-mem
description: Use when starting from raw fastq.gz paired-end sequencing reads of human
  WES/WGS samples that must be aligned to hg19 and processed into analysis-ready BAMs
  for variant calling.
when_to_use_negative:
- Input is already an aligned and processed BAM/VCF — this skill covers raw FASTQ-to-BAM
  alignment only; downstream annotation/score/prioritize modules (eDiVA-Annotate,
  eDiVA-Score, eDiVA-Prioritize) handle pre-called variants
- Reference genome other than hg19 is required, or the sample is non-human — the eDiVA-Predict
  configuration and resources described here are hg19/human-specific
- CNV calling is the goal — the article notes CNV analysis is currently being integrated
  in eDiVA and is not yet available
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: bwa-mem
  role: Aligns reads in fastq format to the hg19 reference genome sample-wise (v0.7.10)
- name: samtools
  role: Post-processing of alignments (v1.3+, 1.5 in README requirements)
- name: Picard
  role: Post-processing of alignments per GATK best practices (v1.119)
- name: GATK
  role: Best-practices post-processing and per-sample variant calling with HaplotypeCaller
    (v3.3+)
- name: eDiVA
  role: Wraps the alignment/calling steps in the eDiVA-Predict NextFlow workflow with
    custom quality filters
  repo: https://github.com/mbosio85/ediva
- name: Nextflow
  role: Workflow engine executing eDiVA-Predict.nf per sample with configurable CPUs
    and work directory
  repo: https://www.nextflow.io/
- name: Fastqc
  role: Produces per-sample quality metrics in the Quality_control output folder (v0.11.5)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:2b573e98f72eee8d77bda423a85c629ef07ff14707f58e6b529f1718f35dd594
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - Fastqc
  - GATK
  - Nextflow
  - Picard
  - bwa-mem
  - eDiVA
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# ngs-read-alignment-bwa-mem

## Summary

Align raw WES/WGS FASTQ reads sample-wise with bwa-mem (v0.7.10) to the hg19 reference and post-process alignments with samtools, Picard, and GATK best practices as implemented in the eDiVA-Predict module, producing per-sample BAM files and variant calls (all_variants.vcf).

## When to use

Use when starting from raw fastq.gz paired-end sequencing reads of human WES/WGS samples that must be aligned to hg19 and processed into analysis-ready BAMs for variant calling.

## When NOT to use

- Input is already an aligned and processed BAM/VCF — this skill covers raw FASTQ-to-BAM alignment only; downstream annotation/score/prioritize modules (eDiVA-Annotate, eDiVA-Score, eDiVA-Prioritize) handle pre-called variants
- Reference genome other than hg19 is required, or the sample is non-human — the eDiVA-Predict configuration and resources described here are hg19/human-specific
- CNV calling is the goal — the article notes CNV analysis is currently being integrated in eDiVA and is not yet available

## Inputs

- Paired-end gzipped FASTQ files (fastq.read1.gz and fastq.read2.gz) of raw WES/WGS reads, e.g. from the 1000 Genomes CEPH trio (NA12878, NA12891, NA12892)
- hg19 reference genome (REF, e.g. hg19.fasta with bwa index)
- EXOME bed file of kit-covered regions (extended 150 bp per side, sorted and merged)
- nextflow.config with tool paths (BWA 0.7.10, GATK 3.3+, Picard 1.119, samtools 1.3+/1.5)
- Sample metadata: NAME and AFFECTED flag (1 affected, 0 non-affected)

## Outputs

- Per-sample BAM file of aligned and filtered reads
- all_variants.vcf with SNPs and INDELs calls from GATK HaplotypeCaller
- sample_info.txt (required for downstream joint multisample calling)
- Quality_control folder with fastqc results and other quality metrics

## How to apply

Launch the eDiVA-Predict NextFlow workflow (eDiVA-Predict.nf) per sample, passing the sample name, the gzipped read1/read2 FASTQ pairs, an AFFECTED flag (1 for affected samples, 0 otherwise), and 2 CPUs per task (the default). Reads in fastq format are aligned using bwa-mem (v0.7.10) to the hg19 reference, then post-processed with samtools (v1.3+/1.5), Picard (v1.119), and GATK (v3.3+) according to the recent GATK best practices, applying the pipeline's custom quality filters. Configure paths in nextflow.config via env variables (REF, BWA, GATK, PICARD, SAMTOOLS, BEDTOOLS, FASTQC, and the EXOME bed file, which the authors extend by 150 bp on each side then sort and merge). Variants are called per sample with GATK HaplotypeCaller, and the pipeline also emits a sample_info.txt (with the AFFECTED column) and a Quality_control folder with fastqc results. Verify outputs by checking that the per-sample BAM of aligned and filtered reads and the all_variants.vcf with SNPs and INDELs calls exist and that quality metrics are present.

## Related tools

- **bwa-mem** (Aligns reads in fastq format to the hg19 reference genome sample-wise (v0.7.10))
- **samtools** (Post-processing of alignments (v1.3+, 1.5 in README requirements))
- **Picard** (Post-processing of alignments per GATK best practices (v1.119))
- **GATK** (Best-practices post-processing and per-sample variant calling with HaplotypeCaller (v3.3+))
- **eDiVA** (Wraps the alignment/calling steps in the eDiVA-Predict NextFlow workflow with custom quality filters) — https://github.com/mbosio85/ediva
- **Nextflow** (Workflow engine executing eDiVA-Predict.nf per sample with configurable CPUs and work directory) — https://www.nextflow.io/
- **Fastqc** (Produces per-sample quality metrics in the Quality_control output folder (v0.11.5))

## Examples

```
nextflow run ~/ediva/edivatools-code/Predict/eDiVA-Predict.nf --NAME NA12878 --READ1 reads.read1.gz --READ2 reads.read2.gz --AFFECTED 1 --CPU 2 --OUTF output_folder/ -w work_folder/
```

## Evaluation signals

- The per-sample output folder contains a BAM of aligned and filtered reads, all_variants.vcf, sample_info.txt, and a Quality_control folder with fastqc results
- all_variants.vcf is valid VCF format with genotype quality and coverage fields for all samples in the trio
- Joint multisample VCF (combined.variants.supplement.vcf) from Step 1.5 contains a variant count consistent with the ~138,705 variants reported for the CEPH trio
- AFFECTED column in the concatenated FAMILY_INFO file is correct: 0 for parents (NA12891, NA12892), 1 for the affected sample if applicable
- Tool versions match the pipeline requirements (bwa 0.7.10, GATK 3.3+, Picard 1.119, samtools 1.3+/1.5)

## Limitations

- CNV analysis is currently being integrated in eDiVA and will be available in the near future, so this skill covers SNV/small indel calling only
- Output VCF file name is always all_variants.vcf, so one folder per sample is recommended to avoid overwriting
- The README-flagged dbNSP/dbSNP resource split must be prepared (bcftools view -v snps/indels with norm) before downstream steps; missing resource paths in nextflow.config will break execution
- Alignment and calling use the paper's custom quality filters and lenient default filter settings (Table S1), which may differ from a user's own GATK defaults

## Evidence

- [methods] The eDiVA-Predict module performs sample-wise variant calling according to the recent GATK (McKenna et al., 2010) best practices: "The eDiVA‐Predict module performs sample‐wise variant calling according to the recent GATK (McKenna et al., 2010) best practices"
- [methods] Reads in fastq format are aligned using bwa-mem (H. Li, 2013): "Reads in fastq format are aligned using bwa‐mem (H. Li, 2013)"
- [readme] README: eDiVA-Predict.nf launch command with NAME, READ1/READ2, AFFECTED and CPU parameters: "nextflow run ~/ediva/edivatools-code/Predict/eDiVA-Predict.nf \
                    --NAME sample_name \
                    --READ1 fastq.read1.gz \
                    --READ2 fastq.read2.gz"
- [readme] README: outputs of Step 1 eDiVA-Predict per sample: "Bam file of aligned and filtered reads
*  all_variants.vcf  with the SNPs and INDELs  calls"
- [readme] README: EXOME bed preparation detail: "We normally extend the factory specifications by 150bp on each side, then we sort and merge the intervals."
