---
name: long-read-ccs-read-generation
description: Use when raw PacBio Sequel II subreads.bam files from short-insert single-cell
  isoform libraries (e.g., HIT-scISOseq, ScISOr-Seq) must be converted to ccs.bam
  before FLNC classification, especially when default CCS settings would discard the
  short ~1.5–5 kb inserts typical of scRNA cDNA.
when_to_use_negative:
- Long-insert (10–20 kb) standard Iso-Seq libraries where default SMRT-Link CCS parameters
  with higher minimum passes are appropriate and would give better accuracy.
- Datasets not sequenced on PacBio platforms (e.g., Oxford Nanopore PromethION reads),
  which have no circular consensus concept.
- Downstream tasks starting from already-generated CCS reads, such as FLNC classification
  or isoform collapsing — use classify_by_primer or cDNA_Cupcake instead.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: PacBio ccs / SMRTlink
  role: Generates ccs.bam from subreads.bam using --min-passes 0 --min-length 50 --max-length
    21000 --min-rq 0.75; CCS4.0 allows --chunk parallelization
- name: samtools
  role: Converts ccs.bam to ccs.fa and ccs.fq (preserving base quality strings) for
    primer blast and classify_by_primer
- name: PacBio Sequel II
  role: Instrument platform producing subreads from HIT-scISOseq SMRT Cell 8M libraries;
    >10 million long-reads per cell
- name: scISA-Tools
  role: Pipeline that consumes the ccs.bam/ccs.fq outputs for downstream FLNC classification
    and cellBC/UMI extraction
  repo: https://github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:597ee638a74b0a879d706caabe7ed5416d50974e178390aa4ffb6625352ccafb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - PacBio Sequel II
  - PacBio ccs / SMRTlink
  - samtools
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# long-read-ccs-read-generation

## Summary

Generate circular consensus sequencing (CCS) reads from PacBio Sequel II subreads BAM files using modified SMRT-Link parameters (--min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75) to maximize yield from short single-cell cDNA/concatemer libraries such as HIT-scISOseq.

## When to use

Use when raw PacBio Sequel II subreads.bam files from short-insert single-cell isoform libraries (e.g., HIT-scISOseq, ScISOr-Seq) must be converted to ccs.bam before FLNC classification, especially when default CCS settings would discard the short ~1.5–5 kb inserts typical of scRNA cDNA.

## When NOT to use

- Long-insert (10–20 kb) standard Iso-Seq libraries where default SMRT-Link CCS parameters with higher minimum passes are appropriate and would give better accuracy.
- Datasets not sequenced on PacBio platforms (e.g., Oxford Nanopore PromethION reads), which have no circular consensus concept.
- Downstream tasks starting from already-generated CCS reads, such as FLNC classification or isoform collapsing — use classify_by_primer or cDNA_Cupcake instead.

## Inputs

- PacBio Sequel II subreads.bam files (one per sample/library)
- SMRTlink 8.0 or later (smrttools-only install acceptable)
- Optional test input: example.ccs.bam from the scISA-Tools repo (5000 CCS reads)

## Outputs

- ccs.bam per sample (circular consensus reads)
- ccs.fa (SAM-derived FASTA of CCS reads)
- ccs.fq (SAM-derived FASTQ retaining base qualities for UMI filtering)
- Per-sample summary metrics: CCS read count and mean CCS read length

## How to apply

Run PacBio ccs (SMRTlink 8.0+; CCS4.0 supports --chunk parallelization) on each deposited subreads BAM with the paper's relaxed parameters: --min-passes 0 (accept single-pass subreads, needed because short concatemer inserts yield few passes), --min-length 50, --max-length 21000, and --min-rq 0.75. These settings deliberately trade per-read accuracy stringency for throughput, since short SMRTbell inserts cannot accumulate the 10+ passes required by default settings. Convert the resulting ccs.bam to FASTA/FASTQ via samtools view and awk so that primer blastn alignment (word_size 5) and classify_by_primer can extract FLNC reads downstream. Validate by recomputing per-sample CCS read counts and mean read lengths against the paper's Table 1 metrics.

## Related tools

- **PacBio ccs / SMRTlink** (Generates ccs.bam from subreads.bam using --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75; CCS4.0 allows --chunk parallelization)
- **samtools** (Converts ccs.bam to ccs.fa and ccs.fq (preserving base quality strings) for primer blast and classify_by_primer)
- **PacBio Sequel II** (Instrument platform producing subreads from HIT-scISOseq SMRT Cell 8M libraries; >10 million long-reads per cell)
- **scISA-Tools** (Pipeline that consumes the ccs.bam/ccs.fq outputs for downstream FLNC classification and cellBC/UMI extraction) — https://github.com/shizhuoxing/scISA-Tools

## Examples

```
ccs *.subreads.bam ccs.bam --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75 -j 20
```

## Evaluation signals

- ccs.bam contains a plausible read count per sample relative to the SMRT Cell 8M output (paper reports >10 million long-reads in a single cell).
- Recomputed per-sample CCS read counts and mean CCS read lengths match the Table 1 rows for ScISOr-Seq, Linked-scISOseq, and HIT-scISOseq.
- Mean read length distribution is consistent with short single-cell concatemer inserts (~5 kb concatemers containing 3–4 cDNAs), not the 15–20 kb long-insert regime.
- samtools-to-FASTQ conversion preserves base quality strings, verified by equal sequence and quality lengths in ccs.fq (needed for downstream UMI quality filtering of bases with Q<10).
- Downstream classify_by_primer with -min_primerlen 16 -min_isolen 50 runs without errors and produces isoseq_flnc.BarcodeUMI.fastq, isoseq_flnc.Transcript.fastq, and isoseq.PrimerStat.csv.

## Limitations

- --min-rq 0.75 and --min-passes 0 accept low-confidence single-pass reads; accuracy below default HiFi standards (>99.9%) is tolerated to preserve yield on short inserts.
- Short cDNA libraries (~1.5 kb average insert for a typical human transcriptome) severely impair PacBio CCS throughput relative to the recommended 10–20 kb inserts.
- HIT-scISOseq concatemers only reach ~5 kb (3–4 cDNAs); longer inserts are limited by nicks introduced during T4 DNA ligase ligation, for which no mitigation is yet available.

## Evidence

- [discussion] CCS read generation with modified SMRT-Link parameters: [evidence span withheld — anchor status: not_found]
- [readme] README Step1 Run CCS: "ccs *.subreads.bam ccs.bam --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75 -j 20"
- [intro] Short inserts impair CCS throughput: "an ordinary cDNA library usually has a much shorter average insert size (for example, ~1.5 kb for a typical human transcriptome), which severely impairs PacBio CCS throughput"
- [readme] Conversion of ccs.bam to fasta/fastq: "samtools view ccs.bam | awk '{print ">"$1"\n"$10}' > ccs.fa"
- [discussion] Concatemer insert length limitation: "the current version of the system enriches ~5 kb concatemers containing 3-4 ligated cDNAs (Fig. 1f)"
