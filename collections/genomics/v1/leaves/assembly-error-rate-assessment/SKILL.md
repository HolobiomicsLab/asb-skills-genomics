---
name: assembly-error-rate-assessment
description: Use when comparing de novo/hybrid assemblers on identical read datasets
  and you need error-rate evidence (misassemblies, mismatch/indel rates) plus contiguity
  (NGA50, genome fraction) and resource cost (runtime, peak memory) per tool.
when_to_use_negative:
- Your goal is to fix or polish assembly errors (e.g., with Pilon or Racon) rather
  than quantify and compare error rates across assemblers
- You only have long reads and no Illumina short reads — HASLR is a hybrid tool that
  requires both third-generation long reads (PacBio/ONT) and NGS short reads from
  the same sample
- You need a phased assembly of a diploid genome — the article states the current
  HASLR version does not generate a phased assembly
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: HASLR
  role: Focal hybrid assembler under test; run via haslr.py on simulated PacBio+Illumina
    datasets to benchmark misassemblies
  repo: https://github.com/vpc-ccg/haslr
- name: QUAST
  role: Assembly evaluation tool that scores contig counts, genome fraction, NGA50,
    misassemblies, and mismatch/indel rates against each reference
- name: ART
  role: Simulates the 50x paired-end Illumina short reads for the four benchmark genomes
- name: PBSIM
  role: Simulates the 50x PacBio long reads for the four benchmark genomes
- name: Minia
  role: HASLR dependency that assembles the short reads into SR contigs feeding the
    hybrid assembly
  repo: https://github.com/GATB/minia
- name: minimap2
  role: HASLR dependency that aligns short read contigs onto long reads
  repo: https://github.com/lh3/minimap2
- name: SPOA
  role: HASLR dependency for consensus calling of long reads between anchors
  repo: https://github.com/rvaser/spoa
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:a6b309503ed6f55ac2b8229486d52d7b53e880f2bce4ed08bff690ff6de74e83
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
  - SPOA
  - fastutils
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# assembly-error-rate-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark workflow for quantifying structural and base-level errors in genome assemblies: run hybrid assemblies (HASLR plus comparators) on simulated PacBio+Illumina datasets and score them with QUAST metrics (misassemblies, NGA50, genome fraction, mismatch/indel rates) to establish which assembler makes the fewest mistakes.

## When to use

Use when comparing de novo/hybrid assemblers on identical read datasets and you need error-rate evidence (misassemblies, mismatch/indel rates) plus contiguity (NGA50, genome fraction) and resource cost (runtime, peak memory) per tool.

## When NOT to use

- Your goal is to fix or polish assembly errors (e.g., with Pilon or Racon) rather than quantify and compare error rates across assemblers
- You only have long reads and no Illumina short reads — HASLR is a hybrid tool that requires both third-generation long reads (PacBio/ONT) and NGS short reads from the same sample
- You need a phased assembly of a diploid genome — the article states the current HASLR version does not generate a phased assembly

## Inputs

- Simulated 50x paired-end Illumina short reads (FASTQ) for E. coli, yeast, C. elegans, and human references, generated with ART
- Simulated 50x PacBio long reads (FASTA/FASTQ) for the same four genomes, generated with PBSIM
- Reference genome sequences (FASTA) matching each simulated dataset
- Estimated genome size per dataset (haslr.py -g)
- Assembly contig files (FASTA) from HASLR and the 11 comparison assemblers

## Outputs

- quast_benchmark_summary.tsv with QUAST metrics (contig counts, genome fraction, NGA50, misassemblies, mismatch/indel rates) per assembler per dataset
- Wall-clock runtime and peak memory measurements per assembler per dataset
- Per-assembly QUAST reports validating each contig set against its reference genome

## How to apply

Simulate 50x paired-end Illumina short reads for the E. coli, yeast, C. elegans, and human reference genomes with ART, and 50x PacBio long reads with PBSIM, so every assembler is evaluated on byte-identical inputs. Run HASLR on each dataset via haslr.py, specifying the estimated genome size with -g, the long read type with -x pacbio, the long read file with -l, and the short read files with -s, then run the 11 comparison assemblers on the same simulated data. Evaluate every assembly against its own reference with QUAST, recording contig counts, genome fraction, NGA50, misassemblies, and mismatch/indel rates, plus wall-clock time and peak memory per assembler. The decision point is structural-error rate: the reproduced quast_benchmark_summary.tsv is accepted if the focal assembler (HASLR) shows the lowest runtime and fewest misassemblies on most samples while remaining on par in NGA50 and mismatch/indel rates.

## Related tools

- **HASLR** (Focal hybrid assembler under test; run via haslr.py on simulated PacBio+Illumina datasets to benchmark misassemblies) — https://github.com/vpc-ccg/haslr
- **QUAST** (Assembly evaluation tool that scores contig counts, genome fraction, NGA50, misassemblies, and mismatch/indel rates against each reference)
- **ART** (Simulates the 50x paired-end Illumina short reads for the four benchmark genomes)
- **PBSIM** (Simulates the 50x PacBio long reads for the four benchmark genomes)
- **Minia** (HASLR dependency that assembles the short reads into SR contigs feeding the hybrid assembly) — https://github.com/GATB/minia
- **minimap2** (HASLR dependency that aligns short read contigs onto long reads) — https://github.com/lh3/minimap2
- **SPOA** (HASLR dependency for consensus calling of long reads between anchors) — https://github.com/rvaser/spoa

## Examples

```
haslr.py -g 4.6m -x pacbio -l lr_pacbio.fasta -s sr_1.fastq,sr_2.fastq
```

## Evaluation signals

- quast_benchmark_summary.tsv is accepted if HASLR shows the lowest runtime and fewest misassemblies on most samples while remaining on par with other assemblers in NGA50 and mismatch/indel rates
- Every assembly is evaluated against its own matching reference genome with QUAST — a mismatch between assembly and reference invalidates the metric row
- All 12 assemblers (HASLR plus 11 comparators) must be run on the identical simulated read files for each of the four genomes; results across tools are not comparable otherwise
- Misassembly counts should be reported separately from mismatch/indel rates so structural errors are not conflated with base-level errors (HASLR is known to have a higher indel rate)

## Limitations

- HASLR usually has a higher indel rate compared with other tools
- HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools, most clearly against Canu on the human genome
- Connectivity of the backbone graph depends on the existence of unique SR contigs; regions of high heterozygosity may go unresolved because Minia short read contigs there are fragmented and filtered out
- An important factor in contiguity is long read length — longer reads generate a more connected and resolved backbone graph, so error profiles shift with read length

## Evidence

- [abstract] fastest with lowest misassemblies: "Our experiments show that HASLR is not only the fastest assembler but also the one with the lowest number of misassemblies on most of the samples"
- [intro] comparative benchmark claim: "Based on our results, HASLR is the fastest between all the assemblers we tested, while generating the lowest number of mis-assemblies on most datasets."
- [readme] hybrid input requirement: "HASLR is a hybrid tool which means it requires long reads generated by Third Generation Sequencing technologies"
- [discussion] indel-rate caveat: "compared with other tools, HASLR usually has a higher indel rate"
