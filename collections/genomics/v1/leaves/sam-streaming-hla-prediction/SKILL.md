---
name: sam-streaming-hla-prediction
description: Use when HLA-I/II allele predictions are needed from long-read WGS data
  (PacBio HiFi or ONT PromethION) with clinically confirmed HLA types available for
  validation, and disk space or runtime favors streaming over writing SAM files.
when_to_use_negative:
- Input is short Illumina-sized reads — the HPTASR targeted-assembly mode (reads 100bp+,
  ideally 150bp) is the recommended route for short reads, not streamed long-read
  HPRA
- Only HLA class I alleles are needed from targeted RNA-seq or exome capture data
  with low depth — the original HPTASR or direct-alignment short-read workflows may
  be more appropriate
- No HLA reference database update or chr6-masked GRCh38 reference is available and
  one cannot be prepared, since alignments to endogenous chr6 would confound scoring
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3534
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Aligns long reads to the chr6-masked GRCh38+HLA reference using -ax map-hifi
    or map-ont presets with --MD; SAM output is piped into HLAminer and dominates
    pipeline runtime and RAM
  repo: https://github.com/lh3/minimap2
- name: HLAminer v1.4
  role: Receives streamed SAM alignments in HPRA mode (-a stream) and scores candidate
    HLA alleles, emitting the CSV summary and log
  repo: https://github.com/bcgsc/HLAminer
- name: HLAminer.pl
  role: Perl driver invoked with -h HLA-I_II_GEN.fasta, -s 500, -q 1, -i 1, -p hla_nom_p.txt,
    -a stream to process streamed alignments
  repo: https://github.com/bcgsc/HLAminer
- name: TASR
  role: Used in the HLAminer database-preparation path to read IMGT/HLA reference
    sequences; relevant to HPTASR mode rather than streaming HPRA
  repo: https://github.com/warrenlr/TASR
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:3159bd7bb56144308bcc38fb22caccf072f7389f263628472f26bed50da4ba79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HLAminer v1.4
  - HLAminer.pl
  - TASR
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3534
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# sam-streaming-hla-prediction

## Summary

Predicts HLA class I and II alleles from whole-genome shotgun long-read sequencing data (PacBio HiFi or ONT) by streaming minimap2 SAM alignment output directly into HLAminer v1.4 (HPRA mode), avoiding bulky intermediate SAM files on disk.

## When to use

Use when HLA-I/II allele predictions are needed from long-read WGS data (PacBio HiFi or ONT PromethION) with clinically confirmed HLA types available for validation, and disk space or runtime favors streaming over writing SAM files.

## When NOT to use

- Input is short Illumina-sized reads — the HPTASR targeted-assembly mode (reads 100bp+, ideally 150bp) is the recommended route for short reads, not streamed long-read HPRA
- Only HLA class I alleles are needed from targeted RNA-seq or exome capture data with low depth — the original HPTASR or direct-alignment short-read workflows may be more appropriate
- No HLA reference database update or chr6-masked GRCh38 reference is available and one cannot be prepared, since alignments to endogenous chr6 would confound scoring

## Inputs

