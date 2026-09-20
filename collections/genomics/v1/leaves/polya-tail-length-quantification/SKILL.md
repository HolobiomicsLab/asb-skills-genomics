---
name: polya-tail-length-quantification
description: Use when PacBio PAIso-seq CCS reads must be converted into per-read/per-gene
  poly(A) tail-length measurements, including spike-in accuracy validation or isoform-level
  tail-length comparisons.
when_to_use_negative:
- The input is short-read Illumina or TAIL-seq/PAL-seq data — TAIL-seq cannot handle
  tails larger than 231 nt and PAL-seq requires the discontinued Illumina Genome Analyzer
  II, whereas this skill depends on full-length PacBio CCS reads
- The goal is annotating poly(A) sites rather than measuring tail lengths — use the
  TAPIS pipeline (run_tapis.py) with GMAP alignments and PolyASite GRCm38-96 clusters
  instead
- 'Reads fail QC: fewer than 10 ccs passes, or a clipped tail shorter than 15 nt without
  five continuous adenosines — such reads must be discarded, not force-called'
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: ccs
  role: Generates circular consensus sequences from subreads (--richQVs or --noPolish
    --minPasses 1); pass number feeds the ≥10-pass QC cutoff
  repo: https://github.com/PacificBiosciences/IsoSeq3
- name: minimap2
  role: Aligns clean CCS reads to mm10/GRCm38 in splice mode (v2.15-r905, -x splice);
    terminal soft clips of primary alignments supply the candidate tail sequence
- name: Perl (run.pl, scripts/trim.py, scripts/GetCCSpass.pl)
  role: Trims barcode/adapter, collects CCS pass counts, and implements the four-criterion
    poly(A) tail-acceptance filter and tail-length recording
  repo: github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: R
  role: Aggregates per-read tail lengths into per-gene (geometric mean) and per-isoform
    summaries with Student's t-tests
  repo: github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_006
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:46e3a39d54071f43153efc499e3efab5a3e3ab87e9755f2b15e9f86c8d5acd63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/IsoSeq3
  tools:
  - Perl
  - Perl (run.pl, scripts/trim.py, scripts/GetCCSpass.pl)
  - R
  - ccs
  - ccs (version 3.4.1)
  - minimap2
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# polya-tail-length-quantification

## Summary

Quantify per-read and per-transcript poly(A) tail lengths from PAIso-seq data by extracting terminal soft-clipped sequences from minimap2 splice alignments of clean CCS reads and applying the paper's four-criterion tail-acceptance filter before recording tail length.

## When to use

Use when PacBio PAIso-seq CCS reads must be converted into per-read/per-gene poly(A) tail-length measurements, including spike-in accuracy validation or isoform-level tail-length comparisons.

## When NOT to use

- The input is short-read Illumina or TAIL-seq/PAL-seq data — TAIL-seq cannot handle tails larger than 231 nt and PAL-seq requires the discontinued Illumina Genome Analyzer II, whereas this skill depends on full-length PacBio CCS reads
- The goal is annotating poly(A) sites rather than measuring tail lengths — use the TAPIS pipeline (run_tapis.py) with GMAP alignments and PolyASite GRCm38-96 clusters instead
- Reads fail QC: fewer than 10 ccs passes, or a clipped tail shorter than 15 nt without five continuous adenosines — such reads must be discarded, not force-called

## Inputs

- PacBio subreads BAM (movieX.subreads.bam) from Sequel sequencing of PAIso-seq libraries
- CCS BAM or bam2fasta-converted CCS FASTA (movieX.ccs.bam / movieX.ccs.fasta)
- CCS pass-count table (ccs.pass.txt) from scripts/GetCCSpass.pl
- Mouse reference genome FASTA (Mus_musculus.GRCm38.dna.toplevel.chromosome.fa) and prebuilt minimap2 index (Mus_musculus.mmi)
- Spike-in reads demultiplexed by the 16-nt barcode GAGTGCTACTCTAGTA with defined tail lengths of 10, 30, 50, 70, and 100 nt

