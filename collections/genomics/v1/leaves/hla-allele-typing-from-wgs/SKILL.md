---
name: hla-allele-typing-from-wgs
description: Use when predicting HLA class I/II alleles from WGS long-read FASTQ data
  (PacBio HiFi or nanopore), especially when clinically confirmed HLA types are available
  for benchmarking and disk-heavy SAM intermediates should be avoided.
when_to_use_negative:
- Input is Illumina short-read data — the README states best short-read results come
  from the HPTASR targeted-assembly route with reads 100 bp and up (ideally 150 bp),
  not streamed long-read HPRA
- Reads are so short (e.g., 50 bp) that 4-digit HLA allele predictions would be ambiguous,
  or no long-read aligner output is available to stream
- The goal is de novo targeted assembly of HLA contigs (HPTASR) rather than direct
  read-alignment-based prediction from long reads
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: minimap2
  role: Aligns long reads to the chr6-masked GRCh38+HLA reference with the -ax map-hifi
    preset and --MD, streaming SAM output to HLAminer
  repo: https://github.com/lh3/minimap2
- name: HLAminer (v1.4 / HLAminer.pl)
  role: Receives streamed SAM alignments and scores candidate HLA-I/II alleles in
    HPRA mode, emitting HLAminer_HPRA.csv and .log
  repo: https://github.com/bcgsc/HLAminer
- name: TASR
  role: Underlying component used to build/read the HLA reference k-mer database in
    the HLAminer distribution
  repo: https://github.com/warrenlr/TASR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:4a02340a92d9f83dd2ed56cc3765e8857517f8276ba006c357dbe6a43e27d071
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HLAminer (v1.4 / HLAminer.pl)
  - TASR
  - minimap2
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hla-allele-typing-from-wgs

## Summary

Predicts HLA class I and II alleles from whole-genome shotgun long-read sequencing (PacBio HiFi, ONT PromethION) by streaming minimap2 SAM alignments directly into HLAminer v1.4 in read-alignment (HPRA) mode, avoiding intermediate .sam files on disk.

## When to use

Use when predicting HLA class I/II alleles from WGS long-read FASTQ data (PacBio HiFi or nanopore), especially when clinically confirmed HLA types are available for benchmarking and disk-heavy SAM intermediates should be avoided.

## When NOT to use

- Input is Illumina short-read data — the README states best short-read results come from the HPTASR targeted-assembly route with reads 100 bp and up (ideally 150 bp), not streamed long-read HPRA
- Reads are so short (e.g., 50 bp) that 4-digit HLA allele predictions would be ambiguous, or no long-read aligner output is available to stream
- The goal is de novo targeted assembly of HLA contigs (HPTASR) rather than direct read-alignment-based prediction from long reads

## Inputs

- WGS long-read FASTQ files (e.g., PacBio HiFi/CCS subreads or ONT PromethION reads, ENA/SRA accessions such as ERR2585112)
- GRCh38 reference FASTA with chr6 removed or masked plus HLA-I/II genomic alleles (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz)
- IMGT/HLA-derived HLA-I/II reference FASTA (../database/HLA-I_II_GEN.fasta)
- hla_nom_p.txt P-designation file (http://hla.alleles.org/wmda/hla_nom_p.txt)

## Outputs

- HLAminer_HPRA.csv — concise summary with top two highest-scoring predictions per HLA gene, P-designated alleles where antigen-binding domains are identical
- HLAminer_HPRA.log — full record of command, alignments and all scores/predictions

## How to apply

Concatenate the input long-read FASTQ files (e.g., the 39 NA24385 PacBio CCS subread files) into a single read set, then align with minimap2 using the -ax map-hifi preset with --MD against a GRCh38-based reference (GCA_000001405.15) comprising all chromosomes except (or with masked) chr6 plus HLA-I/II genomic alleles. Pipe the SAM output directly into HLAminer.pl in HPRA mode with -a stream, -h ../database/HLA-I_II_GEN.fasta, -s 500, -q 1, -i 1 and -p ../database/hla_nom_p.txt, so no bulky .sam files are written to disk. Collect the HLAminer_HPRA.csv summary, which keeps the top two highest-scoring predictions per HLA gene and reports the P-designated allele when antigen-binding domains are identical, plus the HLAminer_HPRA.log detailing the command, alignments and scores. The HPRA mode is chosen over HPTASR because it is conceptually simpler and faster, aligning reads up-front to reference HLA alleles. Validate by comparing ranked predictions against clinically confirmed types; in the NA24385 30x HiFi run (~1 hr 14 min), B*35:01 and DRB1*04:03P were predicted versus clinical B*35:08 and DRB1*04:02P.

## Related tools

- **minimap2** (Aligns long reads to the chr6-masked GRCh38+HLA reference with the -ax map-hifi preset and --MD, streaming SAM output to HLAminer) — https://github.com/lh3/minimap2
- **HLAminer (v1.4 / HLAminer.pl)** (Receives streamed SAM alignments and scores candidate HLA-I/II alleles in HPRA mode, emitting HLAminer_HPRA.csv and .log) — https://github.com/bcgsc/HLAminer
- **TASR** (Underlying component used to build/read the HLA reference k-mer database in the HLAminer distribution) — https://github.com/warrenlr/TASR

## Examples

```
minimap2 -ax map-hifi --MD GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz na24385_hifi.fastq | HLAminer.pl -a stream -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt
```

## Evaluation signals

- HLAminer_HPRA.csv contains exactly the top two highest-scoring predictions per HLA gene, with P-designated alleles reported when the summary includes sequences with identical antigen-binding domains
- Predicted alleles reproduce the clinically confirmed types for the benchmark individual — e.g., B*35:01 and DRB1*04:03P among top predictions for NA24385
- Runtime is in line with the reported ~1 hr 14 min for the 30x NA24385 PacBio HiFi run
- HLAminer_HPRA.log records the command, alignments and scores, and no large .sam files remain on disk (streaming was used)
- Tuning thresholds (-s 500, -q 1, -i 1 here vs. defaults -s 1000, -q 30, -i 99) is documented in the log and justified by the long-read/HPRA setting

## Limitations

- Predictions resolve to P-designated alleles when antigen-binding domains are identical, so reported types may differ from clinical typing at the finest allele level (e.g., B*35:01 vs B*35:08; DRB1*04:03P vs DRB1*04:02P)
- Predictions were demonstrated to be robust with older, less accurate nanopore data and ~10x coverage, but very low or degraded input coverage may weaken allele resolution
- v1.4 features (SAM streaming, nanopore long-read support) require HLAminer v1.4; older versions lack streaming support

## Evidence

- [abstract] a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer: "a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer"
- [abstract] Streamed HPRA eliminates bulky SAM intermediates: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [methods] HPRA mode rationale: "The HLA prediction from direct read alignment (HPRA) method is conceptually simpler and faster to execute, since paired reads are aligned up-front to reference HLA alleles."
- [methods] P-designation and top-two summary behavior: "Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain."
- [intro] Benchmark datasets with clinical truth: "we utilized one PacBio HiFi and four ONT PromethION whole‐genome shotgun datasets derived from three individuals with known, clinically confirmed HLA types"
- [readme] v1.4 streaming capability: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
- [readme] Short-read scope caveat: "BEST SHORT READ RESULTS ARE OBTAINED WITH HPTASR WITH READS 100bp AND UP (IDEALLY 150bp)."
