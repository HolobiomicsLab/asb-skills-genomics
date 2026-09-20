---
name: nanopore-wgs-dataset-acquisition
description: Use when a workflow needs the raw nanopore WGS reads (e.g. ENA accessions
  ERR2585112/ERR2585115 from PRJEB26791, NA19240 PromethION) as FASTQ input for alignment-based
  HLA-I/II prediction or similar long-read benchmarking.
when_to_use_negative:
- The reads are already present locally or the archive provides a dedicated download
  client/API — re-fetching via FTP wget is unnecessary.
- The task is running the HLA prediction itself (minimap2 → HLAminer.pl stream) rather
  than obtaining the input FASTQ data.
- The benchmark requires a different sequencing technology's data, e.g. PacBio HiFi
  WGS or Illumina short reads, which are separate test datasets in the protocol.
edam_operation: http://edamontology.org/operation_2422
tools:
- name: wget
  role: Background (nohup) FTP download of the ENA-hosted gzipped FASTQ run files
    by accession
- name: minimap2
  role: Downstream consumer of the acquired FASTQ reads; alignment uses -x map-ont
    settings appropriate for the ONT data
  repo: lh3/minimap2
- name: HLAminer
  role: Downstream HLA-I/II predictor used to test that the acquired nanopore WGS
    dataset yields robust predictions despite older chemistry and ~10x coverage
  repo: https://github.com/bcgsc/HLAminer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:acbc042fa268752e8c9ce2857f0acbe885a92e3b21b293feaa8455b7c4d968b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HLAminer
  - minimap2
  - wget
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# nanopore-wgs-dataset-acquisition

## Summary

Acquires raw Oxford Nanopore PromethION whole-genome shotgun FASTQ datasets from public archives (ENA) by accession, as required to benchmark the streamed minimap2-to-HLAminer HLA prediction protocol on datasets with clinically confirmed HLA types.

## When to use

Use when a workflow needs the raw nanopore WGS reads (e.g. ENA accessions ERR2585112/ERR2585115 from PRJEB26791, NA19240 PromethION) as FASTQ input for alignment-based HLA-I/II prediction or similar long-read benchmarking.

## When NOT to use

- The reads are already present locally or the archive provides a dedicated download client/API — re-fetching via FTP wget is unnecessary.
- The task is running the HLA prediction itself (minimap2 → HLAminer.pl stream) rather than obtaining the input FASTQ data.
- The benchmark requires a different sequencing technology's data, e.g. PacBio HiFi WGS or Illumina short reads, which are separate test datasets in the protocol.

## Inputs

- ENA study accession (PRJEB26791, NA19240 PromethION WGS)
- Run accessions to download (e.g. ERR2585112, ERR2585115)
- ENA fastq FTP path pattern (ftp.sra.ebi.ac.uk/vol1/fastq/<prefix>/<dir>/<accession>/<accession>.fastq.gz)

## Outputs

- Gzipped nanopore FASTQ files (e.g. ERR2585112.fastq.gz, ERR2585115.fastq.gz) ready for minimap2 map-ont alignment

## How to apply

Identify the ENA/SRA study accession (here PRJEB26791, the NA19240 PromethION WGS dataset) and the run accessions to download (ERR2585112 and ERR2585115 in the protocol). Fetch the gzipped FASTQ files directly from the EBI fastq FTP volume paths (ftp.sra.ebi.ac.uk/vol1/fastq/ERR258/...) with wget, launched with nohup in the background since the files are large. Note the provenance: the 2018 PromethION data use older, less accurate chemistry and provide only ~10x coverage, which is the point of the benchmark — predictions are expected to be robust even under these conditions. Once downloaded, the fastq.gz files feed directly into minimap2 alignment ('-ax map-ont --MD') against the combined GRCh38 chr-only-noChr6 HLA-I/II reference.

## Related tools

- **wget** (Background (nohup) FTP download of the ENA-hosted gzipped FASTQ run files by accession)
- **minimap2** (Downstream consumer of the acquired FASTQ reads; alignment uses -x map-ont settings appropriate for the ONT data) — lh3/minimap2
- **HLAminer** (Downstream HLA-I/II predictor used to test that the acquired nanopore WGS dataset yields robust predictions despite older chemistry and ~10x coverage) — https://github.com/bcgsc/HLAminer

## Examples

```
nohup wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR258/005/ERR2585115/ERR2585115.fastq.gz &
```

## Evaluation signals

- Both run files (ERR2585112, ERR2585115) download from the correct ENA vol1/fastq volume paths without truncation
- Downloaded files pass gzip integrity checks (gunzip -t) and are readable FASTQ
- Read lengths and total base yield are consistent with PromethION WGS at roughly 10x coverage
- Dataset provenance traces to ENA PRJEB26791 (NA19240, 2018 PromethION, older chemistry) as described in the protocol

## Limitations

- The NA19240 PromethION dataset is from 2018 with older, less accurate nanopore chemistry and only ~10x coverage — adequate for robust HLA group prediction but not a high-accuracy chemistry benchmark
- Only FASTQ from ENA fastq volumes is fetched; no BAM/basecalled signal data, so the skill cannot supply reads if the fastq volume path structure changes at EBI

## Evidence

- [methods] EN PRJEB26791 (NA19240 PromethION WGS): "We tested the software on the NA19240 WGS promethion dataset (2018, older chemistry)."
- [methods] ERR2585115 fastq download command: "nohup wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR258/005/ERR2585115/ERR2585115.fastq.gz &"
- [abstract] robustness rationale for choosing this dataset: "demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage"
- [methods] downstream use of the downloaded reads: [evidence span withheld — anchor status: not_found]
