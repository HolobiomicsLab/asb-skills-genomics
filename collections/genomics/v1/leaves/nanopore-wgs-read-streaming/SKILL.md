---
name: nanopore-wgs-read-streaming
description: Use when a whole-genome shotgun long-read FASTQ (e.g. Oxford Nanopore
  PromethION WGS) needs HLA class I/II allele prediction, and disk/time-efficient
  single-command streaming of aligner output into HLAminer is preferred over writing
  SAM files to disk.
when_to_use_negative:
- Input is short Illumina-sized reads — use the HPTASR targeted-assembly or HPRA read-alignment
  modes instead (best short-read results with reads 100bp and up, ideally 150bp)
- Predictions from transcriptome/RNA-seq data are the goal and a read aligner other
  than minimap2 must be used — the skill's minimap2-to-HLAminer pipe applies to WGS
  long-read DNA workflows; any SAM-compatible aligner works but this benchmark used
  minimap2
- Holding the full SAM alignment file is required for downstream reuse — streaming
  is specifically designed to avoid writing bulky alignment files to disk, so the
  SAM output is not retained
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns raw ONT long reads against the GRCh38-noChr6-HLA-I_II_GEN reference
    with map-ont preset and --MD; its SAM output is piped directly into HLAminer and
    dominates pipeline runtime and RAM
  repo: https://github.com/lh3/minimap2
- name: HLAminer v1.4
  role: Receives streamed SAM alignments via './HLAminer.pl ... -a stream' and predicts
    HLA class I/II alleles without saving intermediate alignment files
  repo: https://github.com/bcgsc/HLAminer
- name: HLAminer.pl
  role: Perl driver invoked in stream mode (-a stream) with HLA-I_II_GEN.fasta database
    (-h), score (-s 500), expect (-q 1), identity (-i 1) and hla_nom_p.txt parameters
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:c34eec603e28433457c2adf6d0defff400850c6d53b468493e47d30bbe1de0dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HLAminer v1.4
  - HLAminer.pl
  - minimap2
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nanopore-wgs-read-streaming

## Summary

Stream minimap2 SAM alignments of ONT PromethION whole-genome shotgun reads directly into HLAminer v1.4 via a Unix pipe, eliminating bulky intermediate SAM files while predicting HLA class I and II alleles from ~10x coverage long-read data.

## When to use

Use when a whole-genome shotgun long-read FASTQ (e.g. Oxford Nanopore PromethION WGS) needs HLA class I/II allele prediction, and disk/time-efficient single-command streaming of aligner output into HLAminer is preferred over writing SAM files to disk.

## When NOT to use

- Input is short Illumina-sized reads — use the HPTASR targeted-assembly or HPRA read-alignment modes instead (best short-read results with reads 100bp and up, ideally 150bp)
- Predictions from transcriptome/RNA-seq data are the goal and a read aligner other than minimap2 must be used — the skill's minimap2-to-HLAminer pipe applies to WGS long-read DNA workflows; any SAM-compatible aligner works but this benchmark used minimap2
- Holding the full SAM alignment file is required for downstream reuse — streaming is specifically designed to avoid writing bulky alignment files to disk, so the SAM output is not retained

## Inputs

- Raw ONT PromethION WGS read FASTQ (e.g. ERR2585115.fastq.gz, ~10x coverage NA19240, ENA PRJEB26791)
- GRCh38 reference FASTA with chromosome 6 removed and HLA class I/II sequences appended (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa)
- IMGT/HLA database files: HLA-I_II_GEN.fasta and hla_nom_p.txt (P designation file)
- HLAminer v1.4 installation (./bin, ./databases, ./docs, ./test-demo)

## Outputs

- HLA class I and II allele predictions at group and allele resolution (HLAminer_HPTASR.csv / HLAminer_HPRA.csv style summary with .log of all predictions)
- GNU time profile file containing 'Elapsed (wall clock) time' and 'Maximum resident set size' for the streamed pipeline

## How to apply

Wrap minimap2 and HLAminer.pl in a single piped command: minimap2 maps the ONT FASTQ against the GRCh38 chr-only reference with chromosome 6 removed and HLA-I_II_GEN sequences added (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN), using -t 60 -ax map-ont --MD, and the SAM output streams directly into './HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream' (v1.4). Use -x map-ont presets appropriate for raw uncorrected nanopore reads; the -a stream mode invokes the streaming support added in HLAminer v1.4 so no .sam file is ever written. For benchmarking, prefix the pipeline with '/usr/bin/time -v -o <file>.time' and parse the 'Elapsed (wall clock) time' and 'Maximum resident set size' entries. The NA19240 PromethION dataset (ERR2585115, ~10x coverage, 2018 older chemistry, ENA project PRJEB26791) serves as the reference benchmark: expect approximately 40 minutes wall-clock and peak RAM within the reported up-to-35-GB envelope, with minimap2 dominating both runtime and RAM. Runtime scales with the number of reads, so larger/deeper datasets will proportionally exceed the 40-minute benchmark.

## Related tools

- **minimap2** (Aligns raw ONT long reads against the GRCh38-noChr6-HLA-I_II_GEN reference with map-ont preset and --MD; its SAM output is piped directly into HLAminer and dominates pipeline runtime and RAM) — https://github.com/lh3/minimap2
- **HLAminer v1.4** (Receives streamed SAM alignments via './HLAminer.pl ... -a stream' and predicts HLA class I/II alleles without saving intermediate alignment files) — https://github.com/bcgsc/HLAminer
- **HLAminer.pl** (Perl driver invoked in stream mode (-a stream) with HLA-I_II_GEN.fasta database (-h), score (-s 500), expect (-q 1), identity (-i 1) and hla_nom_p.txt parameters)

## Examples

```
/usr/bin/time -v -o minimap_hlaminerERR2585115-1mod.time minimap2 -t 60 -ax map-ont --MD GRCh38-noChr6-HLA-I_II_GEN.fa ERR2585115.fastq.gz | ./HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream
```

## Evaluation signals

- GNU time file reports wall-clock 'Elapsed (wall clock) time' of approximately 40 minutes for the ~10x coverage ERR2585115 dataset, matching the protocol's Table 2 benchmark
- Peak 'Maximum resident set size' stays within the reported up-to-35-GB envelope
- No bulky .sam file is written to disk — the SAM output exists only in the pipe between minimap2 and HLAminer.pl
- HLAminer summary CSV reports top two highest-scoring predictions per HLA group, consistent with v1.3+ output convention
- Runtime scales with read count; a deeper dataset should show proportionally longer wall-clock time, with minimap2 dominating resource use

## Limitations

- Benchmark uses a 2018 older-chemistry PromethION dataset; predictions are robust at ~10x coverage but accuracy depends on read quality of that chemistry generation
- Peak RAM can reach up to 35 GB, dominated by minimap2, so commodity hardware with <2 GB RAM (sufficient for HPTASR short-read mode) is not adequate for the streaming WGS long-read workflow
- Runtime and RAM scale with the number of sequences; datasets deeper or larger than the benchmarked ~10x NA19240 data will exceed the 40-minute / 35-GB figures

## Evidence

- [abstract] Stream minimap2 SAM alignments directly into HLAminer for HLA prediction: "predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer"
- [abstract] No disk SAM storage: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [methods] Streaming support in v1.4: "HLAminer v1.4 provides initial support for HLA prediction from raw uncorrected shotgun nanopore long reads"
- [results] Benchmark runtime claim: [evidence span withheld — anchor status: not_found]
- [readme] Streamed SAM output into HLAminer (README): "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
