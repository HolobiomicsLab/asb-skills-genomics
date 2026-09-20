---
name: assembly-contiguity-metrics-evaluation
description: 'Use when comparing genome assemblies from several assemblers on the
  same sample and you need reference-based QUAST metrics (contig count, genome fraction,
  NGA50, misassemblies, # mismatches/# indels per 100 kbp) plus wall-clock time and
  peak memory to rank the tools.'
when_to_use_negative:
- No reference genome exists for the sample — the NGA50, genome fraction, and misassembly
  counts in this benchmark all come from reference-guided QUAST evaluation.
- The goal is base-level error correction rather than structural comparison — the
  paper notes most small local assembly mistakes (including mismatch and indel errors)
  can be fixed by further polishing with tools such as Pilon or Racon.
- You need a phased assembly of a diploid genome — the paper states the current version
  of HASLR does not generate a phased assembly for diploid genomes, so phasing metrics
  fall outside this skill.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST
  role: 'Aligns each assembler''s contigs to its reference and reports contig count,
    genome fraction, NGA50, misassemblies, and # mismatches/# indels per 100 kbp for
    the benchmark table'
- name: HASLR
  role: Hybrid assembler under evaluation; run via haslr.py -g <genome size> -x pacbio
    -l <long reads> -s <short reads> and its output scored per dataset
  repo: https://github.com/vpc-ccg/haslr
- name: ART
  role: Simulates the 50x paired-end Illumina short-read half of the four benchmark
    datasets (E. coli, yeast, C. elegans, human)
- name: PBSIM
  role: Simulates the 50x PacBio long-read half of the same four benchmark genomes
- name: Minia
  role: Produces the SR contigs anchoring HASLR's backbone; their fragmentation in
    high-heterozygosity regions explains lower genome fraction/NGA50 outcomes seen
    in the metrics
  repo: https://github.com/GATB/minia
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:0d73de796462a951f42121a2c0db8c73ec8d44103e09ce36e8a8d6afd2c5d700
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
  - ART
  - HASLR
  - Minia
  - PBSIM
  - QUAST
  - fastutils
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# assembly-contiguity-metrics-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A QUAST-driven benchmarking skill that scores each assembler's contigs against a reference genome to compare contiguity (genome fraction, NGA50), structural errors (misassemblies), and base accuracy (# mismatches / # indels per 100 kbp), alongside wall-clock time and peak memory. In the HASLR paper it reproduces the Table 1 comparison of HASLR against 11 other assemblers on simulated E. coli, yeast, C. elegans, and human datasets.

## When to use

Use when comparing genome assemblies from several assemblers on the same sample and you need reference-based QUAST metrics (contig count, genome fraction, NGA50, misassemblies, # mismatches/# indels per 100 kbp) plus wall-clock time and peak memory to rank the tools.

## When NOT to use

- No reference genome exists for the sample — the NGA50, genome fraction, and misassembly counts in this benchmark all come from reference-guided QUAST evaluation.
- The goal is base-level error correction rather than structural comparison — the paper notes most small local assembly mistakes (including mismatch and indel errors) can be fixed by further polishing with tools such as Pilon or Racon.
- You need a phased assembly of a diploid genome — the paper states the current version of HASLR does not generate a phased assembly for diploid genomes, so phasing metrics fall outside this skill.

## Inputs

- Assembled contigs (FASTA) from each compared assembler, e.g., the haslr.py output assembly for each dataset
- Reference genome FASTA per dataset (E. coli, yeast, C. elegans, human)
- Simulated 50x paired-end Illumina FASTQ (ART) and 50x PacBio long-read FASTQ/FASTA (PBSIM) benchmark datasets
- Per-run wall-clock time and peak memory measurements for each assembler

## Outputs

- quast_benchmark_summary.tsv with per-assembler QUAST metrics: contig count, genome fraction, NGA50, misassemblies, # mismatches and # indels per 100 kbp
- Runtime and peak-memory comparison table across assemblers (Table 1 style)

## How to apply

Run every assembler on identical inputs — HASLR via haslr.py with -g (estimated genome size), -x pacbio, -l (long reads), -s (short reads), plus the 11 comparison tools — e.g., 50x ART-simulated paired-end Illumina reads and 50x PBSIM-simulated PacBio reads for the E. coli, yeast, C. elegans, and human genomes. Evaluate every resulting assembly against its own reference genome with QUAST and record contig counts, genome fraction, NGA50, misassemblies, and mismatch/indel rates. Also record wall-clock time and peak memory per assembler on identical hardware and thread counts (reference point: HASLR assembles the CHM1 human dataset in less than 10 hours using 64 CPU threads). Accept the reproduced quast_benchmark_summary.tsv if HASLR shows the lowest runtime and fewest misassemblies on most samples while remaining on par with other assemblers in contiguity (NGA50) and accuracy (mismatch/indel rates). When reading the table, anticipate two documented deviations: HASLR's indel rate is usually higher than other tools, and its genome fraction and/or NGA50 can be lower, clearest against Canu on the human genome.

## Related tools

- **QUAST** (Aligns each assembler's contigs to its reference and reports contig count, genome fraction, NGA50, misassemblies, and # mismatches/# indels per 100 kbp for the benchmark table)
- **HASLR** (Hybrid assembler under evaluation; run via haslr.py -g <genome size> -x pacbio -l <long reads> -s <short reads> and its output scored per dataset) — https://github.com/vpc-ccg/haslr
- **ART** (Simulates the 50x paired-end Illumina short-read half of the four benchmark datasets (E. coli, yeast, C. elegans, human))
- **PBSIM** (Simulates the 50x PacBio long-read half of the same four benchmark genomes)
- **Minia** (Produces the SR contigs anchoring HASLR's backbone; their fragmentation in high-heterozygosity regions explains lower genome fraction/NGA50 outcomes seen in the metrics) — https://github.com/GATB/minia

## Examples

```
quast.py -r ecoli_ref.fasta -o quast_ecoli haslr_ecoli_assembly.fasta minia_ecoli_contigs.fasta
```

## Evaluation signals

- The reproduced quast_benchmark_summary.tsv is accepted only if HASLR shows the lowest runtime and fewest misassemblies on most samples while remaining on par with other assemblers in contiguity (NGA50) and accuracy (mismatch/indel rates)
- Every assembler in the table is scored with the same QUAST metric set — contig count, genome fraction, NGA50, misassemblies, mismatch/indel rates — against the same reference genome
- Expected signatures: HASLR's indel rate is usually higher than competitors, and its genome fraction and/or NGA50 is sometimes lower, clearest versus Canu on human
- Runtime and peak memory are recorded per assembler under identical hardware/threads (reference point: CHM1 human in less than 10 hours using 64 CPU threads)
- Table 1 covers all four simulated datasets (E. coli, yeast, C. elegans, human); a missing dataset or metric column flags an incomplete reproduction

## Limitations

- Compared with other tools, HASLR usually has a higher indel rate, so base-accuracy columns should be read with the knowledge that polishing can fix mismatch/indel errors
- HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools, clearest against Canu on the human genome
- Contiguity depends on long-read length and on the presence of unique SR contigs; high-heterozygosity regions are not resolved because Minia short read contigs are fragmented and filtered out
- Runtime/memory comparisons are only meaningful on identical hardware and thread settings (the CHM1 figure assumes 64 CPU threads)

## Evidence

- [abstract] Our experiments show that HASLR is not only the fastest assembler but also the one with the lowest number of misassemblies on most of the samples: "Our experiments show that HASLR is not only the fastest assembler but also the one with the lowest number of misassemblies on most of the samples"
- [other] Evaluate every assembly against its reference with QUAST and record contig counts, genome fraction, NGA50, misassemblies, and mismatch/indel rates: [evidence span withheld — anchor status: not_found]
- [discussion] HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools. This is clearer when we compare it against Canu: "HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools. This is clearer when we compare it against Canu"
- [discussion] compared with other tools, HASLR usually has a higher indel rate: "compared with other tools, HASLR usually has a higher indel rate"
- [methods] it can assemble a CHM1 human dataset in less than 10 hours using 64 CPU threads.: "it can assemble a CHM1 human dataset in less than 10 hours using 64 CPU threads."
