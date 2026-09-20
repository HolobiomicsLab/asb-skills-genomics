---
name: assembly-contiguity-metric-extraction
description: Use when assemblies built from raw, corrected, or polished long reads
  (Miniasm or Flye) must be compared via reference-based metrics — NGA50, NGA75, genome
  coverage, errors per 100 kbp, misassemblies — e.
when_to_use_negative:
- No reference genome is available for the dataset — NGA50, NGA75, genome coverage,
  and errors per 100 kbp are reference-based QUAST-LG metrics and cannot be computed.
- Input is raw or corrected long reads rather than assembled contigs — metrics are
  per-assembly, so reads must first go through Miniasm or Flye assembly.
- The goal is to assess the error rate of the corrected reads themselves rather than
  the downstream assembly quality that correction confers.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST-LG
  role: Reference-based evaluator run on every assembly (raw, corrected-read, polished)
    to produce contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, and
    misassembly counts
- name: Minimap2
  role: Computes the overlaps feeding the Miniasm overlap-layout-consensus assembly
    of raw and corrected benchmark reads
- name: Miniasm
  role: Assembles raw reads and each tool's corrected reads for the three real benchmark
    datasets into contigs slated for QUAST-LG evaluation
- name: Flye
  role: Assembles raw and corrected H. sapiens chromosome 1 reads; its NGA50/NGA75
    deltas reveal whether correction improves assembly quality
- name: CONSENT
  role: Supplies the corrected reads (--type PB or --type ONT) and the CONSENT-polish
    polishing step whose assemblies are compared; correct-then-assemble vs assemble-then-polish
    resource use is measured on its runs
  repo: https://github.com/morispi/CONSENT
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:75d30a7bde41c8cd2332ff5cb9f57f6b43a17bc84f1625efac776fb6bc9514c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Flye
  - Miniasm
  - Minimap2
  - QUAST-LG
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# assembly-contiguity-metric-extraction

## Summary

Quantifies assembly contiguity and accuracy for long-read assembly comparisons: raw, CONSENT-corrected, and polished assemblies are evaluated with QUAST-LG against a reference genome to extract contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts. This lets a practitioner decide which error-correction tool (or correction-vs-polishing pipeline order) actually improves assembly quality.

## When to use

Use when assemblies built from raw, corrected, or polished long reads (Miniasm or Flye) must be compared via reference-based metrics — NGA50, NGA75, genome coverage, errors per 100 kbp, misassemblies — e.g. to pick an error-correction tool or compare correct-then-assemble vs assemble-then-polish pipelines.

## When NOT to use

- No reference genome is available for the dataset — NGA50, NGA75, genome coverage, and errors per 100 kbp are reference-based QUAST-LG metrics and cannot be computed.
- Input is raw or corrected long reads rather than assembled contigs — metrics are per-assembly, so reads must first go through Miniasm or Flye assembly.
- The goal is to assess the error rate of the corrected reads themselves rather than the downstream assembly quality that correction confers.

## Inputs

- Assembled contigs in FASTA format from each condition (raw-read, corrected-read, and polished assemblies from Miniasm or Flye)
- Long reads in FASTA/FASTQ format (PacBio or ONT) used for assembly and polishing
- Reference genome for each dataset (e.g. H. sapiens chromosome 1) for reference-based QUAST-LG evaluation

## Outputs

- Per-assembly QUAST-LG metrics: contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, misassembly counts (reproducing Table 3)
- Side-by-side pipeline comparison table (consent_pipeline_comparison.tsv) with quality metrics plus runtime and peak RAM for correct-then-assemble vs assemble-then-polish (reproducing Table 6)

## How to apply

First produce the contigs to compare: assemble raw reads and each correction tool's corrected reads (CONSENT-correct with --type PB for PacBio or --type ONT for Oxford Nanopore data) with the Minimap2+Miniasm overlap-layout-consensus pipeline for the benchmark datasets, and assemble raw vs corrected H. sapiens chromosome 1 reads with Flye. For the pipeline trade-off, contrast correct-then-assemble (CONSENT-correct then Flye) against assemble-then-polish (Miniasm assembly of raw reads followed by CONSENT-polish with --contigs rawAssembly.fasta --reads longReads.fastq --out polishedAssembly.fasta). Evaluate every resulting assembly against its reference genome with QUAST-LG and tabulate contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts (Table 3 format); for the pipeline comparison also record runtime and peak memory and write consent_pipeline_comparison.tsv. The rationale is that correction tools are judged by downstream assembly quality, not by the corrected reads alone: in this benchmark only the assembly from CONSENT-corrected reads reached larger NGA50 and NGA75 than the raw-read Flye assembly, and assembling raw data then polishing was both cheaper and better on the full human dataset.

## Related tools

- **QUAST-LG** (Reference-based evaluator run on every assembly (raw, corrected-read, polished) to produce contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts)
- **Minimap2** (Computes the overlaps feeding the Miniasm overlap-layout-consensus assembly of raw and corrected benchmark reads)
- **Miniasm** (Assembles raw reads and each tool's corrected reads for the three real benchmark datasets into contigs slated for QUAST-LG evaluation)
- **Flye** (Assembles raw and corrected H. sapiens chromosome 1 reads; its NGA50/NGA75 deltas reveal whether correction improves assembly quality)
- **CONSENT** (Supplies the corrected reads (--type PB or --type ONT) and the CONSENT-polish polishing step whose assemblies are compared; correct-then-assemble vs assemble-then-polish resource use is measured on its runs) — https://github.com/morispi/CONSENT

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB
```

## Evaluation signals

- Reproducing Table 3: for H. sapiens chr 1 Flye assemblies, only the CONSENT-corrected-read assembly must reach larger NGA50 and NGA75 than the raw-read assembly.
- The metrics table must contain, per assembly: contig count, NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly count.
- Reproducing Table 6: assemble-then-polish must yield higher NGA50 and lower errors per 100 kbp and misassemblies than correct-then-assemble, with runtime 7.5 days vs >17 days and peak RAM <400 GB vs >1 TB.
- Each assembly must be evaluated against its correct matching reference genome — a mispaired reference invalidates NGA50, coverage, and error counts.
- Runtime and peak-memory measurements must be recorded for both pipelines (not only quality metrics) to support the resource-usage conclusion.

## Limitations

- The correct-then-assemble route on the full human dataset required more than 17 days and 1 TB of RAM (versus 7.5 days and <400 GB for assemble-then-polish), so exhaustive metric extraction at that scale is costly.
- Unaligned contigs were not analyzed in detail, and CONSENT assemblies cover a smaller proportion of the reference sequence, so coverage metrics may understate contig content.
- CONSENT's correction step runtime tends to be higher than other state-of-the-art methods, and the overlaps file drives a 5 TB disk requirement on a full human dataset.

## Evidence

- [abstract] CONSENT correction improves Flye assembly quality: "our experiments show that error correction with CONSENT improves the quality of Flye assemblies."
- [abstract] Assemble-then-polish is cheaper and better on human data: "on a human dataset, assembling the raw data and polishing the assembly is less resource consuming than correcting and then assembling the reads, while providing better results."
- [discussion] Only CONSENT scales to ONT ultra-long reads: "CONSENT is the only method able to efficiently scale to the correction of ONT ultra-long reads"
- [readme] CONSENT also polishes raw assemblies: "Additionally to error correction, CONSENT can also perform assembly polishing."
