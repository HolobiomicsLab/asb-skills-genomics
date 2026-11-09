---
name: ont-dataset-acquisition
description: Use when an analysis requires raw ONT long-read WGS FASTQ inputs (e.g.,
  NA24385 guppy v5.0.6 ~67x or dorado kit14-2022-12 ~72x, NA19240 PromethION ~10x)
  to reproduce or benchmark HLA prediction workflows.
when_to_use_negative:
- The task needs short-read (Illumina-sized) data — HPTASR is the targeted-assembly
  route for 100-150bp reads, not ONT WGS FASTQ acquisition
- The analysis requires freshly base-called signal (POD5/FAST5) rather than published
  FASTQ base-calls
- The target individual lacks clinically confirmed HLA types, so the acquisition cannot
  support concordance validation as described
tools:
- name: wget
  role: Background download of ENA/SRA FASTQ files for ONT WGS runs (ERR2585112, ERR2585115)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:18f8eedaee278083e8a99552929cb8dff71abc9b6212df1852fb3a30e93ec2d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - wget
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# ont-dataset-acquisition

## Summary

Acquire Oxford Nanopore whole-genome shotgun FASTQ datasets (PromethION, clinically typed individuals such as NA19240 and NA24385) from public repositories (ENA/SRA/epi2me) as input for long-read HLA prediction with HLAminer.

## When to use

Use when an analysis requires raw ONT long-read WGS FASTQ inputs (e.g., NA24385 guppy v5.0.6 ~67x or dorado kit14-2022-12 ~72x, NA19240 PromethION ~10x) to reproduce or benchmark HLA prediction workflows.

## When NOT to use

- The task needs short-read (Illumina-sized) data — HPTASR is the targeted-assembly route for 100-150bp reads, not ONT WGS FASTQ acquisition
- The analysis requires freshly base-called signal (POD5/FAST5) rather than published FASTQ base-calls
- The target individual lacks clinically confirmed HLA types, so the acquisition cannot support concordance validation as described

## Inputs

- ENA/SRA accession identifiers for ONT PromethION WGS runs (e.g., ERR2585112, ERR2585115; ENA PRJEB26791 for NA19240)
- epi2me-hosted NA24385 ONT WGS FASTQ downloads (guppy v5.0.6 and dorado kit14-2022-12 base-calls)

## Outputs

- Raw ONT long-read FASTQ files (uncompressed .fastq or .fastq.gz) ready for minimap2 alignment
- Coverage/chemistry provenance note (e.g., ~67x guppy v5.0.6, ~72x dorado kit14-2022-12) recorded per dataset

## How to apply

Identify the target individual and base-caller chemistry (e.g., dataset D = NA24385 guppy v5.0.6 at ~67x; dataset E = NA24385 dorado kit14-2022-12 at ~72x; dataset B = NA19240 older 2018 PromethION chemistry). Download the FASTQ files from ENA/SRA using the FTP paths for the accession (e.g., ERR2585112, ERR2585115) with wget in the background, or follow the epi2me download instructions for the NA24385 PromethION datasets. Verify the downloaded FASTQ is complete and matches the expected coverage class (10x-72x range described in the article) before feeding it into minimap2 -ax map-ont alignment. Rationale: coverage drives HLAminer prediction quality — high-coverage PromethION datasets (39x, 67x, 72x) yield class I and II predictions that corroborate clinically typed counterparts, while older 2018 chemistry still works at ~10x.

## Related tools

- **wget** (Background download of ENA/SRA FASTQ files for ONT WGS runs (ERR2585112, ERR2585115))

## Examples

```
nohup wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR258/002/ERR2585112/ERR2585112.fastq.gz &
```

## Evaluation signals

- Downloaded FASTQ files are complete (non-zero size, decompressible) and correspond to the specified ENA accessions (ERR2585112, ERR2585115)
- Coverage class matches the article's expectations: ~67x (dataset D, guppy v5.0.6) and ~72x (dataset E, dorado kit14-2022-12) for NA24385
- Each dataset is associated with the correct provenance (base-caller version and chemistry) recorded for downstream comparison
- The acquired datasets belong to individuals with clinically confirmed HLA types (e.g., NA19240, NA24385), enabling concordance validation

## Limitations

- Older datasets (2018 NA19240 PromethION chemistry) are less accurate, though HLAminer predictions remain robust even at ~10x coverage
- Coverage matters: low-coverage datasets may produce ambiguous HLA predictions; the article's corroborated results come from high-coverage PromethION runs (39x, 67x, 72x)

## Evidence

- [methods] ENA FTP download commands: "nohup wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR258/002/ERR2585112/ERR2585112.fastq.gz &"
- [methods] NA19240 dataset provenance: "We tested the software on the NA19240 WGS promethion dataset (2018, older chemistry)."
- [other] NA24385 dataset acquisition via epi2me: [evidence span withheld — anchor status: not_found]
- [abstract] Coverage robustness finding: "demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage"
