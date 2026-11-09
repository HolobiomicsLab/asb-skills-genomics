---
name: long-read-hla-typing
description: Use when you have long-read WGS FASTQ data (ONT PromethION or PacBio
  HiFi) from a sample with unknown HLA type and need group-level HLA-I/II predictions
  using a single Unix pipeline command.
when_to_use_negative:
- Input is short Illumina-sized reads — use HPTASR (targeted assembly) mode instead;
  BEST SHORT READ RESULTS ARE OBTAINED WITH HPTASR WITH READS 100bp AND UP
- The analysis requires full allele-level (4-digit) resolution validated clinically
  rather than group-level predictions — HPRA streaming reports group-level and top-scoring
  allele predictions that may be ambiguous
- Only bulky SAM/alignment files are wanted as persistent outputs — the streaming
  approach deliberately avoids writing .sam to disk
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3930
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Aligns ONT long reads (-ax map-ont --MD) against the masked GRCh38+HLA reference,
    streaming SAM output into HLAminer via Unix pipe
  repo: https://github.com/lh3/minimap2
- name: HLAminer
  role: Receives streamed SAM alignments and scores HLA-I/II alleles in HPRA mode,
    emitting CSV summary and log
  repo: https://github.com/bcgsc/HLAminer
- name: HLAminer.pl
  role: Perl driver executed with '-a stream' HPRA parameters to process piped alignments
  repo: https://github.com/bcgsc/HLAminer
- name: TASR
  role: Builds the HLA reference sequences from IMGT/HLA repository via targeted assembly
    of known allele sequences
  repo: https://github.com/warrenlr/TASR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:a7e167ae8beade30db08a5478fd3fcb37dda069e35351fab61f84a92d7d72998
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HLAminer
  - HLAminer.pl
  - TASR
  - minimap2
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3930
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# long-read-hla-typing

## Summary

Predicts HLA class I and II allele groups from whole-genome shotgun long-read (ONT/PacBio) sequencing data by streaming minimap2 SAM alignments directly into HLAminer (HPRA mode), avoiding storage of bulky alignment files on disk.

## When to use

Use when you have long-read WGS FASTQ data (ONT PromethION or PacBio HiFi) from a sample with unknown HLA type and need group-level HLA-I/II predictions using a single Unix pipeline command.

## When NOT to use

- Input is short Illumina-sized reads — use HPTASR (targeted assembly) mode instead; BEST SHORT READ RESULTS ARE OBTAINED WITH HPTASR WITH READS 100bp AND UP
- The analysis requires full allele-level (4-digit) resolution validated clinically rather than group-level predictions — HPRA streaming reports group-level and top-scoring allele predictions that may be ambiguous
- Only bulky SAM/alignment files are wanted as persistent outputs — the streaming approach deliberately avoids writing .sam to disk

## Inputs

- Long-read WGS FASTQ reads (e.g. ONT PromethION NA12878 39x, SRR10965087 / ENA PRJEB26781-style accessions)
- Masked GRCh38 reference with appended HLA-I/II allele sequences (GCA_000001405.15_GRCh38_genomic.chr-only-chr6hlamasked-HLA-I_II_GEN.fa.gz)
- IMGT/HLA HLA-I_II_GEN.fasta allele reference database
- hla_nom_p.txt P-designation file from http://hla.alleles.org/wmda/hla_nom_p.txt

## Outputs

- HLAminer_HPRA.csv summary with top two highest-scoring HLA predictions per gene
- HLAminer_HPRA.log with the full mining process and all predictions ranked by score
- Group-level HLA-I (A, B, C) and HLA-II (DQA1, DQB1, DRB1, DPA1, DPB1, DRA) allele predictions

## How to apply

Build a GRCh38-based reference in which chr6 is masked and HLA-I/II allele sequences (from IMGT/HLA, assembled via TASR) are appended (e.g. GCA_000001405.15_GRCh38_genomic.chr-only-chr6hlamasked-HLA-I_II_GEN.fa.gz). Align FASTQ reads with minimap2 using '-t 60 -ax map-ont --MD' and pipe the SAM output directly into HLAminer.pl in HPRA streaming mode with '-h HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p hla_nom_p.txt -a stream', so the bulky .sam file is never written to disk. Collect the HLAminer_HPRA.csv summary (top two highest-scoring predictions per HLA gene, P-designated alleles where antigen binding domains are identical) and the HLAminer_HPRA.log listing all predictions by score. Compare predicted groups for A, B, C, DQA1, DQB1, DRB1, DPA1, DPB1, DRA against clinically confirmed types; the benchmark showed 94% (HLA-I) and 88% (HLA-II) concordance on NA12878 with ~2 hr 21 min total runtime.

## Related tools

- **minimap2** (Aligns ONT long reads (-ax map-ont --MD) against the masked GRCh38+HLA reference, streaming SAM output into HLAminer via Unix pipe) — https://github.com/lh3/minimap2
- **HLAminer** (Receives streamed SAM alignments and scores HLA-I/II alleles in HPRA mode, emitting CSV summary and log) — https://github.com/bcgsc/HLAminer
- **HLAminer.pl** (Perl driver executed with '-a stream' HPRA parameters to process piped alignments) — https://github.com/bcgsc/HLAminer
- **TASR** (Builds the HLA reference sequences from IMGT/HLA repository via targeted assembly of known allele sequences) — https://github.com/warrenlr/TASR

## Examples

```
minimap2 -t 60 -ax map-ont --MD GRCh38_chr6masked_HLA-I_II.fa reads.fastq.gz | HLAminer.pl -h HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p hla_nom_p.txt -a stream
```

## Evaluation signals

- HLAminer_HPRA.csv contains exactly two top-scoring predictions per HLA gene group (A, B, C, DQA1, DQB1, DRB1, DPA1, DPB1, DRA)
- For samples with known types, predicted groups match clinically derived HLA types — benchmark expectation is 94% concordance for HLA-I and 88% for HLA-II (e.g. NA12878)
- No bulky .sam file is created on disk — SAM output is consumed via the Unix pipe
- Total runtime is consistent with the benchmark (~2 hr 21 min for alignment plus prediction on a 39x ONT dataset with -t 60 threads)
- HLAminer_HPRA.log records the full mining process with all predictions ranked by highest score, confirming scoring completed for all nine gene groups

## Limitations

- Group-level predictions may not resolve 4-digit alleles; longer contigs better resolve HLA allelic variants (200 nt minimum contig filter applies to HPTASR route)
- Larger TASR k values yield predictions with increased specificity at the possible expense of sensitivity
- Tuned thresholds differ per mode: streamed long-read run used -s 500 -q 1 -i 1, whereas defaults are -s 1000, -q 30, -i 99 — mismatched parameters can change results
- blast 2.2.22 vs blast+ (2.2.28) can affect speed; older, less accurate nanopore chemistries may degrade accuracy

## Evidence

- [abstract] Stream minimap2 sequence alignments directly into HLAminer for HLA prediction: "predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer"
- [abstract] No SAM on disk: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [abstract] Robustness on older chemistry: "demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage"
- [methods] HPRA mode description: "The HLA prediction from direct read alignment (HPRA) method is conceptually simpler and faster to execute, since paired reads are aligned up-front to reference HLA alleles."
- [readme] v1.4 streaming support: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
- [readme] CSV summary behavior: "Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain."
