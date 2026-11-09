---
name: long-read-allele-prediction
description: Use when you have raw uncorrected WGS long-read FASTQ (ONT PromethION
  or PacBio) and need HLA-I/-II allele predictions at group/allele resolution in a
  single command, without persisting SAM/BAM files.
when_to_use_negative:
- Input reads are Illumina-sized short reads (50-150 bp) — HPTASR targeted assembly
  with reads >=100 bp (ideally 150 bp) is the preferred mode; 4-digit allele predictions
  may be ambiguous with shorter reads
- You need de novo assembled HLA contigs for downstream reciprocal BLASTN analysis
  rather than direct alignment-based allele scoring
- You need variant calling or genome-wide alignment output retained on disk, since
  this skill deliberately streams and discards SAM output
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Aligns ONT long reads to the combined GRCh38+IMGT/HLA reference with the ONT
    preset (-ax map-ont --MD); its SAM output is streamed to HLAminer without intermediate
    files
  repo: https://github.com/lh3/minimap2
- name: HLAminer
  role: Runs HLAminer.pl v1.4 in HPRA stream mode to score streamed SAM alignments
    and emit HLA class I/II allele predictions
  repo: https://github.com/bcgsc/HLAminer
- name: TASR
  role: 'Alternative HPTASR mode: builds a hash table of 15 nt k-mers from IMGT/HLA
    reference sequences to recruit and de novo assemble HLA reads (used instead of
    streaming when targeted assembly is desired)'
  repo: https://github.com/warrenlr/TASR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:286f0df590ec8fd830d42f7aa5d527f6eb9fa2774a6747f6f9dc4e5a7e963a48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HLAminer
  - TASR
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# long-read-allele-prediction

## Summary

Predict HLA class I and II alleles directly from whole-genome shotgun long-read sequencing data by streaming minimap2 SAM alignments into HLAminer (v1.4) HPRA stream mode, avoiding storage of bulky alignment files on disk.

## When to use

Use when you have raw uncorrected WGS long-read FASTQ (ONT PromethION or PacBio) and need HLA-I/-II allele predictions at group/allele resolution in a single command, without persisting SAM/BAM files.

## When NOT to use

- Input reads are Illumina-sized short reads (50-150 bp) — HPTASR targeted assembly with reads >=100 bp (ideally 150 bp) is the preferred mode; 4-digit allele predictions may be ambiguous with shorter reads
- You need de novo assembled HLA contigs for downstream reciprocal BLASTN analysis rather than direct alignment-based allele scoring
- You need variant calling or genome-wide alignment output retained on disk, since this skill deliberately streams and discards SAM output

## Inputs

- Whole-genome shotgun long-read FASTQ (e.g., ONT PromethION NA24385: dataset D guppy v5.0.6 ~67x, dataset E dorado kit14-2022-12 ~72x)
- Combined reference FASTA: GRCh38 chr-only (no chr6 or MHC 28510120-33480577 masked) plus IMGT/HLA class I and II genomic allele sequences (HLA-I_II_GEN.fasta)
- hla_nom_p.txt P designation file (http://hla.alleles.org/wmda/hla_nom_p.txt)

## Outputs

- HLAminer_HPRA.csv — concise HLA allele summary with top two highest-scoring predictions per HLA gene
- HLAminer_HPRA.log — full log listing all predictions

## How to apply

Build a combined long-read reference by concatenating the GRCh38 chromosome-only genome (GCA_000001405.15) with chr6 excluded — or with the MHC region 28510120-33480577 on chr6 masked — and the up-to-date HLA-I and HLA-II genomic allele sequences from IMGT/HLA (scripts in ./database/ download and format these). Align the WGS FASTQ with minimap2 using the ONT preset (-ax map-ont --MD) and pipe the SAM stream directly into HLAminer.pl v1.4 in HPRA stream mode with parameters -h HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p hla_nom_p.txt -a stream, where -s/-q/-i are the minimum score, log10 phred-like expect value, and minimum % sequence identity filters. Collect the HLAminer_HPRA.csv summary and HLAminer_HPRA.log; the CSV keeps the top two highest-scoring predictions per gene and only the 'P' designated allele when sequences share the same antigen binding domain. Rationale: HPRA is conceptually simpler and faster than HPTASR targeted assembly because reads are aligned up-front, and streaming eliminates the need to store bulky alignment files on disk.

## Related tools

- **minimap2** (Aligns ONT long reads to the combined GRCh38+IMGT/HLA reference with the ONT preset (-ax map-ont --MD); its SAM output is streamed to HLAminer without intermediate files) — https://github.com/lh3/minimap2
- **HLAminer** (Runs HLAminer.pl v1.4 in HPRA stream mode to score streamed SAM alignments and emit HLA class I/II allele predictions) — https://github.com/bcgsc/HLAminer
- **TASR** (Alternative HPTASR mode: builds a hash table of 15 nt k-mers from IMGT/HLA reference sequences to recruit and de novo assemble HLA reads (used instead of streaming when targeted assembly is desired)) — https://github.com/warrenlr/TASR

## Examples

```
minimap2 -ax map-ont --MD GRCh38-noChr6-HLA-I_II_GEN.fa NA24385.fastq.gz | ../bin/HLAminer.pl -h HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p hla_nom_p.txt -a stream > HLAminer_HPRA.csv
```

## Evaluation signals

- Top two highest-scoring predictions per HLA gene in HLAminer_HPRA.csv match the clinically confirmed/reference HLA types (e.g., B*35:01, DRB1*04:03P for NA24385)
- Class I and II allele calls are concordant across both base-caller datasets (guppy v5.0.6 and dorado kit14-2022-12) and across sequencing runs/platforms
- HLAminer_HPRA.log contains the full list of all predictions, while HLAminer_HPRA.csv retains only the top two per gene and the 'P' designated allele for identical antigen binding domains
- Predictions reproduce despite differing coverage levels (39x, 67x, 72x), consistent with reported robustness down to ~10x coverage
- The SAM stream was consumed directly (no alignment file on disk), confirming the pipe from minimap2 to HLAminer.pl completed without truncation

## Limitations

- HLAminer v1.4 provides only 'initial support' for predicting from raw uncorrected shotgun nanopore long reads; older, less accurate base-caller chemistry can reduce accuracy
- Robustness degrades at relatively low sequencing coverage (demonstrated at ~10x); high-coverage PromethION datasets (39x-72x) are what yield clinically corroborated calls
- For short reads, HPTASR works best with reads >=100 bp (ideally 150 bp); at 50 bp, 4-digit HLA allele predictions may be ambiguous
- Larger TASR k values (default 15 nt) increase specificity at the possible expense of sensitivity; blast 2.2.22 may be faster than blast+ 2.2.28 while producing accurate results

## Evidence

- [abstract] a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer: "a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer"
- [methods] HLAminer v1.4 provides initial support for HLA prediction from raw uncorrected shotgun nanopore long reads: "HLAminer v1.4 provides initial support for HLA prediction from raw uncorrected shotgun nanopore long reads"
- [readme] The HLA prediction from direct read alignment (HPRA) method is conceptually simpler and faster to execute, since paired reads are aligned up-front to reference HLA alleles.: "The HLA prediction from direct read alignment (HPRA) method is conceptually simpler and faster to execute, since paired reads are aligned up-front to reference HLA alleles."
- [readme] Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer.: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
- [readme] Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain.: "Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain."
