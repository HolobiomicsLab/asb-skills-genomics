---
name: barcode-demultiplexing-of-spikein-cdnas
description: Use when CCS reads from PacBio subreads (movieX.ccs.bam/fasta) contain
  PAIso-seq spike-in cDNAs with the 16-nt barcode 5'-GAGTGCTACTCTAGTA-3' that must
  be assigned to tail-length groups before tail-length accuracy assessment.
when_to_use_negative:
- Reads lack the 16-nt spike-in barcode 5'-GAGTGCTACTCTAGTA-3' (e.g. biological GV-oocyte
  cDNAs barcoded with distinct dU-containing templates rather than the synthetic spike-in
  barcode), since trim.py matching would fail
- Analysis goal is transcriptome-wide poly(A) tail annotation of native transcripts
  rather than spike-in accuracy validation, which requires genome alignment with minimap2
  -x splice and the TAPIS pipeline instead of barcode grouping
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0622
tools:
- name: ccs
  role: Generates circular consensus sequences from spike-in subreads (version 3.4.1;
    --richQVs recommended) to provide the high-accuracy reads that barcode matching
    operates on
  repo: https://github.com/PacificBiosciences/IsoSeq3/blob/master/README_v3.2.md
- name: bam2fasta
  role: Converts the CCS BAM (movieX.ccs.bam) to FASTA (bam2fasta -u -o ccs) required
    by trim.py
- name: scripts/trim.py
  role: Demultiplexes spike-in reads by the 16-nt barcode and trims the 25-nt 3'-adapter
    using the combined 41-nt string with match parameters 22 2
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: scripts/GetCCSpass.pl
  role: Collects CCS pass counts so downstream tail-length calls after demultiplexing
    can retain only reads with >=10 passes
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:3371be1250d714568b442dbdd5ec0649dee6454bad8088478da0e02d1ec2e9fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Perl
  - R
  - bam2fasta
  - ccs
  - scripts/GetCCSpass.pl
  - scripts/trim.py
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# barcode-demultiplexing-of-spikein-cdnas

## Summary

Demultiplexes PAIso-seq spike-in CCS reads by the 16-nt barcode (5'-GAGTGCTACTCTAGTA-3') and trims the 25-nt 3'-adapter (5'-GTACTCTGCGTTGATACCACTGCTT-3') so per-read poly(A) tail lengths can be assigned to each synthetic tail-length group (expected 10/30/50/70/100 nt).

## When to use

Use when CCS reads from PacBio subreads (movieX.ccs.bam/fasta) contain PAIso-seq spike-in cDNAs with the 16-nt barcode 5'-GAGTGCTACTCTAGTA-3' that must be assigned to tail-length groups before tail-length accuracy assessment.

## When NOT to use

- Reads lack the 16-nt spike-in barcode 5'-GAGTGCTACTCTAGTA-3' (e.g. biological GV-oocyte cDNAs barcoded with distinct dU-containing templates rather than the synthetic spike-in barcode), since trim.py matching would fail
- Analysis goal is transcriptome-wide poly(A) tail annotation of native transcripts rather than spike-in accuracy validation, which requires genome alignment with minimap2 -x splice and the TAPIS pipeline instead of barcode grouping

## Inputs

- PacBio subread BAM from SMRTbell spike-in libraries (movieX.subreads.bam)
- Circular consensus sequence BAM (movieX.ccs.bam) generated with ccs --richQVs
- CCS FASTA converted via bam2fasta (movieX.ccs.fasta)
- 16-nt spike-in barcode 5'-GAGTGCTACTCTAGTA-3' and 25-nt 3'-adapter 5'-GTACTCTGCGTTGATACCACTGCTT-3' as a 41-nt match string

## Outputs

- Demultiplexed, adapter-trimmed spike-in FASTA (sample.out.fasta) keyed by barcode
- Rejected/unmatched read FASTA (sample.err.fasta)
- Per-barcode mean poly(A) tail lengths and mismatch rates summarized in spikein_tail_lengths.tsv

## How to apply

