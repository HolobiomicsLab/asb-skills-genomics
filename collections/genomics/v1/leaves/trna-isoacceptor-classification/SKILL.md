---
name: trna-isoacceptor-classification
description: Use when small RNA-seq reads map to tRNA loci and you must classify them
  into tRF-5s, tRF-3s, tRF-1s, or tRNA halves by progenitor isoacceptor (e.g. GluCTC,
  GlyTCC, LeuTAA, ProCGG) and cleavage position.
when_to_use_negative:
- Input reads are human or non-plant small RNA data — the referenced detection tools
  (tDRmapper, MINTmap, tRF2Cancer) are trained/tested on human datasets while this
  classification workflow was validated on six angiosperms
- Reads map ambiguously to both tRNA-flank artificial chromosomes and other genomic
  regions — such reads must not be classified as tncRNAs since only exclusively mapping
  reads qualify
- The goal is de novo small RNA discovery unrelated to tRNA origin (e.g. miRNA or
  piRNA profiling)
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: tncRNA Toolkit
  role: In-house workflow that detects, classifies, and quantifies tncRNAs (tRF-5s,
    tRF-3s, tRF-1, tRNA halves) from genome FASTA and processed single-end sRNA-seq
    FASTQ
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: bowtie1
  role: Aligns processed reads to the artificial tRNA-flank genome index with '--norc
    -v 0 -m 50 --best --strata' to produce unique, error-free tRNA-derived alignments
    used to assign cleavage positions
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tRNAscan-SE
  role: Predicts the tRNA gene pool; pseudogenes and tRNAs scoring below 50 are eliminated
    before building the artificial genome used for isoacceptor assignment
- name: biopython
  role: pairwise2 module builds local alignments of tncRNAs to their tRNAs of origin
    (match=1, mismatch=-1, gap-open=-1, gap-extend=-0.5) to inspect the cleavage-interface
    motif
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Trim Galore
  role: Quality- and adapter-trims single-end small RNA reads before tncRNA classification
    (v0.6.6 used in the study)
  repo: https://github.com/FelixKrueger/TrimGalore
- name: pandas
  role: Loads read FASTA with per-read counts and tRNA annotations, computes per-isoacceptor
    nucleotide frequencies and RPM, and writes the summary TSV
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:23827ed8fa092bf68a632bd79594b0e2db15496a6ae645a85fc8a8ec06abeabf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Trim Galore
  - biopython
  - bowtie1
  - pandas
  - tRNAscan-SE
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# trna-isoacceptor-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies tRNA-derived non-coding RNAs (tncRNAs) by their progenitor mature tRNA isoacceptor and cleavage position, enabling codon-dependent cleavage analysis (e.g. detecting the conserved AG::TGG tRF-5::tRNA interface motif) rather than treating reads as random tRNA degradation products.

## When to use

Use when small RNA-seq reads map to tRNA loci and you must classify them into tRF-5s, tRF-3s, tRF-1s, or tRNA halves by progenitor isoacceptor (e.g. GluCTC, GlyTCC, LeuTAA, ProCGG) and cleavage position.

## When NOT to use

- Input reads are human or non-plant small RNA data — the referenced detection tools (tDRmapper, MINTmap, tRF2Cancer) are trained/tested on human datasets while this classification workflow was validated on six angiosperms
- Reads map ambiguously to both tRNA-flank artificial chromosomes and other genomic regions — such reads must not be classified as tncRNAs since only exclusively mapping reads qualify
- The goal is de novo small RNA discovery unrelated to tRNA origin (e.g. miRNA or piRNA profiling)

## Inputs

- Unique mapped tncRNA reads (FASTA with per-read counts extracted from SAM flags 0 & 16)
- Mature tRNA annotations with 50 bp 5' and 3' flanks (tRNAscan-SE output, score >= 50, pseudogenes removed)
- Quality-filtered, adapter-trimmed single-end small RNA FASTQ (e.g. Trim Galore v0.6.6 processed)
- Bowtie1 alignment (SAM) of reads to the artificial tRNA-flank genome

## Outputs

