---
name: rna-seq-read-simulation-with-rsem
description: Use when a benchmark requires a ground-truth RNA-Seq dataset with known
  transcript provenance, so assembler completeness (95%-assembled genes/isoforms),
  accuracy (misassemblies), and duplication ratio can be evaluated against a reference
  transcriptome.
when_to_use_negative:
- The goal is to benchmark on real data where transcript provenance is unknown — the
  article uses real datasets (SRR5133163, SRR1957703/SRR1957706, SRX648736, SRR1560107,
  SRR1588569, SRR5344669/SRR5344670) separately from the simulated ones
- The application is metatranscriptome assembly — the article states the approach
  'was initially designed for RNA-Seq data obtained from a single organism' and that
  metatranscriptomic quality evaluation is future work
- Only assembly contiguity or reference-free quality is of interest — the article
  warns that 'high database coverage or nucleotide recall can be achieved by a very
  fragmented assembly (or even raw reads)'
tools:
- name: RSEM
  role: Read simulator that generates the synthetic RNA-Seq dataset from a reference
    transcriptome (commands in Supplementary Section S3)
- name: rnaSPAdes
  role: Primary de novo transcriptome assembler under benchmark on the simulated reads
  repo: github.com/ablab/spades
- name: SPAdes
  role: Baseline genome assembler run in single-cell mode (k 21, 33, 55) on the simulated
    reads
  repo: github.com/ablab/spades
- name: Trinity
  role: Competing de novo assembler run with default settings on the simulated reads
- name: RNA-Bloom
  role: Competing de novo assembler run with default settings on the simulated reads
- name: Trans-ABySS
  role: Competing de novo assembler run with default settings on the simulated reads
- name: rnaQUAST
  role: Evaluates each assembly against the reference transcriptome, producing misassembly
    counts and duplication ratios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:c3b84d186c6f748aec0be04e0c95ef06085fe5ac8f53da9ab3b7e640a189c4ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinPacker
  - Bridger
  - IDBA-tran
  - RNA-Bloom
  - RSEM
  - SOAPdenovo-Trans
  - SPAdes
  - Trans-ABySS
  - Trinity
  - rnaQUAST
  - rnaSPAdes
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# rna-seq-read-simulation-with-rsem

## Summary

Generates synthetic RNA-Seq read sets with the RSEM read simulator using the exact command lines from Supplementary Section S3, producing controlled benchmark inputs (e.g., the Human simulated dataset) for de novo transcriptome assembler comparison. Simulated reads allow assembler output to be scored against a known reference transcriptome with rnaQUAST metrics such as 95%-assembled genes/isoforms, database coverage, misassemblies, and duplication ratio.

## When to use

Use when a benchmark requires a ground-truth RNA-Seq dataset with known transcript provenance, so assembler completeness (95%-assembled genes/isoforms), accuracy (misassemblies), and duplication ratio can be evaluated against a reference transcriptome.

## When NOT to use

- The goal is to benchmark on real data where transcript provenance is unknown — the article uses real datasets (SRR5133163, SRR1957703/SRR1957706, SRX648736, SRR1560107, SRR1588569, SRR5344669/SRR5344670) separately from the simulated ones
- The application is metatranscriptome assembly — the article states the approach 'was initially designed for RNA-Seq data obtained from a single organism' and that metatranscriptomic quality evaluation is future work
- Only assembly contiguity or reference-free quality is of interest — the article warns that 'high database coverage or nucleotide recall can be achieved by a very fragmented assembly (or even raw reads)'

## Inputs

- Reference transcriptome (e.g., human) with transcript sequences used by the RSEM simulator to generate reads
- RSEM read-simulator command lines from Supplementary Section S3 (simulation parameters, coverage, read length)

## Outputs

- Simulated RNA-Seq FASTQ read set (e.g., the Human simulated dataset)
- rnaQUAST evaluation reports per assembler containing 95%-assembled genes/isoforms, database coverage, misassemblies, and duplication ratio
- CSV benchmark table with one row per assembler

## How to apply

Run the RSEM read simulator with the exact command lines given in Supplementary Section S3 to produce a simulated dataset such as the Human simulated RNA-Seq dataset. Assemble the resulting reads de novo with each competing assembler (BinPacker, Bridger, IDBA-tran, RNA-Bloom, SOAPdenovo-Trans, Trans-ABySS, Trinity) under default settings, plus rnaSPAdes with default k-values and unmodified SPAdes in single-cell mode (k 21, 33, 55). Evaluate every assembly with rnaQUAST against the reference transcriptome and extract 95%-assembled genes/isoforms, database coverage, misassemblies, and duplication ratio. The rationale is that simulated reads carry known transcript origins, enabling objective reference-based scoring that real reads cannot provide; the article uses exactly this design to show rnaSPAdes leading with 0.50/0.32 fractions of 95%-assembled genes/isoforms while Trinity (1.74) and RNA-Bloom (1.93) show high duplication ratios.

## Related tools

- **RSEM** (Read simulator that generates the synthetic RNA-Seq dataset from a reference transcriptome (commands in Supplementary Section S3))
- **rnaSPAdes** (Primary de novo transcriptome assembler under benchmark on the simulated reads) — github.com/ablab/spades
- **SPAdes** (Baseline genome assembler run in single-cell mode (k 21, 33, 55) on the simulated reads) — github.com/ablab/spades
- **Trinity** (Competing de novo assembler run with default settings on the simulated reads)
- **RNA-Bloom** (Competing de novo assembler run with default settings on the simulated reads)
- **Trans-ABySS** (Competing de novo assembler run with default settings on the simulated reads)
- **rnaQUAST** (Evaluates each assembly against the reference transcriptome, producing misassembly counts and duplication ratios)

## Evaluation signals

- CSV row for rnaSPAdes on the simulated Human dataset reports 309 misassemblies (validation anchor from the source workflow)
- CSV row for Trans-ABySS on the simulated Human dataset reports 126 misassemblies
- Duplication ratios for the leading assemblers on the simulated Human dataset equal 1.74, 1.93, and 2.19
- rnaSPAdes shows the highest fraction of 95%-assembled genes (0.50) and isoforms (0.32), with Trinity (0.44, 0.30) and RNA-Bloom (0.41, 0.28) second and third
- Every assembler row contains all four metrics (95%-assembled genes/isoforms, database coverage, misassemblies, duplication ratio) extracted from rnaQUAST reports

## Limitations

- Simulated benchmarks do not capture real-dataset effects; the article notes no single assembler is the absolute leader across all quality metrics and all datasets (e.g., simulated Mouse is the only dataset where original SPAdes out-assembled rnaSPAdes)
- High database coverage or nucleotide recall can be achieved by very fragmented assemblies (or raw reads), so these metrics do not reflect assembly contiguity
- Metric differences between Detonate and rnaQUAST (cutoff on mapped fraction) mean contig recall and 95%-assembled isoforms do not perfectly correlate

## Evidence

- [supplementary] Generate the Human simulated RNA-Seq dataset by running the RSEM read simulator with the exact command lines from Supplementary Section S3.: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation anchors from the source workflow: [evidence span withheld — anchor status: not_found]
- [abstract] rnaSPAdes outperformance rationale: "rnaSPAdes typically outperforms other assemblers by such important property as the number of assembled genes and isoforms, and at the same time has higher accuracy statistics on average"
- [discussion] Fragmented-assembly caveat for coverage metrics: "high database coverage or nucleotide recall can be achieved by a very fragmented assembly (or even raw reads)"
