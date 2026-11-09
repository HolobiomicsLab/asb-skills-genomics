---
name: short-read-splice-aware-alignment
description: 'Use when an lrCaptureSeq (PacBio) isoform catalog needs independent
  validation: short-read RNA-seq (GSE101986, GSE79416) can confirm splice junctions,
  and CAGE-seq (DRA002410) can confirm 5'' transcription start sites.'
when_to_use_negative:
- The goal is to discover novel full-length isoform sequences de novo — typical RNA-seq
  read lengths are <200 bp and cannot resolve full-length multi-kilobase transcripts,
  so long-read lrCaptureSeq is required instead.
- The long-read isoform catalog has not yet been filtered (no 4+ exon, truncation-removed,
  95% abundance filtering), since aligning against an unfiltered GFF propagates truncated
  or low-confidence junctions.
- Validating isoforms that differ only at 3' ends in single-cell data, where 3' bias
  makes junction-based validation inappropriate.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: IsoPops
  role: Exports the filtered isoform catalog as a GFF splice-junction annotation for
    genome index construction
  repo: https://github.com/kellycochran/IsoPops
- name: STAR
  role: Two-pass splice-aware alignment of short-read RNA-seq reads against a genome
    index built with the isoform GFF (--sjdbGTFfile)
- name: HISAT2
  role: Splice-alignment of short CAGE reads to the mm10 (GRCm38) reference genome
- name: BedTools
  role: coverage/intersect to quantify CAGE 5'-end coverage over isoform transcription
    start sites
- name: SQANTI
  role: Classifies each junction by splice-site motif as canonical vs non-canonical
- name: R
  role: Computes supported-junction fractions and writes per-isoform/per-junction
    result tables
  repo: https://github.com/kellycochran/IsoPops
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:9465d9675d7dcadc940e76aa1745c2645a27ba66fc79ea5d11b219cee7ceb9d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kellycochran/IsoPops
  tools:
  - BedTools
  - HISAT2
  - IsoPops
  - R
  - SQANTI
  - STAR
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# short-read-splice-aware-alignment

## Summary

Splice-aware alignment of independent short-read RNA-seq or CAGE-seq reads against the mouse genome, guided by long-read-defined isoform annotations, to validate lrCaptureSeq splice junctions and transcription start sites.

## When to use

Use when an lrCaptureSeq (PacBio) isoform catalog needs independent validation: short-read RNA-seq (GSE101986, GSE79416) can confirm splice junctions, and CAGE-seq (DRA002410) can confirm 5' transcription start sites.

## When NOT to use

- The goal is to discover novel full-length isoform sequences de novo — typical RNA-seq read lengths are <200 bp and cannot resolve full-length multi-kilobase transcripts, so long-read lrCaptureSeq is required instead.
- The long-read isoform catalog has not yet been filtered (no 4+ exon, truncation-removed, 95% abundance filtering), since aligning against an unfiltered GFF propagates truncated or low-confidence junctions.
- Validating isoforms that differ only at 3' ends in single-cell data, where 3' bias makes junction-based validation inappropriate.

## Inputs

- Filtered lrCaptureSeq isoform catalog GFF (e.g. fullfilter.gff from IsoPops filter_db with exon_min_count=4 and truncations removed)
- Short-read RNA-seq FASTQ files from GEO accessions GSE101986 and GSE79416 (retina and brain)
- Adult mouse retina CAGE-seq reads from DDBJ accession DRA002410 (samples DRX019832-DRX019834)
- mm10 (GRCm38) mouse reference genome FASTA

## Outputs

- Per-junction classification TSV (junction coordinates, short-read support status, canonical/non-canonical label from SQANTI)
- Per-isoform CAGE 5'-end coverage values and TSS-validation fraction (cage_tss_coverage.csv; expected 1051/1076 = 97.7%)
- STAR BAM alignments of short reads against the isoform-annotated genome index

## How to apply

