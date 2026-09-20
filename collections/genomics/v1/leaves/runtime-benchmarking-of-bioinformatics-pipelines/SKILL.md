---
name: runtime-benchmarking-of-bioinformatics-pipelines
description: Use when executing or reproducing a published pipeline (e.g. streamed
  minimap2-to-HLAminer HLA typing) and you need to verify that wall-clock runtime
  scales as reported for a given dataset, such as ~40 minutes for 10x ONT WGS.
when_to_use_negative:
- The dataset has drastically different coverage or read chemistry (e.g. PacBio HiFi
  instead of older-chemistry ONT), so published runtimes like ~40 minutes at 10x are
  not comparable baselines.
- The goal is per-read alignment speed of minimap2 alone (e.g. aligner benchmarking),
  rather than end-to-end runtime of the piped alignment-into-prediction protocol.
- The pipeline is short-read Illumina-based HPTASR assembly, whose runtime profile
  and stages differ from the streamed long-read HPRA workflow.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Long-read aligner run with '-t 60 -ax map-ont --MD' whose SAM output is streamed
    into HLAminer; its runtime is the benchmarked stage.
  repo: https://github.com/lh3/minimap2
- name: HLAminer
  role: Downstream HLA-I/II prediction tool (v1.4, HPRA mode with '-a stream') that
    consumes the streamed SAM; overall protocol runtime and prediction outputs are
    benchmarked together.
  repo: https://github.com/bcgsc/HLAminer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:66d258c0e189f2edeae704479a6a78da5eea14b832f6ff95b5558eac47c8c107
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HLAminer
  - minimap2
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# runtime-benchmarking-of-bioinformatics-pipelines

## Summary

Measure wall-clock runtime and resource usage of a multi-stage bioinformatics protocol (e.g. streaming minimap2 alignments into HLAminer) using /usr/bin/time -v, so runtimes can be compared against published expectations and dataset coverage.

## When to use

Use when executing or reproducing a published pipeline (e.g. streamed minimap2-to-HLAminer HLA typing) and you need to verify that wall-clock runtime scales as reported for a given dataset, such as ~40 minutes for 10x ONT WGS.

## When NOT to use

- The dataset has drastically different coverage or read chemistry (e.g. PacBio HiFi instead of older-chemistry ONT), so published runtimes like ~40 minutes at 10x are not comparable baselines.
- The goal is per-read alignment speed of minimap2 alone (e.g. aligner benchmarking), rather than end-to-end runtime of the piped alignment-into-prediction protocol.
- The pipeline is short-read Illumina-based HPTASR assembly, whose runtime profile and stages differ from the streamed long-read HPRA workflow.

## Inputs

- FASTQ nanopore WGS reads (e.g. ERR2585115 from ENA PRJEB26791, NA19240 PromethION)
- Pre-built minimap2-indexed combined reference GRCh38 (GCA_000001405.15, chr-only-noChr6) with HLA-I and -II allele sequences from IMGT/HLA
- HLA-I_II_GEN.fasta HLAminer database
- hla_nom_p.txt P-designation file from http://hla.alleles.org/wmda/hla_nom_p.txt

## Outputs

- Wall-clock time and memory statistics from /usr/bin/time -v for the alignment/streaming stage
- HLAminer_HPRA_<run>.csv summary of top two highest-scoring HLA predictions per gene
- HLAminer_HPRA_<run>.log listing all scored predictions

## How to apply

Wrap the compute-intensive stage of the pipeline (here, minimap2 aligning ONT fastq reads with '-t 60 -ax map-ont --MD' against a GRCh38 chr-only-noChr6 HLA-I_II_GEN reference) in '/usr/bin/time -v' to capture wall-clock time and peak memory while the SAM output is streamed via '-a stream' directly into HLAminer.pl (with '-s 500 -q 1 -i 1' relaxed thresholds for long reads). Record the runtime for a dataset with known coverage (e.g. ERR2585115, 10x PromethION WGS from NA19240) and compare it to the published expectation of ~40 minutes. Correctness is judged by the run completing within the expected time window AND producing the expected downstream outputs (HLAminer_HPRA_*.csv/.log with top HLA-A predictions matching clinically confirmed allele groups, e.g. A26/A33). Stream rather than writing .sam to disk, since streaming eliminates bulky intermediate alignment files and is part of the benchmarked protocol.

## Related tools

- **minimap2** (Long-read aligner run with '-t 60 -ax map-ont --MD' whose SAM output is streamed into HLAminer; its runtime is the benchmarked stage.) — https://github.com/lh3/minimap2
- **HLAminer** (Downstream HLA-I/II prediction tool (v1.4, HPRA mode with '-a stream') that consumes the streamed SAM; overall protocol runtime and prediction outputs are benchmarked together.) — https://github.com/bcgsc/HLAminer

## Examples

```
/usr/bin/time -v minimap2 -t 60 -ax map-ont --MD GRCh38.chr-only-noChr6-HLA-I_II_GEN.fa ERR2585115.fastq.gz | ../bin/HLAminer.pl -a stream -d HLA-I_II_GEN.fasta -p hla_nom_p.txt -s 500 -q 1 -i 1
```

## Evaluation signals

- The minimap2 command, timed with /usr/bin/time -v, completes in approximately ~40 minutes on the 10x coverage ERR2585115 dataset.
- No intermediate .sam file is written to disk — SAM is piped via '-a stream', confirming the streaming configuration was used.
- Output files HLAminer_HPRA_ERR2585115.csv and .log are produced with the expected naming and content schema.
- The CSV top predictions for HLA-A match the clinically confirmed allele groups A26/A33 (alleles A*26:33 and A*33:24), showing correct runtime AND correct functional behavior.
- Predictions remain robust despite older, less accurate ONT chemistry at low (10x) coverage, consistent with the article's reported robustness finding.

## Limitations

- Runtime expectations (~40 min at 10x) are specific to older-chemistry (2018) ONT PromethION data and 60-thread minimap2 settings; other datasets will scale differently.
- The protocol was benchmarked on one PacBio HiFi and four ONT PromethION WGS datasets from three individuals with clinically confirmed HLA types; results for other platforms may vary.
- Relaxed HPRA thresholds for long reads ('-s 500', '-q 1', '-i 1' versus defaults -s 1000, -q 30, -i 99) trade specificity for sensitivity, which may affect prediction confidence alongside runtime.

## Evidence

- [abstract] demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage: "demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage"
- [abstract] works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [methods] HLAminer fairly accurately predicts HLA-I and -II types from direct and streamed nanopore sequencing read alignments: "HLAminer's ability to fairly accurately predict HLA-I and -II types from direct [and streamed] nanopore sequencing reads [old chemistry] alignments"
- [intro] we utilized one PacBio HiFi and four ONT PromethION whole-genome shotgun datasets derived from three individuals with known, clinically confirmed HLA types: "we utilized one PacBio HiFi and four ONT PromethION whole‐genome shotgun datasets derived from three individuals with known, clinically confirmed HLA types"
- [readme] Ability to stream the (.sam) output of modern read aligners, directly into HLAminer.: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
