---
name: sam-streaming-bioinformatics-pipelines
description: Use when running long-read WGS (e.g. ONT PromethION) alignments with
  minimap2 and you need immediate downstream prediction (HLA typing) without storing
  large intermediate SAM files.
when_to_use_negative:
- Input reads are short Illumina-sized reads better handled by HLAminer's targeted-assembly
  HPTASR mode
- You need to retain or re-examine the raw alignments afterwards — stream mode discards
  the SAM, so write alignments to disk instead
- Data are already assembled contigs or an existing alignment file rather than raw
  sequencing reads
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3542
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: Long-read aligner (-x map-ont) whose SAM output is streamed into HLAminer
    through a Unix pipe
  repo: https://github.com/lh3/minimap2
- name: HLAminer v1.4
  role: Consumes the streamed SAM alignments in HPRA mode (-a stream) and scores HLA-I/II
    predictions
  repo: https://github.com/bcgsc/HLAminer
- name: HLAminer.pl
  role: Perl driver script invoked with '-h HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p
    hla_nom_p.txt -a stream'
  repo: https://github.com/bcgsc/HLAminer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:39855ec5b3764a863e8905c9d78c65a7141d2ba51604f83e7b168f2aad619afa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HLAminer v1.4
  - HLAminer.pl
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3542
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sam-streaming-bioinformatics-pipelines

## Summary

Streams a read aligner's SAM output directly into a downstream analysis tool (HLAminer) via a Unix pipe, avoiding writing bulky .sam files to disk. Used here to derive HLA class I/II predictions from long-read (ONT/PacBio) WGS alignments in a single command.

## When to use

Use when running long-read WGS (e.g. ONT PromethION) alignments with minimap2 and you need immediate downstream prediction (HLA typing) without storing large intermediate SAM files.

## When NOT to use

- Input reads are short Illumina-sized reads better handled by HLAminer's targeted-assembly HPTASR mode
- You need to retain or re-examine the raw alignments afterwards — stream mode discards the SAM, so write alignments to disk instead
- Data are already assembled contigs or an existing alignment file rather than raw sequencing reads

## Inputs

- FASTQ long-read WGS data (e.g. ONT PromethION or PacBio HiFi, via ENA/SRA accession)
- Masked reference genome FASTA with HLA-I/II alleles appended (GCA_000001405.15_GRCh38_genomic.chr-only-chr6hlamasked-HLA-I_II_GEN.fa.gz)
- IMGT/HLA allele reference FASTA (HLA-I_II_GEN.fasta)
- HLA P-designation file (hla_nom_p.txt from http://hla.alleles.org/wmda/hla_nom_p.txt)

## Outputs

- HLAminer_HPRA.csv summary with top two highest-scoring predictions per HLA gene
- HLAminer_HPRA.log listing all predictions by highest score
- SAM alignment stream consumed in-memory (no .sam file persisted)

## How to apply

Align the FASTQ reads against a prepared reference in which chr6 is masked and IMGT/HLA-I/II allele sequences are appended (GCA_000001405.15_GRCh38_genomic.chr-only-chr6hlamasked-HLA-I_II_GEN.fa.gz) using minimap2 with '-t 60 -ax map-ont --MD' for ONT reads. Pipe the SAM output directly into HLAminer.pl in HPRA stream mode with parameters '-h HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p hla_nom_p.txt -a stream', so the .sam is never materialized on disk. The stream mode was introduced in HLAminer v1.4 and works with any aligner emitting SAM. Collect the HLAminer_HPRA.csv summary (top two highest-scoring predictions per HLA gene) and HLAminer_HPRA.log (all predictions by highest score), then compare predicted allele groups (A, B, C, DQA1, DQB1, DRB1, DPA1, DPB1, DRA) against clinically confirmed types. On NA12878 39x ONT (SRR10965087) this took about 2 hr 21 min total for alignment plus prediction and matched all but one (94%) of HLA-I and all but three (88%) of HLA-II clinical groups.

## Related tools

- **minimap2** (Long-read aligner (-x map-ont) whose SAM output is streamed into HLAminer through a Unix pipe) — https://github.com/lh3/minimap2
- **HLAminer v1.4** (Consumes the streamed SAM alignments in HPRA mode (-a stream) and scores HLA-I/II predictions) — https://github.com/bcgsc/HLAminer
- **HLAminer.pl** (Perl driver script invoked with '-h HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p hla_nom_p.txt -a stream') — https://github.com/bcgsc/HLAminer

## Examples

```
minimap2 -t 60 -ax map-ont --MD GCA_000001405.15_GRCh38_genomic.chr-only-chr6hlamasked-HLA-I_II_GEN.fa.gz reads.fastq | ../bin/HLAminer.pl -h HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p hla_nom_p.txt -a stream
```

## Evaluation signals

- No .sam file is present on disk after the run — the pipe consumed the alignment output
- HLAminer_HPRA.csv contains exactly two highest-scoring predictions per HLA gene for all 9 genes (A, B, C, DQA1, DQB1, DRB1, DPA1, DPB1, DRA)
- Predicted allele groups match clinically confirmed HLA types at ~94% (HLA-I) and ~88% (HLA-II) accuracy on NA12878 benchmark data
- Total runtime of approximately 2 hr 21 min for alignment plus prediction is consistent with the reported benchmark (Table 2)

## Limitations

- Predictions are group-level; full 4-digit allele resolution may be ambiguous, especially at lower coverage
- Stream mode discards the SAM alignments, so alignments cannot be re-mined or audited without rerunning
- Requires a specially prepared reference (chr6 masked, HLA-I/II alleles appended) and current IMGT/HLA database files; older nanopore chemistry datasets are less accurate but predictions remain robust even at ~10x coverage

## Evidence

- [abstract] Stream minimap2 sequence alignments directly into HLAminer for HLA prediction: "predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer"
- [abstract] works with any read aligner compatible with the SAM file format: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [readme] v1.4 stream capability: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
- [methods] HPRA mode description: "The HLA prediction from direct read alignment (HPRA) method is conceptually simpler and faster to execute, since paired reads are aligned up-front to reference HLA alleles."
- [other] benchmark concordance: [evidence span withheld — anchor status: not_found]
