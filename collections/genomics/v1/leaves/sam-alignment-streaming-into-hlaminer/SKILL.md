---
name: sam-alignment-streaming-into-hlaminer
description: Use when you have WGS long-read FASTQ data (ONT PromethION or PacBio)
  and need HLA-I/II allele group predictions without storing bulky alignment files,
  using any SAM-with-MD-tag outputting aligner.
when_to_use_negative:
- Input is Illumina short-read data — HPTASR targeted assembly is the preferred route
  (100–150 bp reads give best 4-digit resolution), not SAM streaming of long-read
  alignments
- You need to retain the full alignment file for downstream non-HLA analyses — the
  streaming approach deliberately discards the SAM output without writing it to disk
- Reads are not from a shotgun WGS/exome/RNA-seq dataset compatible with read-alignment
  HLA inference (HPRA expects alignments to HLA-bearing references)
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3534
tools:
- name: HLAminer
  role: Consumes streamed SAM alignments and derives HLA-I/II allele predictions via
    HPRA scoring
  repo: https://github.com/bcgsc/HLAminer
- name: minimap2
  role: Reference aligner used to produce the baseline SAM stream ('-t 60 -ax map-ont
    --MD' against the HLA-substituted GRCh38 reference); substitutable by any SAM/MD-tag
    aligner
  repo: https://github.com/lh3/minimap2
- name: TASR
  role: Component used to build the k-mer hash table of reference HLA alleles for
    candidate mining in the HPTASR mode
  repo: https://github.com/warrenlr/TASR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:993ae972a6418f8063cd325be0fbe9ffb22a9be4fc847cc835079d3e6ef39a33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - HLAminer v1.4
  - HLAminer.pl
  - TASR
  - minimap2
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3534
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# sam-alignment-streaming-into-hlaminer

## Summary

Stream SAM output from any SAM-compatible read aligner (e.g. minimap2 with -ax map-ont --MD) directly into HLAminer v1.4 via a Unix pipe to predict HLA class I and II allele groups from long-read WGS data, avoiding intermediate .sam files on disk.

## When to use

Use when you have WGS long-read FASTQ data (ONT PromethION or PacBio) and need HLA-I/II allele group predictions without storing bulky alignment files, using any SAM-with-MD-tag outputting aligner.

## When NOT to use

- Input is Illumina short-read data — HPTASR targeted assembly is the preferred route (100–150 bp reads give best 4-digit resolution), not SAM streaming of long-read alignments
- You need to retain the full alignment file for downstream non-HLA analyses — the streaming approach deliberately discards the SAM output without writing it to disk
- Reads are not from a shotgun WGS/exome/RNA-seq dataset compatible with read-alignment HLA inference (HPRA expects alignments to HLA-bearing references)

## Inputs

- WGS long-read FASTQ reads (e.g. NA19240 PromethION nanopore, ENA ERR2585115 under PRJEB26791)
- GRCh38 reference FASTA with HLA region replaced by HLA-I_II_GEN sequences (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz)
- IMGT/HLA HLA-I_II_GEN reference FASTA database
- hla_nom_p.txt P-designation file from http://hla.alleles.org/wmda/hla_nom_p.txt

## Outputs

- HLAminer_HPRA.csv (ranked per-gene HLA prediction summary, top two per group)
- HLAminer_HPRA.log (full prediction list)

## How to apply

Prepare a reference genome that lacks the polymorphic HLA region on chr6 but includes the IMGT/HLA HLA-I_II_GEN sequences appended (e.g. GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz). Align reads with minimap2 ('-t 60 -ax map-ont --MD') or any aligner that emits SAM with MD tags, and pipe the SAM output directly into HLAminer.pl with parameters '-h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream', where -s/-q/-i are relaxed score, expect-value and identity thresholds suitable for error-prone nanopore reads. HLAminer runs the HPRA (HLA prediction by read alignment) mode and writes HLAminer_HPRA.csv plus HLAminer_HPRA.log; the CSV summary keeps the top two highest-scoring predictions per gene and collapses alleles sharing the same antigen binding domain to the 'P' designated allele. Aligner-agnostic substitution (e.g. an alternative long-read aligner producing SAM with MD tags) is valid as long as predictions remain concordant with the minimap2 baseline and known clinical HLA types.

## Related tools

- **HLAminer** (Consumes streamed SAM alignments and derives HLA-I/II allele predictions via HPRA scoring) — https://github.com/bcgsc/HLAminer
- **minimap2** (Reference aligner used to produce the baseline SAM stream ('-t 60 -ax map-ont --MD' against the HLA-substituted GRCh38 reference); substitutable by any SAM/MD-tag aligner) — https://github.com/lh3/minimap2
- **TASR** (Component used to build the k-mer hash table of reference HLA alleles for candidate mining in the HPTASR mode) — https://github.com/warrenlr/TASR

## Examples

```
minimap2 -t 60 -ax map-ont --MD GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz ERR2585115.fastq.gz | ../bin/HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream
```

## Evaluation signals

- HLAminer_HPRA.csv exists and parses as a ranked per-gene prediction table with top-two alleles per HLA gene
- Predicted allele groups match clinically confirmed NA19240 types (e.g. A*30/A*68, B*35/B*57, C*04/C*18)
- Alternative-aligner predictions are concordant with the minimap2 baseline groups on the same dataset
- Predictions are reproducible from the single piped command with no intermediate .sam file created on disk
- Predictions remain robust at ~10x coverage even with older-chemistry nanopore data

## Limitations

- Older, less accurate nanopore chemistry datasets and low (~10x) coverage are tolerated, but predictions are described as 'fairly accurate' rather than definitive
- HPTASR mode needs reads ≥100 bp (ideally 150 bp) for unambiguous 4-digit allele resolution; 50 bp reads give ambiguous calls
- No changelog was found in the article's discussion section; version behavior differences (e.g. v1.2 vs v1.3 summary format) must be tracked manually

## Evidence

- [abstract] a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer: "a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer"
- [abstract] works with any read aligner compatible with the SAM file format: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [readme] Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer.: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer.
Initial support for predicting HLA types from long nanopore reads"
- [methods] The minimap2-based baseline on dataset A predicts HLA-I heterozygosity with groups A26 and A33: [evidence span withheld — anchor status: not_found]
- [methods] Stream the SAM output directly into HLAminer.pl via a Unix pipe: [evidence span withheld — anchor status: not_found]
- [methods] Keeps top two [highest-scoring by HLA group] predictions per gene: "Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain."