First export the filtered IsoPops isoform catalog (e.g. the 4+ exon, truncation-removed, 95% abundance filtered fullfilter.gff) as a GFF using IsoPops write_gff_data. Build a STAR genome index supplying this GFF via --sjdbGTFfile so reads are guided to the long-read-defined junctions, then align short reads (GEO GSE101986, GSE79416) in two-pass mode and tally reads spanning each predicted donor-acceptor junction; expect ~98.1% of 79,020 junctions to be short-read-supported and run SQANTI to classify junctions as canonical vs non-canonical (~98.9% canonical expected). Separately, splice-align adult retina CAGE reads (DRA002410, samples DRX019832-DRX019834) to mm10/GRCm38 with HISAT2, then use BedTools coverage/intersect to quantify CAGE 5'-end coverage over each isoform's TSS; expect 1051/1076 (97.7%) of adult retina isoforms corroborated. The rationale is that short reads alone cannot resolve full-length multi-kilobase transcripts, but junction-spanning and 5'-end reads independently confirm the long-read-defined structures.

## Related tools

- **IsoPops** (Exports the filtered isoform catalog as a GFF splice-junction annotation for genome index construction) — https://github.com/kellycochran/IsoPops
- **STAR** (Two-pass splice-aware alignment of short-read RNA-seq reads against a genome index built with the isoform GFF (--sjdbGTFfile))
- **HISAT2** (Splice-alignment of short CAGE reads to the mm10 (GRCm38) reference genome)
- **BedTools** (coverage/intersect to quantify CAGE 5'-end coverage over isoform transcription start sites)
- **SQANTI** (Classifies each junction by splice-site motif as canonical vs non-canonical)
- **R** (Computes supported-junction fractions and writes per-isoform/per-junction result tables) — https://github.com/kellycochran/IsoPops

## Examples

```
STAR --runMode genomeGenerate --genomeDir star_idx --genomeFastaFiles mm10.fa --sjdbGTFfile fullfilter.gff --sjdbOverhang 149; STAR --genomeDir star_idx --readFilesIn sample_R1.fastq sample_R2.fastq --twopassMode Basic
```

## Evaluation signals

- Short-read-supported junction fraction falls in [0.96, 0.99]; the paper reports 98.1% across n = 79,020 junctions.
- Canonical-junction fraction from SQANTI splice-site motif classification is approximately 98.9%.
- CAGE TSS-validation fraction equals 1051/1076 ≈ 97.7% of adult retina isoforms showing CAGE 5'-end coverage at their start sites.
- junction_classification.tsv contains per-junction coordinates, short-read support, and canonical/non-canonical labels; cage_tss_coverage.csv contains per-isoform 5' coverage values.
- STAR index was built with --sjdbGTFfile pointing at the long-read-defined isoform GFF, not only the genome FASTA.

## Limitations

- Independent short-read datasets can validate junctions and 5' ends but cannot resolve full-length sequences of multi-kilobase transcripts by themselves.
- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of the lrCaptureSeq library preparation, so validation fractions are conditional on the cataloged set.
- The catalogs may lack certain short and/or rare transcripts; absence of short-read support cannot fully exclude the possibility that a cataloged isoform is an artifact or that others are missing.

## Evidence

- [results] Independent short-read RNA-seq datasets supported 98.1% of lrCaptureSeq splice junctions (79,020 junctions), validating the full-length sequences in the isoform catalog.: [evidence span withheld — anchor status: not_found]
- [results] CAGE reads from adult mouse retina corroborated 97.7% of lrCaptureSeq transcription start sites: [evidence span withheld — anchor status: not_found]
- [methods] Build a STAR genome index supplying the isoform GFF as the splice-junction annotation (--sjdbGTFfile): [evidence span withheld — anchor status: not_found]
- [methods] Splice-align the short CAGE reads to the mm10 (GRCm38) mouse reference genome using HISAT2: [evidence span withheld — anchor status: not_found]
- [intro] because typical RNA-seq read lengths are <200 bp, the method is not able to resolve the full-length sequence of multi-kilobase transcripts: "because typical RNA-seq read lengths are <200 bp, the method is not able to resolve the full-length sequence of multi-kilobase transcripts"