- TSV table of cleavage-position counts and nucleotide compositions per tRNA isoacceptor
- tncRNA subtype classification (tRF-5, tRF-3, tRF-1, tRNA half) with progenitor tRNA locus and cleavage position
- RPM-normalized abundance per progenitor tRNA isoacceptor
- Detected conserved tRF-5::tRNA interface motif (AG::TGG) at tRF-5 5'-ends

## How to apply

Build an artificial genome by masking genuine tRNA gene regions with 50 nt upstream and downstream flanks (tRNAscan-SE predicted, pseudogenes and score <50 eliminated) and classify reads as tncRNAs only if they map exclusively to these artificial chromosomes. Align processed reads with bowtie1 using '--norc -v 0 -m 50 --best --strata' so multi-mapped reads (>50 alignments) are discarded and only uniquely mapping, error-free reads are retained. For each read, determine the cleavage position relative to the progenitor mature tRNA from the locus, location, and read-length mapping, classifying tRNA halves by cleavage within the 7 nt anticodon loop window (2 nt + 3 nt anticodon + 2 nt). Compute nucleotide frequencies at cleavage positions per isoacceptor and normalize abundance as RPM = reads mapped to progenitor tRNA / (total mapped reads / 1e6); then build local alignments of tncRNAs to their tRNAs of origin with biopython pairwise2 (match=1, mismatch=-1, gap-open=-1, gap-extend=-0.5) to inspect read-end/tRNA interfaces for the conserved AG::TGG motif marking tRF-5::tRNA cleavage at 15–19 nt tRF-5s.

## Related tools

- **tncRNA Toolkit** (In-house workflow that detects, classifies, and quantifies tncRNAs (tRF-5s, tRF-3s, tRF-1, tRNA halves) from genome FASTA and processed single-end sRNA-seq FASTQ) — https://github.com/skbinfo/tncRNA-Toolkit
- **bowtie1** (Aligns processed reads to the artificial tRNA-flank genome index with '--norc -v 0 -m 50 --best --strata' to produce unique, error-free tRNA-derived alignments used to assign cleavage positions) — https://github.com/skbinfo/tncRNA-Toolkit
- **tRNAscan-SE** (Predicts the tRNA gene pool; pseudogenes and tRNAs scoring below 50 are eliminated before building the artificial genome used for isoacceptor assignment)
- **biopython** (pairwise2 module builds local alignments of tncRNAs to their tRNAs of origin (match=1, mismatch=-1, gap-open=-1, gap-extend=-0.5) to inspect the cleavage-interface motif) — https://github.com/skbinfo/tncRNA-Toolkit
- **Trim Galore** (Quality- and adapter-trims single-end small RNA reads before tncRNA classification (v0.6.6 used in the study)) — https://github.com/FelixKrueger/TrimGalore
- **pandas** (Loads read FASTA with per-read counts and tRNA annotations, computes per-isoacceptor nucleotide frequencies and RPM, and writes the summary TSV) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python3 tncRNAs.py -s arabidopsis -i processed_sRNA.fastq -o out_dir -v 0 -m 50 -c 10
```

## Evaluation signals

- The cleavage-position nucleotide distribution per isoacceptor is non-uniform and codon-dependent (not uniform random cleavage), consistent with precise endoribonucleolytic cleavage
- The AG::TGG interface motif is detected at the 5'-ends of tRF-5s of 15–19 nt across all plants analyzed
- All classified tncRNAs map exclusively to the artificial tRNA-flank chromosomes, and no read with more than 50 bowtie alignments was retained
- tRNA halves fall within the 7 nt anticodon loop window (2 nt + 3 nt anticodon + 2 nt) for their progenitor tRNA
- RPM values computed as reads mapped to progenitor tRNA / (total mapped reads / 1e6) are finite and traceable to the input read counts

## Limitations

- Conventional RNA-seq suffers from tRNA modification biases that can skew cleavage-position nucleotide composition; tRNA-seq is recommended to overcome this
- Allowing mismatches to tolerate sequencing errors may cause base misidentification and inflation of false negatives, hence the workflow uses -v 0 (zero mismatches) with bowtie1
- Mechanisms by which tRNA modifications affect cleavage regulation remain unclear, and the role of isodecoders (tRNAs sharing the same anticodon) in tncRNA biogenesis requires further exploration
- In-silico predictions require experimental validation to distinguish bona fide tncRNAs

## Evidence
