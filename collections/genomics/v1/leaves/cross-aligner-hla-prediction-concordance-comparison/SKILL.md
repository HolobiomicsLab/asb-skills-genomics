---
name: cross-aligner-hla-prediction-concordance-comparison
description: Use when evaluating whether a SAM-with-MD-tag output from any long-read
  aligner can replace minimap2 in a streamed HLAminer.pl pipeline, or when validating
  aligner-agnostic concordance of HLA allele-group predictions on WGS nanopore data.
when_to_use_negative:
- The input is short-read (Illumina-sized) data better suited to HPTASR targeted assembly
  rather than streamed long-read SAM alignments.
- The alternative aligner cannot emit SAM with MD tags, which the streamed HPRA mode
  relies on.
- The sample lacks clinically confirmed HLA types or a minimap2 baseline, so concordance
  cannot be judged.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3050
tools:
- name: HLAminer v1.4
  role: Receives streamed SAM alignments (-a stream mode) and produces HLAminer_HPRA.csv/log
    HLA predictions
  repo: https://github.com/bcgsc/HLAminer
- name: HLAminer.pl
  role: Command-line entry point invoked in the piped streaming command with -h, -s
    500, -q 1, -i 1, -p, -a stream options
  repo: https://github.com/bcgsc/HLAminer
- name: minimap2
  role: Baseline long-read aligner (run with '-t 60 -ax map-ont --MD') whose predictions
    serve as the reference for concordance comparison
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:fd6dad367eab5dda5bc432eb9d88095a74831ed0ffc0c4f46a679fa1aa10fb70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer v1.4
  - HLAminer.pl
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cross-aligner-hla-prediction-concordance-comparison

## Summary

Tests whether HLAminer v1.4's streaming HPRA protocol is aligner-agnostic by substituting a non-minimap2 SAM-producing long-read aligner (with MD tags) in the piped command and comparing predicted HLA allele groups against the minimap2 baseline and clinically confirmed NA19240 types.

## When to use

Use when evaluating whether a SAM-with-MD-tag output from any long-read aligner can replace minimap2 in a streamed HLAminer.pl pipeline, or when validating aligner-agnostic concordance of HLA allele-group predictions on WGS nanopore data.

## When NOT to use

- The input is short-read (Illumina-sized) data better suited to HPTASR targeted assembly rather than streamed long-read SAM alignments.
- The alternative aligner cannot emit SAM with MD tags, which the streamed HPRA mode relies on.
- The sample lacks clinically confirmed HLA types or a minimap2 baseline, so concordance cannot be judged.

## Inputs

- NA19240 PromethION WGS nanopore FASTQ reads (e.g. ERR2585115, ENA PRJEB26791)
- GRCh38 reference without chr6 HLA region (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz)
- IMGT/HLA reference database (HLA-I_II_GEN.fasta) and P designation file (hla_nom_p.txt)
- minimap2-baseline HLAminer_HPRA.csv predictions for dataset A (groups A26 and A33, alleles A*26:33 and A*33:24)

## Outputs

- HLAminer_HPRA.csv (ranked per-gene HLA allele-group prediction table, top two predictions per gene)
- HLAminer_HPRA.log (full prediction listing)
- Concordance comparison of predicted allele groups vs minimap2 baseline and reported NA19240 HLA-I types

## How to apply

Download the NA19240 PromethION WGS nanopore FASTQ (e.g. ERR2585115 from ENA PRJEB26791) and align reads to the prepared GRCh38 reference lacking chr6 HLA regions (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz), substituting minimap2 (originally run with '-t 60 -ax map-ont --MD') with an alternative aligner whose SAM output includes MD tags. Stream the SAM directly into HLAminer.pl via a Unix pipe using '-h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream', avoiding intermediate .sam files on disk. Collect HLAminer_HPRA.csv and HLAminer_HPRA.log, which contain the ranked per-gene predictions keeping the top two highest-scoring predictions per gene. The run is accepted if the CSV parses as a ranked per-gene table and the predicted allele groups are concordant with both the minimap2 baseline groups and the reported NA19240 types (e.g. A*30/A*68, B*35/B*57, C*04/C*18).

## Related tools

- **HLAminer v1.4** (Receives streamed SAM alignments (-a stream mode) and produces HLAminer_HPRA.csv/log HLA predictions) — https://github.com/bcgsc/HLAminer
- **HLAminer.pl** (Command-line entry point invoked in the piped streaming command with -h, -s 500, -q 1, -i 1, -p, -a stream options) — https://github.com/bcgsc/HLAminer
- **minimap2** (Baseline long-read aligner (run with '-t 60 -ax map-ont --MD') whose predictions serve as the reference for concordance comparison) — https://github.com/lh3/minimap2

## Examples

```
alternative_aligner --MD -x ont GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz ERR2585115.fastq.gz | ../bin/HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream
```

## Evaluation signals

- HLAminer_HPRA.csv exists and parses as a ranked per-gene prediction table (top two highest-scoring predictions per gene, P-designated alleles where antigen-binding domains are identical).
- Predicted HLA-I allele groups from the alternative-aligner run match the minimap2 baseline groups (dataset A baseline: A26 and A33, alleles A*26:33 and A*33:24).
- Predicted groups agree with the clinically reported NA19240 types (e.g. A*30/A*68, B*35/B*57, C*04/C*18).
- No intermediate .sam files were written to disk — the SAM output was consumed directly through the Unix pipe.
- The .log file lists all predictions and confirms the stream mode was invoked with the expected parameters.

## Limitations

- Robustness was demonstrated on older, less accurate WGS nanopore (2018 PromethION chemistry) datasets at relatively low (~10x) coverage, so newer/other chemistries or coverage regimes are not directly benchmarked.
- The protocol article benchmarks minimap2 specifically; substituting another aligner is a hypothesis ('works with any read aligner compatible with the SAM file format') that must be validated by concordance, not assumed.
- Predictions are at group resolution in the v1.3+ summary CSV; 4-digit allele resolution can remain ambiguous for some data types.

## Evidence

- [abstract] works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [intro] we adapted HLAminer (v1.4) to predict HLA alleles from any long sequence reads, by streaming sequence alignment outputs directly into it: "we adapted HLAminer (v1.4) to predict HLA alleles from any long sequence reads, by streaming sequence alignment outputs directly into it"
- [readme] Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer.: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
- [readme] Keeps top two [highest-scoring by HLA group] predictions per gene: "Keeps top two [highest-scoring by HLA group] predictions per gene"
- [methods] Results below demonstrate HLAminer's ability to fairly accurately predict HLA-I and -II types from direct [and streamed] nanopore sequencing reads [old chemistry] alignments: "Results below demonstrate HLAminer's ability to fairly accurately predict HLA-I and -II types from direct [and streamed] nanopore sequencing reads [old chemistry] alignments"
