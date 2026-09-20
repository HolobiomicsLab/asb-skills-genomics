---
name: pacbio-long-read-simulation
description: Use when you need ground-truth-controlled hybrid sequencing data (short
  + long reads) to benchmark genome assemblers on misassemblies, NGA50, genome fraction,
  runtime, and memory.
when_to_use_negative:
- You need to benchmark on real, previously published sequencing data — use the paper's
  real datasets (e.g., E. coli Loman lab ultra-long Nanopore reads with Illumina MiSeq,
  yeast ERX1725434/ERX1725435/ERX1725441 PacBio with ERX1943903 Illumina, C. elegans
  DevNet PacBio with SRR065390, CHM1 SRP044331 PacBio with SRX652547) instead of simulation
- The goal is phased assembly of diploid genomes — the paper states the current HASLR
  version does not generate phased assemblies
- Reads are high-fidelity (e.g., PacBio HiFi/CCS) rather than error-prone long reads
  — HASLR is designed for error-prone PacBio/ONT long reads combined with Illumina
  short reads
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: HASLR
  role: Hybrid assembler under evaluation, run per dataset via haslr.py with -g, -x
    pacbio, -l, -s flags
  repo: https://github.com/vpc-ccg/haslr
- name: ART
  role: Simulates 50x paired-end Illumina short reads from each reference genome
- name: PBSIM
  role: Simulates 50x PacBio long reads from each reference genome
- name: QUAST
  role: Evaluates each assembly against its reference to record contigs, genome fraction,
    NGA50, misassemblies, and mismatch/indel rates
- name: Minia
  role: Internal short-read assembly dependency used by HASLR to generate SR contigs
  repo: https://github.com/GATB/minia
- name: minimap2
  role: Internal dependency aligning short read contigs onto long reads
  repo: https://github.com/lh3/minimap2
- name: SPOA
  role: Internal dependency for consensus calling of long reads
  repo: https://github.com/rvaser/spoa
- name: fastutils
  role: FASTA/Q manipulation utility used in the HASLR pipeline
  repo: https://github.com/haghshenas/fastutils
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:48e76e9875ac9ff81fcd35e3cd8bc5cc1cec196eac1c62b553bfc942aac1db9e
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
  - Minimap2
  - PBSIM
  - PBSIM2
  - PacBio
  - QUAST
  - Racon
  - SPOA
  - VeChat
  - fastutils
  - fpa
  - minimap2
  - yacrd
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- pacbio-clr-long-read-simulation
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# pacbio-long-read-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate 50x paired-end Illumina short reads (ART) and 50x PacBio long reads (PBSIM) from reference genomes (E. coli, yeast, C. elegans, human) to build controlled benchmark datasets for comparing hybrid assemblers such as HASLR against alternatives using QUAST metrics.

## When to use

Use when you need ground-truth-controlled hybrid sequencing data (short + long reads) to benchmark genome assemblers on misassemblies, NGA50, genome fraction, runtime, and memory.

## When NOT to use

- You need to benchmark on real, previously published sequencing data — use the paper's real datasets (e.g., E. coli Loman lab ultra-long Nanopore reads with Illumina MiSeq, yeast ERX1725434/ERX1725435/ERX1725441 PacBio with ERX1943903 Illumina, C. elegans DevNet PacBio with SRR065390, CHM1 SRP044331 PacBio with SRX652547) instead of simulation
- The goal is phased assembly of diploid genomes — the paper states the current HASLR version does not generate phased assemblies
- Reads are high-fidelity (e.g., PacBio HiFi/CCS) rather than error-prone long reads — HASLR is designed for error-prone PacBio/ONT long reads combined with Illumina short reads

## Inputs

- Reference genome FASTA files (E. coli, yeast, C. elegans, human)
- Per-genome target coverage specification (50x for both read types)
- Estimated genome size for each organism (for the assembler -g flag)

## Outputs

- 50x paired-end Illumina FASTQ files simulated with ART per genome
- 50x PacBio long-read FASTQ/FASTA files simulated with PBSIM per genome
- Assemblies from HASLR and 11 comparison assemblers per dataset
- QUAST benchmark summary table (e.g., quast_benchmark_summary.tsv) with contigs, genome fraction, NGA50, misassemblies, mismatch/indel rates, runtime, and peak memory

## How to apply

Simulate 50x paired-end Illumina short reads for each reference genome (E. coli, yeast, C. elegans, human) using ART, and simulate 50x PacBio long reads for the same four genomes using PBSIM. Run HASLR on each dataset via haslr.py, specifying the estimated genome size with -g, the long read type with -x pacbio, the long read file with -l, and the short read files with -s; run the comparison assemblers (11 tools in the HASLR paper) on identical simulated inputs. Evaluate every assembly against its reference with QUAST, recording contig counts, genome fraction, NGA50, misassemblies, and mismatch/indel rates, plus wall-clock time and peak memory per assembler. The rationale is that simulated reads from known references allow misassembly and accuracy metrics to be attributed to the assembler rather than to unknown dataset variation.

## Related tools

- **HASLR** (Hybrid assembler under evaluation, run per dataset via haslr.py with -g, -x pacbio, -l, -s flags) — https://github.com/vpc-ccg/haslr
- **ART** (Simulates 50x paired-end Illumina short reads from each reference genome)
- **PBSIM** (Simulates 50x PacBio long reads from each reference genome)
- **QUAST** (Evaluates each assembly against its reference to record contigs, genome fraction, NGA50, misassemblies, and mismatch/indel rates)
- **Minia** (Internal short-read assembly dependency used by HASLR to generate SR contigs) — https://github.com/GATB/minia
- **minimap2** (Internal dependency aligning short read contigs onto long reads) — https://github.com/lh3/minimap2
- **SPOA** (Internal dependency for consensus calling of long reads) — https://github.com/rvaser/spoa
- **fastutils** (FASTA/Q manipulation utility used in the HASLR pipeline) — https://github.com/haghshenas/fastutils

## Examples

```
haslr.py -g 12000000 -x pacbio -l sim_pacbio_50x.fastq -s sim_illumina_R1.fastq.gz -s sim_illumina_R2.fastq.gz
```

## Evaluation signals

- The reproduced quast_benchmark_summary.tsv is accepted if HASLR shows the lowest runtime and fewest misassemblies on most samples
- HASLR's NGA50 and mismatch/indel rates should remain on par with the best-performing comparison assemblers on the simulated datasets
- Each simulated dataset must achieve approximately 50x coverage for both short reads (ART) and long reads (PBSIM), verifiable with fastutils stat/length
- Every assembler must be run on the identical simulated read sets per genome so QUAST metric differences are attributable to the assembler
- QUAST runs must report metrics for all required columns: contig counts, genome fraction, NGA50, misassemblies, mismatch/indel rates, wall-clock time, and peak memory

## Limitations

- Simulated-data conclusions do not fully transfer to real data: on real datasets HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools, especially Canu on the human genome
- HASLR usually has a higher indel rate compared with other tools
- Backbone graph connectivity depends on the existence of unique SR contigs; lack of them results in multiple connected components, and high-heterozygosity regions are not resolved because Minia short read contigs are fragmented and filtered out
- The current version of HASLR does not generate a phased assembly for diploid genomes

## Evidence