- Long-read WGS FASTQ files (PacBio CCS/HiFi subreads or ONT PromethION reads, e.g. ERR2585115.fastq.gz)
- GRCh38 (GCA_000001405.15) genomic reference with chr6 excluded/masked plus HLA-I/II genomic alleles (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz)
- HLA-I_II_GEN.fasta HLA reference database from IMGT/HLA
- hla_nom_p.txt P designation file (http://hla.alleles.org/wmda/hla_nom_p.txt)
- Clinically confirmed HLA types for the sample (validation)

## Outputs

- HLAminer_HPRA.csv — top two highest-scoring HLA predictions per gene, P-designated alleles where antigen-binding domains are identical
- HLAminer_HPRA.log — command, alignments, and scores for all predictions
- GNU time profile file (e.g. minimap_hlaminerERR2585115-1mod.time) with elapsed wall-clock time and maximum resident set size

## How to apply

Build a GRCh38-based reference (GCA_000001405.15) with all chromosomes but with chr6 removed or masked, plus HLA-I/II genomic alleles from IMGT/HLA, so reads map to HLA references rather than endogenous chr6. Align reads with minimap2 using the appropriate preset (-ax map-hifi for PacBio CCS, -ax map-ont for nanopore) with --MD, and pipe the SAM output directly into HLAminer.pl in HPRA mode with -a stream, -h ../database/HLA-I_II_GEN.fasta, -s 500, -q 1, -i 1, and -p ../database/hla_nom_p.txt (P designation file), so no bulky .sam files are written. Collect HLAminer_HPRA.csv, which keeps the top two highest-scoring predictions per gene and only the P-designated allele where antigen-binding domains are identical, plus HLAminer_HPRA.log with full command, alignments, and scores. Validate by comparing ranked predictions against clinically confirmed HLA types (e.g. NA24385: B*35:01 predicted vs B*35:08 clinical; DRB1*04:03P vs DRB1*04:02P) and by checking runtime against benchmarks (~40 min for 10x ONT data, ~1 hr 14 min for 30x PacBio HiFi), optionally wrapping the command in GNU time -v to capture elapsed wall-clock time and maximum resident set size.

## Related tools

- **minimap2** (Aligns long reads to the chr6-masked GRCh38+HLA reference using -ax map-hifi or map-ont presets with --MD; SAM output is piped into HLAminer and dominates pipeline runtime and RAM) — https://github.com/lh3/minimap2
- **HLAminer v1.4** (Receives streamed SAM alignments in HPRA mode (-a stream) and scores candidate HLA alleles, emitting the CSV summary and log) — https://github.com/bcgsc/HLAminer
- **HLAminer.pl** (Perl driver invoked with -h HLA-I_II_GEN.fasta, -s 500, -q 1, -i 1, -p hla_nom_p.txt, -a stream to process streamed alignments) — https://github.com/bcgsc/HLAminer
- **TASR** (Used in the HLAminer database-preparation path to read IMGT/HLA reference sequences; relevant to HPTASR mode rather than streaming HPRA) — https://github.com/warrenlr/TASR

## Examples

```
/usr/bin/time -v -o minimap_hlaminerERR2585115-1mod.time minimap2 -t 60 -ax map-ont --MD <GRCh38-noChr6-HLA-I_II_GEN.fa> ERR2585115.fastq.gz | ./HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream
```

## Evaluation signals

- HLAminer_HPRA.csv contains the top two highest-scoring predictions per HLA gene, with P-designated alleles where antigen-binding domains are identical
- Predicted alleles match or are P-equivalent to clinically confirmed types (e.g. B*35:01 predicted vs B*35:08 clinical; DRB1*04:03P vs DRB1*04:02P for NA24385)
- Wall-clock runtime matches reported benchmarks: ~40 min for the ~10x ONT NA19240 dataset (ERR2585115) and ~1 hr 14 min for 30x PacBio HiFi NA24385, confirming minimap2 dominates runtime
- Peak RAM (maximum resident set size from GNU time -v) stays within the reported up-to-35-GB envelope
- No bulky .sam files are written to disk — SAM output is streamed via pipe directly into HLAminer.pl

## Limitations

- Predictions may differ from clinical typing at fine allele resolution (e.g. B*35:01 vs B*35:08, DRB1*04:03P vs DRB1*04:02P), so P-designated (antigen-binding-domain-identical) alleles are reported
- Runtime scales with read count and peak RAM can reach up to 35 GB; minimap2 dominates both, so resource budgeting is needed for high-coverage inputs
- Larger k values for TASR assembly yield predictions with increased specificity at possible expense of sensitivity (relevant to the HPTASR mode sharing the same scoring software)
- Older/less accurate nanopore chemistry at low (~10x) coverage is supported but predictions are described as robust rather than definitive

## Evidence

- [abstract] a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer: "a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer"
- [abstract] works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [methods] The HLA prediction from direct read alignment (HPRA) method is conceptually simpler and faster to execute, since paired reads are aligned up-front to reference HLA alleles.: "The HLA prediction from direct read alignment (HPRA) method is conceptually simpler and faster to execute, since paired reads are aligned up-front to reference HLA alleles."
- [readme] Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain.: "Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain."
- [results] Executing the streaming minimap2-HLAminer workflow on the 10x ONT dataset A (NA19240, ERR2585115) took 40 minutes: [evidence span withheld — anchor status: not_found]
