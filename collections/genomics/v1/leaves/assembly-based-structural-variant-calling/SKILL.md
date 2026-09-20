---
name: assembly-based-structural-variant-calling
description: Use when long-read (HiFi/CLR/ONT) BAMs must be assessed for SV detection
  quality versus assembly-based baselines, or when false discovery of SVs against
  an alternate reference (T2T-CHM13) must be quantified.
when_to_use_negative:
- The input is a CHM13 sample for VolcanoSV-asm haplotype phasing, since the sample
  is homozygous and unsuitable for phasing
- Detection of complex SVs (translocations, duplications, inversions) on single-chromosome
  data, since translocation detection requires a WGS BAM as support
- Genotyping reciprocal translocations, where genotyping accuracy is ambiguous and
  only F1/GT_F1 should be calculated
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: VolcanoSV
  role: Hybrid SV detection pipeline combining reference-guided and local de novo
    assembly; run per chromosome with platform-appropriate -dtype
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: Dipcall
  role: Assembly-based baseline SV caller (v0.3) for DEL, INS, small indel comparison
  repo: https://github.com/lh3/dipcall
- name: SVIM-asm
  role: Assembly-based baseline caller (v1.0.2) for DEL, INS, DUP, INV, TRA comparison
  repo: https://github.com/eldariont/svim-asm
- name: PAV
  role: Assembly-based baseline caller (v2.0.0) for DEL, INS, INV, small indel comparison
  repo: https://github.com/EichlerLab/pav
- name: Truvari
  role: Benchmarking tool (v4.0.0) comparing SV calls against the GIAB gold standard
- name: rasusa
  role: Subsampling long-read BAMs to 40x, 30x, 20x, 10x, and 5x coverage (v0.6.0)
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:486eef9cebc47e351f25b0fb3b8dbedf2f144679dba9c1f273c766dd2da7ee22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Dipcall
  - PAV
  - SVIM-asm
  - Truvari
  - VolcanoSV
  - rasusa
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 24
---

# assembly-based-structural-variant-calling

## Summary

Benchmark and compare assembly-based structural variant callers (VolcanoSV-vc, PAV, SVIM-asm, Dipcall) on long-read data, evaluating DEL/INS detection and false discovery rates against gold standards.

## When to use

Use when long-read (HiFi/CLR/ONT) BAMs must be assessed for SV detection quality versus assembly-based baselines, or when false discovery of SVs against an alternate reference (T2T-CHM13) must be quantified.

## When NOT to use

- The input is a CHM13 sample for VolcanoSV-asm haplotype phasing, since the sample is homozygous and unsuitable for phasing
- Detection of complex SVs (translocations, duplications, inversions) on single-chromosome data, since translocation detection requires a WGS BAM as support
- Genotyping reciprocal translocations, where genotyping accuracy is ambiguous and only F1/GT_F1 should be calculated

## Inputs

- Long-read BAM files (PacBio HiFi, CLR, or ONT) aligned to hg19/GRCh38 reference
- hg19/GRCh38 and T2T-CHM13 reference FASTA files with .fai indexes
- chrom_list.txt listing one contig name per line matching the reference
- GIAB HG002 SV gold standard VCF/BED
- chm13v2.0_censat_v2.0.bed and chm13v2.0_telomere.bed annotation tracks
- CHM13 diploid assembly FASTA with hp1/hp2 haplotype tags

## Outputs

- Per-tool DEL/INS VCFs for each dataset and coverage level
- sv_benchmark_metrics.csv with recall, precision, F1, and genotype accuracy per tool, SV type, and coverage
- false_discovery_rate_table.tsv with per-tool FDR and centromere/telomere overlap counts
- Truvari comparison outputs in high-confidence regions

## How to apply

