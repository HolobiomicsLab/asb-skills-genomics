---
name: sra-dataset-retrieval
description: Use when a plant tncRNA or small-RNA analysis needs raw single-end sRNA-seq
  data from public archives, and the run must be downloaded from NCBI SRA as FASTQ
  before adapter/quality trimming.
when_to_use_negative:
- The reads are already available locally as FASTQ — proceed directly to Trim Galore
  filtering rather than re-downloading from SRA.
- The required input is paired-end or non-small-RNA (e.g. mRNA-seq) data; the article's
  pipeline consumes single-end small RNA reads.
- The goal is annotated tncRNA exploration rather than raw data acquisition — use
  the tncRNA database (http://www.nipgr.ac.in/tncRNA) or tRex/PtRFdb instead.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NCBI SRA Toolkit
  role: Downloads raw single-end small RNA-seq runs from NCBI SRA and converts them
    to FASTQ for the Trim Galore filtering stage.
- name: Trim Galore
  role: 'Downstream consumer: its v0.6.6 adapter/quality trimming defines the expected
    raw single-end FASTQ input produced by this retrieval step.'
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:cc7c38e7df1ca5fff675f5ed66571f64435767ad81dd79641275bf0987829453
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/FelixKrueger/TrimGalore
  tools:
  - NCBI SRA Toolkit
  - Trim Galore
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sra-dataset-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves raw single-end small RNA-seq FASTQ files from NCBI's Sequence Read Archive (SRA) using the SRA Toolkit, providing the raw reads that enter the tncRNA Toolkit pipeline's first filtering stage (Trim Galore v0.6.6 adapter/quality trimming).

## When to use

Use when a plant tncRNA or small-RNA analysis needs raw single-end sRNA-seq data from public archives, and the run must be downloaded from NCBI SRA as FASTQ before adapter/quality trimming.

## When NOT to use

- The reads are already available locally as FASTQ — proceed directly to Trim Galore filtering rather than re-downloading from SRA.
- The required input is paired-end or non-small-RNA (e.g. mRNA-seq) data; the article's pipeline consumes single-end small RNA reads.
- The goal is annotated tncRNA exploration rather than raw data acquisition — use the tncRNA database (http://www.nipgr.ac.in/tncRNA) or tRex/PtRFdb instead.

## Inputs

- NCBI SRA run accession(s) or study accession for single-end small RNA-seq samples
- SRA-format archive files (.sra) or direct SRA URLs
- Sample metadata (species, tissue, stress condition) for the 2,448-sample angiosperm compendium

## Outputs

- Raw single-end sRNA-seq FASTQ file(s) (.fastq/.fq) per run, suitable as input to Trim Galore adapter/quality trimming
- Download/provenance log mapping each FASTQ to its SRA accession and sample metadata

## How to apply

Identify the sRNA-seq accessions of interest from SRA (the article drew on 2,448 samples across six angiosperms: Arabidopsis thaliana, Medicago truncatula, Solanum lycopersicum, Cicer arietinum, rice, and maize). Download each run with the NCBI SRA Toolkit, converting SRA files to single-end FASTQ (.fastq/.fq), which is the input format the downstream Trim Galore v0.6.6 filtering stage expects. Validate that each downloaded file parses as valid single-end FASTQ and contains raw (untrimmed) reads with 3' Illumina adapter and low-quality bases still present, since trimming is deliberately deferred to the next skill. Keep provenance (accession, sample/tissue metadata) alongside each FASTQ so filtered reads can be traced back to SRA entries when supplied to the tncRNA Toolkit.

## Related tools

- **NCBI SRA Toolkit** (Downloads raw single-end small RNA-seq runs from NCBI SRA and converts them to FASTQ for the Trim Galore filtering stage.)
- **Trim Galore** (Downstream consumer: its v0.6.6 adapter/quality trimming defines the expected raw single-end FASTQ input produced by this retrieval step.) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
fastq-dump --gzip SRR0000000
```

## Evaluation signals

- Each downloaded output file exists, is non-empty, and parses as valid FASTQ with 4-line records.
- Reads are single-end (one FASTQ per run), matching the pipeline's expectation of single-end small RNA reads.
- Reads are raw: 3' Illumina adapter sequence and low-quality trailing bases are still detectable, since trimming is the next stage (Trim Galore v0.6.6).
- Every FASTQ is traceable to an SRA accession and its sample metadata (species/tissue) from the SRA record.
- The set of retrieved runs covers the intended samples (e.g. the six angiosperm species analyzed in the study).

## Limitations

- Conventional sRNA-seq library prep carries tRNA modification biases; the article notes that switching to tRNA-seq would be advantageous for identifying tncRNAs, so SRA-derived reads may underrepresent modified tRNA regions.
- The provided context does not specify the SRA Toolkit version, prefetch/fastq-dump options, or the full accession list, so exact download parameters must be chosen by the practitioner.
- Retrieval only yields raw reads; tncRNA identification additionally requires the tncRNA Toolkit's genome fasta, bowtie1 index, and downstream filtering (e.g. tRNAscanSE score ≥ 50).

## Evidence

- [other] Raw single-end sRNA FASTQ files downloaded from NCBI SRA with the SRA Toolkit: [evidence span withheld — anchor status: not_found]
