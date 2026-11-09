---
name: multi-assembler-benchmarking
description: Use when comparing de novo genome assemblers across shared hybrid (long-read
  + short-read) datasets and you need standardized QUAST/BUSCO metrics plus runtime
  and memory per tool.
when_to_use_negative:
- Only a single assembler or a single dataset is available, so no cross-tool comparison
  on identical inputs can be performed
- The data are long-read-only with no Illumina short reads from the same sample, since
  HASLR (the hybrid assembler under test) requires both long reads and NGS reads from
  the same sample
- No reference genome is available for the organism, preventing reference-based QUAST
  metrics such as NGA50, genome fraction, and misassembly counts
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: HASLR
  role: Hybrid assembler under test; run via haslr.py with -g (genome size), -x pacbio,
    -l (long reads), -s (short reads)
  repo: https://github.com/vpc-ccg/haslr
- name: QUAST
  role: Reference-based assembly evaluation producing contig counts, genome fraction,
    NGA50, misassemblies, and mismatch/indel rates
- name: BUSCO
  role: Genome-mode gene completeness assessment against lineage-specific odb10 databases
- name: ART
  role: Simulates 50x paired-end Illumina short reads for the simulated benchmark
    datasets
- name: PBSIM
  role: Simulates 50x PacBio long reads for the simulated benchmark datasets
- name: minimap2
  role: Internal HASLR dependency used to align short read contigs onto long reads
    during assembly
  repo: https://github.com/lh3/minimap2
- name: SPOA
  role: Internal HASLR dependency used for consensus calling of long reads
  repo: https://github.com/rvaser/spoa
- name: Minia
  role: Internal HASLR dependency used for assembling short reads (SR contigs / backbone
    anchors)
  repo: https://github.com/GATB/minia
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:0f7cf85671fc18200d32eb05e06907dcfc86189c78d4b5ec1768a31b270c8636
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
  - BUSCO
  - HASLR
  - MaSuRCA
  - Minia
  - PBSIM
  - PBcR
  - QUAST
  - SPOA
  - fastutils
  - minimap2
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# multi-assembler-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple genome assemblers (HASLR vs. 11 comparison tools) on identical datasets, scoring each assembly with QUAST reference-based metrics (contigs, genome fraction, NGA50, misassemblies, mismatch/indel rates), BUSCO gene completeness, wall-clock time, and peak memory.

## When to use

Use when comparing de novo genome assemblers across shared hybrid (long-read + short-read) datasets and you need standardized QUAST/BUSCO metrics plus runtime and memory per tool.

## When NOT to use

- Only a single assembler or a single dataset is available, so no cross-tool comparison on identical inputs can be performed
- The data are long-read-only with no Illumina short reads from the same sample, since HASLR (the hybrid assembler under test) requires both long reads and NGS reads from the same sample
- No reference genome is available for the organism, preventing reference-based QUAST metrics such as NGA50, genome fraction, and misassembly counts

## Inputs

- Paired-end Illumina short-read FASTQ files for each organism
- PacBio or Oxford Nanopore long-read FASTQ files for each organism
- Reference genome FASTA files (E. coli, yeast, C. elegans, human) for QUAST evaluation
- Final assembly FASTA files (e.g., asm.final.fa) from each benchmarked assembler

## Outputs

- QUAST benchmark summary table (quast_benchmark_summary.tsv) with contig counts, genome fraction, NGA50, misassemblies, mismatch/indel rates, runtime, and peak memory per assembler per dataset
- BUSCO comparative table with complete/single-copy/duplicated/fragmented/missing percentages per assembler per organism
- BUSCO short_summary output files per assembly

## How to apply

Assemble the identical input reads with each tool: for HASLR, run haslr.py specifying the estimated genome size with -g, the long read type with -x pacbio (or ONT), the long read file with -l, and the paired short read files with -s; run all comparison assemblers on the very same datasets. Evaluate every assembly FASTA against its reference with QUAST, recording contig counts, genome fraction, NGA50, number of misassemblies, and mismatch/indel rates, plus wall-clock time and peak memory per assembler. Assess gene completeness by running BUSCO in genome mode against lineage-specific odb10 databases (e.g., enterobacteriales_odb10 for E. coli, saccharomycetes_odb10 for yeast, nematoda_odb10 for C. elegans) and extracting the complete/single-copy/duplicated/fragmented/missing percentages. Accept the benchmark as reproduced if the comparative trends hold — e.g., HASLR shows the lowest runtime and fewest misassemblies on most samples while remaining on par in NGA50 and accuracy — and report known trade-offs (e.g., higher indel rate, lower genome fraction versus Canu on human).

## Related tools

- **HASLR** (Hybrid assembler under test; run via haslr.py with -g (genome size), -x pacbio, -l (long reads), -s (short reads)) — https://github.com/vpc-ccg/haslr
- **QUAST** (Reference-based assembly evaluation producing contig counts, genome fraction, NGA50, misassemblies, and mismatch/indel rates)
- **BUSCO** (Genome-mode gene completeness assessment against lineage-specific odb10 databases)
- **ART** (Simulates 50x paired-end Illumina short reads for the simulated benchmark datasets)
- **PBSIM** (Simulates 50x PacBio long reads for the simulated benchmark datasets)
- **minimap2** (Internal HASLR dependency used to align short read contigs onto long reads during assembly) — https://github.com/lh3/minimap2
- **SPOA** (Internal HASLR dependency used for consensus calling of long reads) — https://github.com/rvaser/spoa
- **Minia** (Internal HASLR dependency used for assembling short reads (SR contigs / backbone anchors)) — https://github.com/GATB/minia

## Examples

```
haslr.py -g 100m -x pacbio -l long_reads.fastq -s reads_1.fastq reads_2.fastq
```

## Evaluation signals

- On simulated E. coli, yeast, C. elegans, and human datasets, HASLR shows the lowest number of misassemblies among all compared assemblers on most samples (Table 1 trend)
- HASLR shows the lowest wall-clock time among all compared assemblers (e.g., assembles a CHM1 human dataset in less than 10 hours using 64 CPU threads)
- NGA50 and mismatch/indel rates for HASLR remain on par with the best performing tools, with the documented exception of lower genome fraction/NGA50 versus Canu on human
- BUSCO complete percentages are comparable to competitors (e.g., C. elegans: 97.1% complete for HASLR vs 97.4% for Canu)
- Every assembler receives byte-identical input reads per dataset; time and peak memory are recorded under comparable thread counts

## Limitations

- Compared with other tools, HASLR usually has a higher indel rate
- HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools, especially Canu on the human genome
- Connectivity of the backbone graph depends on the existence of unique short read (SR) contigs; regions of high heterozygosity may not be resolved because Minia SR contigs there are fragmented and filtered out
- The current version of HASLR does not generate a phased assembly for diploid genomes

## Evidence
