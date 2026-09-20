---
name: long-read-alignment-streaming-hla-prediction
description: Use when HLA-I/II allele predictions are needed from raw uncorrected
  long nanopore or PacBio WGS reads, especially at low (~10x) coverage, without storing
  bulky alignment files.
when_to_use_negative:
- Input is Illumina-sized short reads where targeted assembly (HPTASR) with 100-150bp
  reads is the recommended route
- Reads require alignment against a reference that has not been supplemented with
  up-to-date IMGT/HLA allele sequences
- A .sam alignment file must be retained as a deliverable, since streaming bypasses
  on-disk SAM storage
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_0102
tools:
- name: HLAminer
  role: HLAminer v1.4 scores streamed SAM alignments and predicts HLA class I/II groups
    and 4-digit alleles via its HPRA mode
  repo: https://github.com/bcgsc/HLAminer
- name: minimap2
  role: Read aligner run with '-ax map-ont --MD' to produce SAM alignments of ONT
    reads streamed directly into HLAminer
  repo: https://github.com/lh3/minimap2
- name: TASR
  role: Used in the HPTASR alternative mode to build k-mer hash tables from IMGT/HLA
    references and targeted-assemble recruited reads
  repo: https://github.com/warrenlr/TASR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:e376b0fc084b874191239cc273884810e153ab686251b0b16e240620b58c4c11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - TASR
  - minimap2
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# long-read-alignment-streaming-hla-prediction

## Summary

Predict HLA class I and II alleles from whole-genome shotgun long-read (ONT/PacBio) sequencing data by streaming minimap2 SAM alignment output directly into HLAminer v1.4, eliminating on-disk SAM files and enabling single-command prediction at group and allele resolution.

## When to use

Use when HLA-I/II allele predictions are needed from raw uncorrected long nanopore or PacBio WGS reads, especially at low (~10x) coverage, without storing bulky alignment files.

## When NOT to use

- Input is Illumina-sized short reads where targeted assembly (HPTASR) with 100-150bp reads is the recommended route
- Reads require alignment against a reference that has not been supplemented with up-to-date IMGT/HLA allele sequences
- A .sam alignment file must be retained as a deliverable, since streaming bypasses on-disk SAM storage

## Inputs

- FASTQ whole-genome shotgun long-read nanopore files (e.g. ERR2585112, ERR2585115 from ENA PRJEB26791)
- Combined GRCh38 (GCA_000001405.15) chr-only-noChr6 reference with HLA-I and HLA-II allele sequences (HLA-I_II_GEN.fasta), indexed for minimap2
- hla_nom_p.txt P-designation file from hla.alleles.org

## Outputs

- HLAminer_HPRA_<run_id>.csv with concise HLA-I/II allele summary (top two highest-scoring predictions per gene)
- HLAminer_HPRA_<run_id>.log listing all predictions
- Streamed SAM alignments (piped, not stored on disk)

## How to apply

Download the WGS FASTQ reads (e.g. ERR2585112/ERR2585115 from ENA PRJEB26791) and build a combined GRCh38 reference (chr-only, no chr6) augmented with updated IMGT/HLA class I/II allele sequences, indexed for minimap2. Run minimap2 with '-t 60 -ax map-ont --MD' against this reference and pipe the SAM output directly into HLAminer.pl using '-a stream' so no .sam file is written to disk. Supply HLAminer.pl with the HLA-I_II_GEN.fasta database and relaxed long-read thresholds (-s 500, -q 1, -i 1) plus the hla_nom_p.txt P-designation file; larger TASR k values increase specificity at possible expense of sensitivity if using the HPTASR mode instead. Collect HLAminer_HPRA_<run>.csv and .log outputs, which keep the top two highest-scoring predictions per HLA group and only the 'P' designated allele when sequences share the same antigen binding domain.

## Related tools

- **HLAminer** (HLAminer v1.4 scores streamed SAM alignments and predicts HLA class I/II groups and 4-digit alleles via its HPRA mode) — https://github.com/bcgsc/HLAminer
- **minimap2** (Read aligner run with '-ax map-ont --MD' to produce SAM alignments of ONT reads streamed directly into HLAminer) — https://github.com/lh3/minimap2
- **TASR** (Used in the HPTASR alternative mode to build k-mer hash tables from IMGT/HLA references and targeted-assemble recruited reads) — https://github.com/warrenlr/TASR

## Examples

```
minimap2 -t 60 -ax map-ont --MD GRCh38.chr-only-noChr6-HLA-I_II_GEN.fa ERR2585115.fastq.gz | ../bin/HLAminer.pl -a stream -d ../databases/HLA-I_II_GEN.fasta -p hla_nom_p.txt -s 500 -q 1 -i 1
```

## Evaluation signals

- Run completes within ~40 minutes for the 10x coverage NA19240 ONT dataset when timed with /usr/bin/time -v
- HLAminer_HPRA CSV predictions report heterozygous HLA-A groups A26/A33 with alleles A*26:33 and A*33:24, matching clinically confirmed types
- No .sam file is written to disk when '-a stream' is used (streaming pipe verified)
- CSV summary keeps top two highest-scoring predictions per gene and only P-designated alleles where antigen binding domains are identical
- HLAminer .log file lists all predictions, not just the summarized top hits

## Limitations

- Predictions from older, less accurate nanopore chemistry (e.g. 2018 PromethION) at ~10x coverage are robust but may be less precise; 4-digit calls can be ambiguous
- blast 2.2.22 may be faster than blast+ (2.2.28) while producing accurate results — tool version choice affects performance
- Larger k values in TASR-based assembly increase specificity at the possible expense of sensitivity
- The approach relies on continually updated IMGT/HLA reference databases; no changelog tracking of database updates was available

## Evidence

- [abstract] Streaming avoids SAM storage: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [abstract] Robustness at low coverage: "demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage"
- [methods] Streaming into HLAminer: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
- [methods] Relaxed long-read thresholds: "In our hands, a few tests show that blast 2.2.22 may be faster than blast+ (2.2.28) while producing accurate results"
- [readme] Output summarization rule: "Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain."
- [methods] Nanopore support in v1.4: "HLAminer v1.4 provides initial support for HLA prediction from raw uncorrected shotgun nanopore long reads"
