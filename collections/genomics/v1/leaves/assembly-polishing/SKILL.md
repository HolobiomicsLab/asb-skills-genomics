---
name: assembly-polishing
description: Decision support for choosing between polishing a raw long-read assembly
  vs. correcting reads before assembly, and for executing CONSENT-polish on Miniasm/Flye
  contigs built from raw ONT or PacBio reads.
when_to_use_negative:
- The goal is to correct the reads themselves rather than an assembly — use CONSENT-correct
  (--type ONT/PB) instead of the polishing mode
- The contigs were assembled from reads that have already been error-corrected, so
  a further polishing pass on already-corrected input is not the target comparison
- Only short reads are available — this skill addresses third-generation (PacBio/ONT)
  long-read assemblies, not short-read assembly polishing
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: CONSENT
  role: Provides the polishing mode (CONSENT-polish) that corrects raw long-read assemblies
    via MSA consensus plus local de Bruijn graph polishing; also offers CONSENT-correct
    for the correct-then-assemble comparison
  repo: https://github.com/morispi/CONSENT
- name: Flye
  role: Assembles the corrected reads in the correct-then-assemble arm of the pipeline
    comparison; CONSENT correction is reported to improve Flye assembly quality
- name: Miniasm
  role: Assembles the raw ONT reads to produce the raw assembly (rawAssembly.fasta)
    that serves as polishing input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:d72d8b92d51cbdc14f63a07d33361130d13c67a398d706396227784569e77a83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Flye
  - Miniasm
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# assembly-polishing

## Summary

Use CONSENT's assembly polishing mode to correct contigs assembled from raw long reads (e.g., Miniasm ONT assemblies) with multiple sequence alignments of overlapping reads plus local de Bruijn graph polishing, instead of the more expensive correct-then-assemble pipeline. On a full human ONT dataset this halves runtime and memory while yielding better assembly metrics.

## When to use

Decision support for choosing between polishing a raw long-read assembly vs. correcting reads before assembly, and for executing CONSENT-polish on Miniasm/Flye contigs built from raw ONT or PacBio reads.

## When NOT to use

- The goal is to correct the reads themselves rather than an assembly — use CONSENT-correct (--type ONT/PB) instead of the polishing mode
- The contigs were assembled from reads that have already been error-corrected, so a further polishing pass on already-corrected input is not the target comparison
- Only short reads are available — this skill addresses third-generation (PacBio/ONT) long-read assemblies, not short-read assembly polishing

## Inputs

- Raw contigs assembled from uncorrected long reads (FASTA/FASTQ, e.g., Miniasm rawAssembly.fasta)
- Raw long reads used for polishing support (FASTA/FASTQ, e.g., longReads.fastq, ONT or PacBio technology)
- Optional: reference genome for reference-based assembly metric computation (NGA50, NGA75, errors per 100 kbp, misassemblies)

## Outputs

- Polished assembly (polishedAssembly.fasta) with corrected contig sequences
- Reference-based assembly metrics table (NGA50, NGA75, genome coverage, errors per 100 kbp, misassembly counts)
- Runtime and peak-memory measurements for the polishing pipeline (e.g., consent_pipeline_comparison.tsv)

## How to apply

First assemble raw long reads (e.g., Miniasm for the polishing pipeline, or Flye for the correct-then-assemble comparison) without prior error correction. Run CONSENT-polish, supplying the raw contigs file, the raw long reads in FASTA/FASTQ, and an output path; CONSENT computes overlaps, builds MSA piles of supporting reads per contig window (maxSupport default 150), derives window consensus sequences, then corrects remaining errors with a local de Bruijn graph at the window scale (solid k-mer threshold default 4). Evaluate the polished assembly against the reference using NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts, and record runtime and peak RAM. The rationale is that CONSENT is 2-38x faster than other polishing tools, and on the full human dataset the assemble-then-polish route (7.5 days, <400 GB RAM) beats correction+assembly (>17 days, 1 TB RAM) in both cost and quality.

## Related tools

- **CONSENT** (Provides the polishing mode (CONSENT-polish) that corrects raw long-read assemblies via MSA consensus plus local de Bruijn graph polishing; also offers CONSENT-correct for the correct-then-assemble comparison) — https://github.com/morispi/CONSENT
- **Flye** (Assembles the corrected reads in the correct-then-assemble arm of the pipeline comparison; CONSENT correction is reported to improve Flye assembly quality)
- **Miniasm** (Assembles the raw ONT reads to produce the raw assembly (rawAssembly.fasta) that serves as polishing input)

## Examples

```
./CONSENT-polish --contigs rawAssembly.fasta --reads longReads.fastq --out polishedAssembly.fasta
```

## Evaluation signals

- Polished assembly shows higher NGA50/NGA75, lower errors per 100 kbp, and fewer misassemblies than the raw pre-polish assembly when evaluated against the reference
- On the full human ONT dataset, the assemble-then-polish pipeline completes in about 7.5 days using <400 GB RAM, versus >17 days and 1 TB RAM for correction then assembly
- Runtime is consistent with the reported 2-38x speed advantage of CONSENT over other polishing tools at comparable result quality
- Output is a valid FASTA of polished contigs whose total contig content corresponds to the input raw assembly rather than a new read set
- The side-by-side comparison table (e.g., consent_pipeline_comparison.tsv) reproduces the qualitative finding that assemble-then-polish is cheaper and yields better assembly metrics

## Limitations

- CONSENT's strategy of only selecting the N highest identity overlaps might prove inefficient, especially when the length of the repeated regions grows longer
- CONSENT's assembly was observed to cover a smaller proportion of the reference sequence, and unaligned contigs were not yet analyzed in detail
- CONSENT has been developed and tested on x86-64 GNU/Linux only; support for other platforms has not been tested (README)
- The corresponding correction step (if comparing pipelines) requires substantial resources, including ~5 TB disk space on a full human dataset driven by the overlaps file

## Evidence

- [abstract] CONSENT implements a polishing feature, allowing to correct raw assemblies.: "CONSENT implements a polishing feature, allowing to correct raw assemblies."
- [abstract] Assemble-then-polish is cheaper and better than correct-then-assemble on human data: "on a human dataset, assembling the raw data and polishing the assembly is less resource consuming than correcting and then assembling the reads, while providing better results."
- [abstract] CONSENT polishing speed advantage over other tools: "CONSENT is 2-38x times faster than other polishing tools, while providing comparable results."
- [discussion] Local de Bruijn graph polishing step: "it is further polished with the help of a local de Bruijn graph, at the scale of the window"
- [readme] README polishing command with example inputs: "./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta"
