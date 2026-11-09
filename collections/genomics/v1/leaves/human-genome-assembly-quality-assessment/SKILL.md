---
name: human-genome-assembly-quality-assessment
description: Use when a de novo human or large-genome assembly (e.g., from HASLR on
  PacBio + Illumina reads) must be validated for repeat content and contiguity/accuracy
  before or alongside comparison to published assemblies.
when_to_use_negative:
- The assembly is of a small or non-mammalian genome (e.g., the E. coli, yeast, or
  C. elegans datasets in the HASLR evaluation) where a mammalian repeat library census
  is not the relevant quality axis
- The goal is read-level QC or assembly of raw long reads rather than assessing a
  completed assembly FASTA
- Phased or haplotype-resolved assessment is required — the current HASLR version
  does not generate a phased assembly for diploid genomes
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0622
tools:
- name: HASLR
  role: Hybrid assembler producing the assembly FASTA to be assessed (asm.final.fa)
  repo: https://github.com/vpc-ccg/haslr
- name: RepeatMasker
  role: Annotates repeat classes (SINE, LINE, LTR, DNA-transposon, simple repeats,
    low-complexity) and reports counts and span fractions
- name: Minia
  role: HASLR dependency that assembles the Illumina short reads into contigs used
    in the hybrid assembly
  repo: https://github.com/GATB/minia
- name: SPOA
  role: Consensus calling of long reads during assembly, affecting base-level accuracy
    (indel rate) measured in the assessment
  repo: https://github.com/rvaser/spoa
- name: minimap2
  role: Aligns short-read contigs onto long reads during assembly, shaping contiguity
    metrics
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:5cf0684ce72a0ed16d9f818099573b7299ef0bc027a3bd3a11e14b15f4b1d6dd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/haslr
  repo_url: https://github.com/vpc-ccg/haslr
  tools:
  - HASLR
  - Minia
  - RepeatMasker
  - SPOA
  - minimap2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# human-genome-assembly-quality-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the quality and biological content of a human genome assembly (e.g., HASLR's CHM1 PacBio+Illumina hybrid assembly) by running RepeatMasker against a curated mammalian repeat library and parsing per-family element counts and genomic span fractions (SINE, LINE, LTR, DNA-transposon, simple repeats, low-complexity).

## When to use

Use when a de novo human or large-genome assembly (e.g., from HASLR on PacBio + Illumina reads) must be validated for repeat content and contiguity/accuracy before or alongside comparison to published assemblies.

## When NOT to use

- The assembly is of a small or non-mammalian genome (e.g., the E. coli, yeast, or C. elegans datasets in the HASLR evaluation) where a mammalian repeat library census is not the relevant quality axis
- The goal is read-level QC or assembly of raw long reads rather than assessing a completed assembly FASTA
- Phased or haplotype-resolved assessment is required — the current HASLR version does not generate a phased assembly for diploid genomes

## Inputs

- Final assembly FASTA (e.g., asm.final.fa from HASLR on CHM1 PacBio + Illumina reads)
- Curated mammalian repeat library for RepeatMasker
- Reference assembly or published figures for comparison (e.g., reported CHM1 repeat census)

## Outputs

- RepeatMasker .tbl summary of per-class element counts and span fractions
- RepeatMasker .out per-element annotation file
- Parsed per-family census (e.g., 1,519,699 SINEs at 13.22% span; 922,706 LINEs at 21.73%; 485,530 LTRs at 9.21%)
- Assembly quality metrics (misassemblies, genome fraction, NGA50, indel rate)

## How to apply

Assemble the genome first — for the CHM1 human dataset run HASLR's wrapper (haslr.py -g ~3g -l chm1_pacbio.fastq -x pacbio -s illumina_1.fastq illumina_2.fastq) with 64 CPU threads, producing asm.final.fa. Then run RepeatMasker on the assembly contigs against the curated mammalian repeat library to classify SINE, LINE, LTR, DNA-transposon, simple-repeat, and low-complexity elements. Parse the RepeatMasker .tbl and .out outputs to extract per-family element counts and the fraction of genomic span occupied by each class; for CHM1 the reported anchors are 1,519,699 SINEs (13.22%), 922,706 LINEs (21.73%), and 485,530 LTRs (9.21%). Accept the assessment if element counts match reported figures and span fractions fall within a small tolerance (e.g., ±1% relative). Complement the repeat census with assembly metrics such as misassemblies, genome fraction, NGA50, and indel rate when comparing assemblers, since HASLR was evaluated on these axes and is known to sometimes have lower genome fraction/NGA50 (especially versus Canu on human) and a higher indel rate.

## Related tools

- **HASLR** (Hybrid assembler producing the assembly FASTA to be assessed (asm.final.fa)) — https://github.com/vpc-ccg/haslr
- **RepeatMasker** (Annotates repeat classes (SINE, LINE, LTR, DNA-transposon, simple repeats, low-complexity) and reports counts and span fractions)
- **Minia** (HASLR dependency that assembles the Illumina short reads into contigs used in the hybrid assembly) — https://github.com/GATB/minia
- **SPOA** (Consensus calling of long reads during assembly, affecting base-level accuracy (indel rate) measured in the assessment) — https://github.com/rvaser/spoa
- **minimap2** (Aligns short-read contigs onto long reads during assembly, shaping contiguity metrics) — https://github.com/lh3/minimap2

## Examples

```
haslr.py -g ~3g -l chm1_pacbio.fastq -x pacbio -s illumina_1.fastq illumina_2.fastq -t 64 && RepeatMasker -lib mammalian_repeat_library.fasta asm.final.fa
```

## Evaluation signals

- Per-class element counts match reported CHM1 figures (1,519,699 SINEs, 922,706 LINEs, 485,530 LTRs)
- Span fractions fall within ±1% relative of reported values: 13.22% SINE, 21.73% LINE, 9.21% LTR
- RepeatMasker .tbl parses without error and sums of class spans are consistent with total assembly length
- Misassembly count and indel rate are compared against other assemblers, where HASLR is expected to have the lowest misassemblies but a higher indel rate

## Limitations

- HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools, especially versus Canu on the human genome
- Compared with other tools, HASLR usually has a higher indel rate
- Regions with high heterozygosity are not resolved because Minia short-read contigs are fragmented and filtered out
- Backbone graph connectivity depends on the existence of unique SR contigs; lack of them results in multiple connected components, lowering contiguity
- The current version of HASLR does not generate a phased assembly for diploid genomes

## Evidence

- [other] Run RepeatMasker on the HASLR assembly contigs against the curated mammalian repeat library: [evidence span withheld — anchor status: not_found]
- [other] Parse the RepeatMasker .tbl and .out outputs: [evidence span withheld — anchor status: not_found]
