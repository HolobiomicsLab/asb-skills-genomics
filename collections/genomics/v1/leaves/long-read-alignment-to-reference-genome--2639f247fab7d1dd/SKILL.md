---
name: long-read-alignment-to-reference-genome
description: Use when raw ONT or PacBio long-read sequencing data (FASTQ) must be
  mapped to GRCh38 to generate haplotype-resolved BAM alignments for genome-wide analysis
  such as TRiCoLOR TR calling.
when_to_use_negative:
- Input is already a haplotype-resolved BAM alignment (e.g., the deposited PB alignments
  for HG00731/HG00732, which the paper uses as-is) — skip realignment
- Input is short-read Illumina data — minimap2 long-read alignment is designed for
  error-prone third-generation reads, not Illumina FM-index-based validation
- The goal is variant calling on reads from unassembled regions such as centromeres
  and telomeres, where TRiCoLOR-based downstream profiling cannot operate
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns ONT FASTQ long reads to the human GRCh38 reference genome to produce
    haplotype-resolved BAMs for TRiCoLOR
- name: TRiCoLOR
  role: Downstream consumer of the aligned haplotype-resolved BAM files for de novo
    tandem repeat profiling
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:0e96bdc6abde6bdbcb9864b91e3275e501ef260ef20bf01a2ae9b7071bdc27d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - Docker
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# long-read-alignment-to-reference-genome

## Summary

Align error-prone long-read sequencing data (ONT FASTQ or PB reads) to a haplotype-resolved reference genome such as human GRCh38 using minimap2, producing haplotype-resolved BAM files that serve as input for downstream de novo tandem repeat profiling with TRiCoLOR.

## When to use

Use when raw ONT or PacBio long-read sequencing data (FASTQ) must be mapped to GRCh38 to generate haplotype-resolved BAM alignments for genome-wide analysis such as TRiCoLOR TR calling.

## When NOT to use

- Input is already a haplotype-resolved BAM alignment (e.g., the deposited PB alignments for HG00731/HG00732, which the paper uses as-is) — skip realignment
- Input is short-read Illumina data — minimap2 long-read alignment is designed for error-prone third-generation reads, not Illumina FM-index-based validation
- The goal is variant calling on reads from unassembled regions such as centromeres and telomeres, where TRiCoLOR-based downstream profiling cannot operate

## Inputs

- ONT whole-genome sequencing FASTQ files (e.g., HGSVC ONT rebasecalled files from http://ftp.ebi.ac.uk/1000g/ftp/data_collections/hgsv_sv_discovery/working/20181210_ONT_rebasecalled)
- Human GRCh38 reference genome in FASTA format
- Deposited haplotype-resolved PacBio alignments (BAM) from http://ftp.ebi.ac.uk/1000g/ftp/data_collections/hgsv_sv_discovery/working/20180102_pacbio_blasr_reheader

## Outputs

- Haplotype-resolved BAM files of ONT long reads aligned to GRCh38, ready for TRiCoLOR SENSoR and REFER

## How to apply

Obtain the HGSVC ONT FASTQ files (e.g., for HG00514, HG00733, NA19240 from the EBI FTP deposit) and align them to the human GRCh38 reference genome with minimap2, using the ONT-appropriate preset since minimap2 compared favorably to alternative aligners on synthetic data in both speed and mapping accuracy. For PacBio data (HG00731/HG00732 trio), use the deposited haplotype-resolved PB alignments as-is rather than re-aligning. The choice of minimap2 is grounded in the article's benchmark, and the resulting BAMs must preserve read-to-haplotype assignment to enable downstream TRiCoLOR SENSoR/REFER processing on each haplotype-specific BAM with default parameters (20 bp sliding windows, entropy drop <=1.23, >=5 supporting reads).

## Related tools

- **minimap2** (Aligns ONT FASTQ long reads to the human GRCh38 reference genome to produce haplotype-resolved BAMs for TRiCoLOR)
- **TRiCoLOR** (Downstream consumer of the aligned haplotype-resolved BAM files for de novo tandem repeat profiling) — https://github.com/davidebolo1993/TRiCoLOR

## Examples

```
minimap2 -ax map-ont GRCh38.fa HG00733.ont.fastq > HG00733.aln.sam
```

## Evaluation signals

- Alignment rate and mapping quality consistent with ONT WGS on GRCh38 (minimap2 was benchmarked favorably on speed and mapping accuracy against alternative aligners)
- Resulting BAM is haplotype-resolved so TRiCoLOR SENSoR can be run separately per haplotype
- Downstream SENSoR with default parameters (entropy drop <=1.23, >=5 supporting reads, 20 bp windows, merging within 100 bp) yields tens of thousands of candidate regions that reduce to ~150,000 after coverage filtering (mean coverage >8 for ONT, >10x for PB)
- Coverage filter statistics per sample fall in the ranges reported by the paper after coverage-based filtering

## Limitations

- Downstream TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)
- TRiCoLOR does not support polyploid species or haploid chromosomes (e.g., the human Y chromosome)
- Alignment quality depends on long reads being long enough to encompass entire expanded repeats; insufficient read lengths challenge accurate TR resolution

## Evidence

- [results] We aligned the ONT FASTQ files to the human GRCh38 reference genome using minimap2: "We aligned the ONT FASTQ files to the human GRCh38 reference genome using minimap2"
- [methods] minimap2 compared favorably to alternative aligners on synthetic data, in terms of both speed and mapping accuracy: "minimap2 [19], which compared favorably to alternative aligners on syntethic data, in terms of both speed and mapping accuracy"
- [discussion] ONT FASTQ files: http://ftp.ebi.ac.uk/1000g/ftp/data_collections/hgsv_sv_discovery/working/20181210_ONT_rebasecalled: "ONT FASTQ files: http://ftp.ebi.ac.uk/1000g/ftp/data_collections/hgsv_sv_discovery/working/20181210_ONT_rebasecalled"
- [methods] TRiCoLOR (Tandem Repeats Caller for Long Reads) requires haplotype-resolved long-read alignments as input: "TRiCoLOR (Tandem Repeats Caller for Long Reads) requires haplotype-resolved long-read alignments as input"
