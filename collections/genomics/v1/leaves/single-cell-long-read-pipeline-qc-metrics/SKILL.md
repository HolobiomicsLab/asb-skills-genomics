---
name: single-cell-long-read-pipeline-qc-metrics
description: Use when deposited subreads BAM files from a 10x Genomics + PacBio single-cell
  isoform experiment must be reprocessed through CCS and FLNC classification to verify
  reported per-sample QC metrics.
when_to_use_negative:
- Input is already demultiplexed FLNC fastq or HiFi reads — no CCS or primer-classification
  step is needed
- The goal is isoform-level characterization (gffcompare, SQANTI3 filtering, isoform
  count matrices) rather than Table 1 read-processing metrics
- Libraries were not constructed via the 10x Genomics ScISOr-Seq/Linked-scISOseq/HIT-scISOseq
  primer structure (classify_by_primer detects full-length transcripts specifically
  for these protocols)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: PacBio ccs (SMRTlink)
  role: Generates circular consensus reads from subreads BAM with the paper's modified
    parameters
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: samtools
  role: Converts ccs.bam to ccs.fa and ccs.fq for BLAST and classification
- name: ncbi-blast
  role: makeblastdb/blastn align primers to CCS reads (word_size 5, outfmt 7) as input
    to FLNC classification
- name: scISA-Tools classify_by_primer.pl
  role: Extracts FLNC reads with cell barcode/UMI/polyA handling under min_primerlen
    16 and min_isolen 50
  repo: https://github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:0b999176b06d6d2f6019c95936264278b3a8cbbe889f46b5a0b77210280dd72e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - PacBio Sequel II
  - PacBio ccs (SMRTlink)
  - ncbi-blast
  - samtools
  - scISA-Tools
  - scISA-Tools classify_by_primer.pl
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-long-read-pipeline-qc-metrics

## Summary

Reproduce the Table 1 read-processing QC metrics (CCS read counts/lengths, FLNC counts, artifact cDNA percentages) for ScISOr-Seq, Linked-scISOseq, and HIT-scISOseq from deposited PacBio subreads BAMs using the paper's stated CCS and FLNC-filtering parameters.

## When to use

Use when deposited subreads BAM files from a 10x Genomics + PacBio single-cell isoform experiment must be reprocessed through CCS and FLNC classification to verify reported per-sample QC metrics.

## When NOT to use

- Input is already demultiplexed FLNC fastq or HiFi reads — no CCS or primer-classification step is needed
- The goal is isoform-level characterization (gffcompare, SQANTI3 filtering, isoform count matrices) rather than Table 1 read-processing metrics
- Libraries were not constructed via the 10x Genomics ScISOr-Seq/Linked-scISOseq/HIT-scISOseq primer structure (classify_by_primer detects full-length transcripts specifically for these protocols)

## Inputs

- PacBio subreads BAM files per sample (HIT-scISOseq and ScISOr-Seq/Linked-scISOseq baseline libraries)
- primer.fa with primer_F (5') and primer_S (3') sequences
- scISA-Tools classify_by_primer.pl script
- ncbi-blast (>=2.10.0+), samtools, SMRTlink >= 8.0 installed and on PATH

## Outputs

- Per-sample ccs.bam and converted ccs.fa/ccs.fq
- mapped.m7 primer BLAST output (-outfmt 7)
- isoseq_flnc.BarcodeUMI.fastq, isoseq_flnc.Transcript.fastq, isoseq.PrimerStat.csv
- Per-sample Table 1 metrics: total CCS reads, mean CCS read length, FLNC read count, artifact cDNA percentage

## How to apply

Retrieve the per-sample subreads BAM files and run PacBio ccs with the paper's modified SMRT-Link parameters '--min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75' to generate ccs.bam per sample. Convert ccs.bam to ccs.fa/ccs.fq with samtools view plus awk, then build a primer BLAST database from primer.fa (primer_F = 5' primer AAGCAGTGGTATCAACGCAGAGTACATGGG, primer_S = 3' primer CTACACGACGCTCTTCCGATCT) and align reads with blastn -word_size 5 -outfmt 7. Run classify_by_primer.pl with -min_primerlen 16 -min_isolen 50 to extract FLNC reads and split BarcodeUMI vs Transcript fastqs. Count total CCS reads, mean CCS read length, FLNC reads, and derive artifact cDNA percentage (reads lacking valid cell barcode/primer structure) per sample; HIT-scISOseq's biotin-capture step should drop this from ~51-53% (baseline protocols) to ~7.6-8.2%.

## Related tools

- **PacBio ccs (SMRTlink)** (Generates circular consensus reads from subreads BAM with the paper's modified parameters) — https://github.com/shizhuoxing/scISA-Tools
- **samtools** (Converts ccs.bam to ccs.fa and ccs.fq for BLAST and classification)
- **ncbi-blast** (makeblastdb/blastn align primers to CCS reads (word_size 5, outfmt 7) as input to FLNC classification)
- **scISA-Tools classify_by_primer.pl** (Extracts FLNC reads with cell barcode/UMI/polyA handling under min_primerlen 16 and min_isolen 50) — https://github.com/shizhuoxing/scISA-Tools

## Examples

```
ccs *.subreads.bam ccs.bam --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75 -j 20
```

## Evaluation signals

- Recomputed artifact cDNA percentages fall within reported ranges: ~51-53% for ScISOr-Seq/Linked-scISOseq (51.31%/53.44% s1/s2; 53.42% Linked) and ~7.6-8.2% for HIT-scISOseq (7.58% s1, 8.20% s2)
- Per-sample total CCS read counts, mean CCS read lengths, and FLNC counts match the corresponding Table 1 rows
- classify_by_primer.pl produces the three expected outputs: isoseq_flnc.BarcodeUMI.fastq, isoseq_flnc.Transcript.fastq, isoseq.PrimerStat.csv
- Read counts in ccs.fq equal the record count in ccs.bam after the awk conversion
- The BLAST step uses the exact 5'/3' primer names primer_F/primer_S required by classify_by_primer

## Limitations

- The example.ccs.bam in the repo (5000 CCS reads) is a test fixture from the BGI patented HIT-scISOseq protocol, not full sample data
- classify_by_primer is specific to the 10x Genomics ScISOr-Seq primer structure and HIT-scISOseq protocol; other library designs will misclassify
- CCS parameter changes (min-passes 0, min-rq 0.75) intentionally include low-pass reads, so absolute counts differ from default SMRT-Link QC outputs

## Evidence

- [discussion] CCS reads were generated with the following modified parameters: "--min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75": [evidence span withheld — anchor status: not_found]
- [methods] classify_by_primer perl command with min_primerlen 16 and min_isolen 50: "perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./"
- [readme] readme classification step with example.ccs.bam test data: "perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./"
- [intro] ~50% undesirable cell-barcode-free reads in ScISOr-Seq: "the 10× Genomics single-cell preparation pipeline of ScISOr-Seq introduces a high proportion (~50%) of undesirable cell-barcode-free reads"
- [full_text] task finding on artifact cDNA percentages: [evidence span withheld — anchor status: not_found]