Subsample long-read BAMs (Hifi_L1-L5, CLR_L1-L3, ONT_L1-L6) with rasusa v0.6.0 to 40x/30x/20x/10x/5x coverage, then run VolcanoSV in WGS mode per chromosome via volcanosv-asm.py with the platform-appropriate -dtype (Hifi/CLR/ONT), followed by volcanosv-vc-large-indel.py, volcanosv-vc-complex-sv.py, and volcanosv-vc-small-indel.py with -cl chrom_list.txt, merging outputs with Merge_VCF.py. Run the assembly-based baselines Dipcall, SVIM-asm (v1.0.2), and PAV (v2.0.0) on the same BAMs. Benchmark DEL/INS VCFs against the GIAB HG002 SV gold standard with Truvari v4.0.0 using p=0.5, P=0.5, r=500, S=30, O=0.01, extracting recall, precision, F1, and genotype accuracy per tool, SV type, and coverage. For false discovery analysis, compute FDR as the ratio of SVs detected against T2T-CHM13 to SVs against GRCh38, and intersect false discoveries with chm13v2.0_censat_v2.0.bed and chm13v2.0_telomere.bed.

## Related tools

- **VolcanoSV** (Hybrid SV detection pipeline combining reference-guided and local de novo assembly; run per chromosome with platform-appropriate -dtype) — https://github.com/maiziezhoulab/VolcanoSV
- **Dipcall** (Assembly-based baseline SV caller (v0.3) for DEL, INS, small indel comparison) — https://github.com/lh3/dipcall
- **SVIM-asm** (Assembly-based baseline caller (v1.0.2) for DEL, INS, DUP, INV, TRA comparison) — https://github.com/eldariont/svim-asm
- **PAV** (Assembly-based baseline caller (v2.0.0) for DEL, INS, INV, small indel comparison) — https://github.com/EichlerLab/pav
- **Truvari** (Benchmarking tool (v4.0.0) comparing SV calls against the GIAB gold standard)
- **rasusa** (Subsampling long-read BAMs to 40x, 30x, 20x, 10x, and 5x coverage (v0.6.0))

## Examples

```
volcanosv-asm.py -bam Hifi_L1.bam -ref GRCh38.fa -chr chr10 -t 16 -asm hifiasm; volcanosv-vc-large-indel.py -cl chrom_list.txt; Merge_VCF.py; truvari bench -g GIAB.vcf --pctsize 0.5 --pctseq 0.5 --refdist 500 --sizemin 30 --offset 0.01
```

## Evaluation signals

- Across Hifi datasets VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions; deviations from these ranges suggest misconfiguration
- VolcanoSV preserves high F1 for both insertions and deletions even at 10x coverage on Hifi_L1 subsampling, outperforming PAV, SVIM-asm, and Dipcall under subsampling
- VolcanoSV-vc should exhibit the lowest false discovery rate (ratio of SVs against CHM13 to SVs against GRCh38) among the four assembly-based callers
- Truvari benchmarking must use high-confidence GIAB regions with p=0.5, P=0.5, r=500, S=30, O=0.01 to reproduce reported metrics
- All merged VCFs from Merge_VCF.py should contain phased, haplotype-resolved SV calls consistent with the chrom_list contigs supplied

## Limitations

- The GIAB gold standard SV callset is not a complete set and could also contain false positives
- VolcanoSV only achieved the best insertion/deletion recall in 3–4 out of 6 ONT datasets
- Detecting small indels based on assembly proved ineffective for ONT data (low precision for all three tools)
- VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different

## Evidence

- [abstract] VolcanoSV surpasses state-of-the-art assembly-based tools in the detection of insertion and deletion SVs, exhibiting superior recall, precision, F1 scores, and genotype accuracy: "VolcanoSV surpasses state-of-the-art assembly-based tools in the detection of insertion and deletion SVs, exhibiting superior recall, precision, F1 scores, and genotype accuracy"
- [results] Truvari parameters used for benchmarking: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Subsampling with rasusa to coverage levels: "All three datasets were subsampled to 40x, 30x, 20x, 10x, and 5x coverage using rasusa (v0.6.0)"
- [results] False discovery rate definition: "The false discovery rate was computed as the ratio of the number of detected SVs against CHM13 to the number of SVs against GRCh38"
- [readme] README on chrom_list requirement for WGS mode: "Whenever a pipeline step needs to know the full set of chromosomes/contigs to process (WGS mode), you must supply a --chrom_list/-cl (or -chl) text file listing one chromosome/contig name per line"