## Outputs

- Per-read poly(A) tail-length table (polya_tail_lengths.csv)
- Spike-in per-barcode mean measured tail lengths and mismatch rate (spikein_tail_lengths.tsv)
- Per-gene tail-length summaries, e.g. tail_length_protein_abundance.tsv (GV oocyte means: 62 nt high vs 56 nt low protein abundance)
- Per-isoform tail-length summaries with t-test p-values (isoform_tail_length_summary.tsv)

## How to apply

Generate circular consensus sequences with ccs (e.g. --richQVs), demultiplex by the 16-nt barcode GAGTGCTACTCTAGTA and trim the 25-nt 3'-adapter with scripts/trim.py using the combined string GAGTGCTACTCTAGTAGTACTCTGCGTTGATACCACTGCTT and match parameters 22 2, then keep reads with ≥10 ccs passes (via scripts/GetCCSpass.pl). Align clean CCS reads to the mm10/GRCm38 genome with minimap2 v2.15-r905 in splice mode and extract the terminal soft-clipped sequence of each primary alignment (ignoring alignments with a supplementary alignment tag) as the candidate tail. Accept the candidate as a poly(A) tail only if it is ≥15 nt long and contains at least five continuous adenosines; additionally require non-A residues to be <20 in count and <50% of the tail before recording the per-read tail length. Aggregate per-read lengths to per-gene (geometric mean) or per-isoform summaries in R; validate against synthetic spike-ins with expected 10/30/50/70/100-nt tails, which PAIso-seq measures as 10/28/48/67/97 nt.

## Related tools

- **ccs** (Generates circular consensus sequences from subreads (--richQVs or --noPolish --minPasses 1); pass number feeds the ≥10-pass QC cutoff) — https://github.com/PacificBiosciences/IsoSeq3
- **minimap2** (Aligns clean CCS reads to mm10/GRCm38 in splice mode (v2.15-r905, -x splice); terminal soft clips of primary alignments supply the candidate tail sequence)
- **Perl (run.pl, scripts/trim.py, scripts/GetCCSpass.pl)** (Trims barcode/adapter, collects CCS pass counts, and implements the four-criterion poly(A) tail-acceptance filter and tail-length recording) — github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **R** (Aggregates per-read tail lengths into per-gene (geometric mean) and per-isoform summaries with Student's t-tests) — github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- Every retained tail satisfies all four criteria: length ≥15 nt, ≥5 continuous adenosines, non-A count <20, and non-A percentage <50%
- Only primary, non-chimeric alignments (no supplementary alignment tag) contribute tails, and only reads with ≥10 ccs passes are scored
- Spike-in validation: measured mean tail lengths for the five barcoded spike-ins should approximate 10/28/48/67/97 nt against expected 10/30/50/70/100 nt
- Biological sanity check: high protein-abundance genes in GV oocytes show longer mean tails (62 nt) than low-abundance genes (56 nt)
- Isoform consistency: Ccnb1 APA3 tails significantly longer than APA1/APA2 (p=0.0067); Wee2 APA isoforms differ (p=3.9e-12)

## Limitations

- PAIso-seq cannot accurately quantify the frequency of non-A modifications at the very last 3'-end base, and a minimal number of additional A residues may be introduced during pairing of the extension primer
- 13-kb tails observed in only one replicate and very low abundance are not independently verified; biological significance of >231-nt tails awaits further investigation
- Replicate correlation of per-gene measurements (Fig. 2c) is limited by PacBio sequencing cost and expected to improve with deeper sequencing

## Evidence

- [discussion] Tail-acceptance criteria: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [discussion] Non-A residue filter: "the percentage of the non-A residues was <50% and the number of non-A residues were <20"
- [discussion] Pass-number cutoff: "We require a pass number of 10 or more to ensure the accuracy of base calling."
- [methods] minimap2 invocation: "minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa"
- [results] Spike-in accuracy: [evidence span withheld — anchor status: not_found]
