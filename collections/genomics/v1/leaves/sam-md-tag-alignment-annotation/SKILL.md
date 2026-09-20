---
name: sam-md-tag-alignment-annotation
description: Use when a long-read (nanopore/PacBio) WGS FASTQ must be typed for HLA
  class I and II alleles and the aligner output is SAM with MD tags — e.g., substituting
  a non-minimap2 aligner in the HLAminer streaming protocol.
when_to_use_negative:
- Short Illumina-sized reads — the HPTASR targeted-assembly mode (reads ≥100 bp, ideally
  150 bp) is the recommended route, not SAM streaming from long-read aligners
- The input is already a completed SAM/BAM alignment without MD tags — MD tags are
  required for the streaming protocol; re-alignment or re-tagging is needed first
- A bulky intermediate alignment file must be archived anyway — this skill's rationale
  (in-memory piping) is moot if the SAM must be stored, though it still functions
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: HLAminer
  role: Receives the streamed SAM via stdin and performs HPRA-based HLA-I/II allele
    scoring and prediction
  repo: https://github.com/bcgsc/HLAminer
- name: HLAminer v1.4
  role: The specific version adding SAM streaming support and initial nanopore read
    prediction; executed via HLAminer.pl with '-a stream'
  repo: https://github.com/bcgsc/HLAminer
- name: minimap2
  role: Baseline long-read aligner producing SAM with MD tags ('-t 60 -ax map-ont
    --MD'); benchmark against which alternative SAM-with-MD-tag aligners are compared
  repo: https://github.com/lh3/minimap2
- name: TASR
  role: Used within HLAminer to read IMGT/HLA reference sequences and build the k-mer
    (15-mer) hash database used for prediction
  repo: https://github.com/warrenlr/TASR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:f07cb3c38160efdecf7640e28654bfd100a67cce3f384a6d11ee6c4512c11001
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - HLAminer v1.4
  - TASR
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# sam-md-tag-alignment-annotation

## Summary

Streams SAM-formatted long-read alignments (with MD tags) directly from any SAM-compatible read aligner into HLAminer v1.4 to predict HLA-I and -II allele groups, avoiding intermediate SAM files on disk. Enables aligner-agnostic HLA typing from ONT PromethION or PacBio whole-genome shotgun data via a single Unix pipe.

## When to use

Use when a long-read (nanopore/PacBio) WGS FASTQ must be typed for HLA class I and II alleles and the aligner output is SAM with MD tags — e.g., substituting a non-minimap2 aligner in the HLAminer streaming protocol.

## When NOT to use

- Short Illumina-sized reads — the HPTASR targeted-assembly mode (reads ≥100 bp, ideally 150 bp) is the recommended route, not SAM streaming from long-read aligners
- The input is already a completed SAM/BAM alignment without MD tags — MD tags are required for the streaming protocol; re-alignment or re-tagging is needed first
- A bulky intermediate alignment file must be archived anyway — this skill's rationale (in-memory piping) is moot if the SAM must be stored, though it still functions

## Inputs

- Long-read WGS FASTQ (e.g., ERR2585115 from ENA accession PRJEB26791, NA19240 PromethION nanopore reads)
- Reference genome FASTA with HLA-I/II sequences: GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz
- IMGT/HLA reference database: HLA-I_II_GEN.fasta
- IMGT P designation file: hla_nom_p.txt (http://hla.alleles.org/wmda/hla_nom_p.txt)

## Outputs

- HLAminer_HPRA.csv — ranked per-gene HLA allele group prediction table (top two predictions per gene)
- HLAminer_HPRA.log — log listing all predictions
- Streamed SAM alignment output (with MD tags) consumed in-pipe, not persisted

## How to apply

Align the long-read FASTQ against a reference genome prepared as GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz (GRCh38 minus chr6 HLA region plus HLA-I/II GEN sequences), forcing MD tags in the SAM output (minimap2 baseline used '-t 60 -ax map-ont --MD'). Pipe the SAM stream directly into HLAminer.pl with '-h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream', so no bulky .sam file is written to disk; HLAminer v1.4 added this streaming capability specifically for nanopore reads. The reference HLA database should be refreshed from the IMGT/HLA repository using the download shell scripts in ./database/, and the P designation file (hla_nom_p.txt) supplies collapsed allele designations. The HPRA (HLA prediction from read alignment) mode scores candidate alignments and emits the top two highest-scoring predictions per HLA gene; parameters -s (minimum score), -q (minimum log10 phred-like expect value), and -i (minimum % sequence identity) control the sensitivity/specificity trade-off. To validate an aligner substitution, compare predicted allele groups (e.g., A*26:33/A*33:24 for the NA19240 minimap2 baseline on dataset A) against the baseline groups and the clinically confirmed NA19240 types (A*30/A*68, B*35/B*57, C*04/C*18).

## Related tools

- **HLAminer** (Receives the streamed SAM via stdin and performs HPRA-based HLA-I/II allele scoring and prediction) — https://github.com/bcgsc/HLAminer
- **HLAminer v1.4** (The specific version adding SAM streaming support and initial nanopore read prediction; executed via HLAminer.pl with '-a stream') — https://github.com/bcgsc/HLAminer
- **minimap2** (Baseline long-read aligner producing SAM with MD tags ('-t 60 -ax map-ont --MD'); benchmark against which alternative SAM-with-MD-tag aligners are compared) — https://github.com/lh3/minimap2
- **TASR** (Used within HLAminer to read IMGT/HLA reference sequences and build the k-mer (15-mer) hash database used for prediction) — https://github.com/warrenlr/TASR

## Examples

```
minimap2 -t 60 -ax map-ont --MD GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz ERR2585115.fastq.gz | ../bin/HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream
```

## Evaluation signals

- HLAminer_HPRA.csv exists and parses as a ranked per-gene prediction table with the top two highest-scoring predictions per HLA group
- Predicted allele groups are concordant with the minimap2 baseline (e.g., HLA-A groups A26 and A33, alleles A*26:33 and A*33:24 on dataset A)
- Predictions match the clinically confirmed NA19240 types: A*30/A*68, B*35/B*57, C*04/C*18
- The SAM stream carries MD tags (e.g., produced via minimap2 '--MD' or the alternative aligner's equivalent option)
- No intermediate .sam file is written to disk — the pipe passes aligner output directly to HLAminer.pl

## Limitations

- Predictions were demonstrated robust even with older, less accurate (2018) nanopore chemistry and only ~10x coverage, but accuracy is qualified as 'fairly accurate', not definitive; clinically confirmed types should be the arbiter
- Any read aligner must emit SAM compatible with MD tags; aligners lacking MD-tag generation cannot be substituted without post-processing
- Version 1.4 does not include the concise allele summary improvements beyond keeping top two per-gene predictions; the full prediction list lives in the .log only

## Evidence

- [abstract] works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [readme] SAM streaming capability in v1.4: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
- [methods] nanopore prediction accuracy: "Results below demonstrate HLAminer's ability to fairly accurately predict HLA-I and -II types from direct [and streamed] nanopore sequencing reads [old chemistry] alignments"
- [methods] identity threshold parameter: "-i minimum % sequence identity...............<99>"
