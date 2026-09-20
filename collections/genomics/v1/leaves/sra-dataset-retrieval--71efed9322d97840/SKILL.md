---
name: sra-dataset-retrieval
description: Use when a benchmark or reproduction requires the exact real RNA-Seq
  reads used in the rnaSPAdes/GigaScience 2019 paper, identified by the paper's SRA
  accessions (e.g., Human SRR5133163, Human large SRR1957703/SRR1957706).
when_to_use_negative:
- The downstream goal is metatranscriptome assembly — the article states the approach
  was initially designed for RNA-Seq from a single organism and metatranscriptome
  quality evaluation remains future work.
- The input data are long reads (Oxford Nanopore/PacBio) rather than short-read Illumina
  RNA-Seq — high error rate prevents their direct use as complete transcripts and
  they are outside this benchmark's design.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3308
tools:
- name: rnaSPAdes
  role: Downstream consumer of the retrieved paired-end FASTQ reads; its default k-mer
    schedule depends on read length, which motivates lossless FASTQ retrieval
  repo: github.com/ablab/spades
- name: rnaQUAST
  role: Downstream evaluation tool applied to assemblies of the retrieved datasets;
    per-dataset rnaQUAST metrics are keyed to the same accessions
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:b4f3d34d048d5edc0d12c5034b26deee8cfe8c5ae88d4d07bc45b2a988166d20
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - rnaQUAST
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sra-dataset-retrieval

## Summary

Retrieve the public paired-end Illumina RNA-Seq datasets used in the rnaSPAdes benchmark (Human, Human large, Mouse, Worm, Corn, Arabidopsis) from NCBI SRA in FASTQ format, using the exact accessions tabulated in the paper's methods section.

## When to use

Use when a benchmark or reproduction requires the exact real RNA-Seq reads used in the rnaSPAdes/GigaScience 2019 paper, identified by the paper's SRA accessions (e.g., Human SRR5133163, Human large SRR1957703/SRR1957706).

## When NOT to use

- The downstream goal is metatranscriptome assembly — the article states the approach was initially designed for RNA-Seq from a single organism and metatranscriptome quality evaluation remains future work.
- The input data are long reads (Oxford Nanopore/PacBio) rather than short-read Illumina RNA-Seq — high error rate prevents their direct use as complete transcripts and they are outside this benchmark's design.

## Inputs

- List of NCBI SRA accessions (SRR*/SRX*) from the paper's methods table: Human SRR5133163; Human large SRR1957703, SRR1957706; Mouse SRX648736; Worm SRR1560107; Corn SRR1588569; Arabidopsis SRR5344669, SRR5344670
- NCBI SRA archive (paired-end Illumina RNA-Seq reads)

## Outputs

- Paired-end FASTQ read files (mate 1 and mate 2) per accession
- Manifest mapping each SRR accession to organism label and downloaded FASTQ paths

## How to apply

Map each organism in the benchmark to its accession from the paper's methods table: Human SRR5133163; Human large SRR1957703 and SRR1957706; Mouse SRX648736 (an SRA experiment accession, resolve to runs); Worm SRR1560107; Corn SRR1588569; Arabidopsis SRR5344669 and SRR5344670. Download each run in FASTQ format with paired-end reads split into mate files, since rnaSPAdes and the subsequent rnaQUAST evaluation assume paired-end Illumina input. Note that rnaSPAdes default k-mer sizes are computed from read length (lower k ≈ read_length/3, min 29; upper k = largest odd number ≤ read_length/2 − 1), so preserve read-length information when fetching. Keep a manifest of accession → organism → file paths so downstream rnaQUAST table compilation can trace each assembly back to its dataset.

## Related tools

- **rnaSPAdes** (Downstream consumer of the retrieved paired-end FASTQ reads; its default k-mer schedule depends on read length, which motivates lossless FASTQ retrieval) — github.com/ablab/spades
- **rnaQUAST** (Downstream evaluation tool applied to assemblies of the retrieved datasets; per-dataset rnaQUAST metrics are keyed to the same accessions)

## Examples

```
fasterq-dump SRR5133163 --split-files --outdir reads/human
```

## Evaluation signals

- Each of the 6 benchmark datasets resolves to the exact accession(s) listed in the methods table, with Human large and Arabidopsis yielding two runs each and Mouse resolved from the SRX experiment accession to underlying runs.
- Downloaded files are FASTQ and paired-end, producing matched mate-1/mate-2 files with equal read counts per accession.
- Read lengths in the fetched FASTQs are consistent with the default k-mer assumptions (lower k ≈ read_length/3, min 29; upper k = largest odd ≤ read_length/2 − 1) needed for the subsequent rnaSPAdes runs.
- A manifest exists with one row per accession→organism mapping, enabling later compilation of rnaQUAST metrics (95%-assembled genes/isoforms, database coverage, misassemblies) into per-dataset CSV tables such as kmer_comparison.csv.

## Limitations

- Mouse is given as an SRX experiment accession rather than an SRR run accession, so an extra resolution step (experiment → runs) is required.
- No single assembler wins on all metrics and datasets, so the retrieved benchmark set must include all organisms for a faithful reproduction — omitting datasets changes which assembler leads.
- The paper notes assembler choice 'may require additional' benchmarks; dataset retrieval alone does not determine the winning tool.

## Evidence

- [intro] Benchmark datasets enumerated in the paper: "we assembled several simulated and publicly available real RNA-Seq datasets from the organisms"
- [methods] Human large accessions: "Human large: SRR1957703, SRR1957706"
- [supplementary] Default k-mer schedule tied to read length: [evidence span withheld — anchor status: not_found]