First generate circular consensus sequences from spike-in subreads with ccs using --richQVs (the README suggests `ccs movieX.subreads.bam movieX.ccs.bam --richQVs`), then convert the CCS BAM to FASTA with `bam2fasta -u -o ccs`. Next run scripts/trim.py on movieX.ccs.fasta with the combined barcode+adapter string GAGTGCTACTCTAGTAGTACTCTGCGTTGATACCACTGCTT and match parameters 22 2, which demultiplexes by the 16-nt barcode and trims the 25-nt 3'-adapter, writing sample.out.fasta (correct reads) and sample.err.fasta (rejected reads). Discard ambiguous reads that could be classified into two or more samples. The demultiplexed FASTA is then passed to the measurement pipeline (perl run.pl --ccs sample.out.fasta ... --pass ccs.pass.txt), where reads with >=10 passes support tail-length calls; correct application is judged by each barcode's mean measured tail length closely matching its expected length (10, 28, 48, 67, and 97 nt observed vs 10, 30, 50, 70, and 100 nt expected).

## Related tools

- **ccs** (Generates circular consensus sequences from spike-in subreads (version 3.4.1; --richQVs recommended) to provide the high-accuracy reads that barcode matching operates on) — https://github.com/PacificBiosciences/IsoSeq3/blob/master/README_v3.2.md
- **bam2fasta** (Converts the CCS BAM (movieX.ccs.bam) to FASTA (bam2fasta -u -o ccs) required by trim.py)
- **scripts/trim.py** (Demultiplexes spike-in reads by the 16-nt barcode and trims the 25-nt 3'-adapter using the combined 41-nt string with match parameters 22 2) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **scripts/GetCCSpass.pl** (Collects CCS pass counts so downstream tail-length calls after demultiplexing can retain only reads with >=10 passes) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
./scripts/trim.py movieX.ccs.fasta sample GAGTGCTACTCTAGTAGTACTCTGCGTTGATACCACTGCTT 22 2 1>sample.out.fasta 2>sample.err.fasta
```

## Evaluation signals

- Each barcode group's mean measured tail length closely matches its expected spike-in length: 10, 28, 48, 67, and 97 nt vs expected 10, 30, 50, 70, and 100 nt
- Ambiguous reads classifiable into two or more samples are discarded rather than assigned
- Non-adenosine mismatch rate within spike-in poly(A) sequences is computed and is very low (spike-ins should be pure A-tails)
- Per-read tail lengths are only aggregated from reads with >=10 CCS passes
- Unmatched reads are written to sample.err.fasta and excluded from sample.out.fasta

## Limitations

- PAIso-seq cannot accurately quantify non-A modifications at the very last 3'-end base, since terminal G/C/U pair to the adaptor and are lost, which slightly biases measured spike-in tail ends
- A minimal number of additional A residues may be introduced during pairing of the extension primer, contributing small deviations (e.g. 97 nt observed vs 100 nt expected)
- Accuracy depends on high CCS pass counts; the pipeline requires a pass number of 10 or more to ensure the accuracy of base calling

## Evidence

- [results] PAIso-seq measured mean poly(A) tail lengths of 10, 28, 48, 67, and 97 nt for the spike-ins, very close to the expected 10, 30, 50, 70, and 100 nt, demonstrating accurate tail-length assessment.: [evidence span withheld — anchor status: not_found]
- [readme] Barcode Sequence: 5'-GAGTGCTACTCTAGTA-3' (16 nt): "Barcode Sequence: 5'-GAGTGCTACTCTAGTA-3' (16 nt)"
- [readme] bam2fasta -u -o ccs movieX.ccs.bam: "bam2fasta -u -o ccs movieX.ccs.bam"
- [readme] trim.py movieX.ccs.fasta sample GAGTGCTACTCTAGTAGTACTCTGCGTTGATACCACTGCTT 22 2: "trim.py movieX.ccs.fasta sample GAGTGCTACTCTAGTAGTACTCTGCGTTGATACCACTGCTT 22 2"
- [discussion] ambiguous reads that could be classified into two or more samples, were discarded: "ambiguous reads that could be classified into two or more samples, were discarded"
