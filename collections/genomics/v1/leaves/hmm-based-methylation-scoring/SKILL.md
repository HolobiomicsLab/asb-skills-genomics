---
name: hmm-based-methylation-scoring
description: Use when basecalled Nanopore reads (FASTQ) with raw FAST5 signals must
  be scored for CpG methylation via an HMM, or when comparing GPU vs CPU (OpenMP)
  methylation-calling performance.
when_to_use_negative:
- Input reads are from R10.4 flowcells — Nanopolish does not support R10.4 for methylation
  calling
- Signals are in S/BLOW5 or POD5 format without FAST5 conversion — the indexed FAST5
  workflow described here expects FAST5 signal data
- The goal is methylation frequencies rather than per-site HMM scoring — downstream
  tools like f5c meth-freq or Nanopolish compare_methylation.py would be more direct
edam_operation: http://edamontology.org/operation_3186
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: Galaxy-methyl
  role: Runs the GPU-parallelized HMM methylation score calculation via call-methylation
    after building the signal index
  repo: https://github.com/fengyilin118/Galaxy-methyl
- name: f5c
  role: CPU (OpenMP) baseline re-implementation of call-methylation used for execution-time
    comparison against the GPU HMM step
  repo: https://github.com/hasindu2008/f5c
- name: Nanopolish
  role: Reference implementation of HMM call-methylation used for correctness verification
    of Galaxy-methyl log-likelihood ratios
  repo: https://github.com/jts/nanopolish
- name: minimap2
  role: Aligns basecalled ONT reads to the reference genome with -a -x map-ont before
    methylation calling
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts minimap2 output and indexes the sorted BAM consumed by call-methylation
  repo: https://www.htslib.org/
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:dc8b723df7c986182693075c8577f38c513286fdfb92cbed352c09a6e291b81b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fengyilin118/Galaxy-methyl
  tools:
  - Galaxy-methyl
  - Nanopolish
  - f5c
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3186
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# hmm-based-methylation-scoring

## Summary

Detects methylated cytosines in Oxford Nanopore reads by running the Hidden Markov Model (HMM) methylation score calculation of the call-methylation module, as parallelized on GPU in Galaxy-methyl (a fork of f5c/Nanopolish) for ~2x speedup on the HMM step and identical log-likelihood ratio outputs.

## When to use

Use when basecalled Nanopore reads (FASTQ) with raw FAST5 signals must be scored for CpG methylation via an HMM, or when comparing GPU vs CPU (OpenMP) methylation-calling performance.

## When NOT to use

- Input reads are from R10.4 flowcells — Nanopolish does not support R10.4 for methylation calling
- Signals are in S/BLOW5 or POD5 format without FAST5 conversion — the indexed FAST5 workflow described here expects FAST5 signal data
- The goal is methylation frequencies rather than per-site HMM scoring — downstream tools like f5c meth-freq or Nanopolish compare_methylation.py would be more direct

## Inputs

- Basecalled Nanopore reads in FASTQ (.fastq.gz)
- Raw signal data in FAST5 files
- Sorted, indexed BAM of reads aligned with minimap2 -a -x map-ont
- Reference genome FASTA (e.g. hg38, GRCh38.primary_assembly.genome.fa, or GCA_000005845.2 E. coli genome)
- Signal index linking read ids to FAST5 signal-level data

## Outputs

- methylation_calls.tsv with per-CpG-site log-likelihood ratios for methylated and unmethylated groups
- Execution-time tabulation across datasets when used for the GPU-vs-CPU benchmark

## How to apply

First create a signal index linking read ids to FAST5 signal data (./Galaxy-methyl index -d <fast5_dir> <fastq.gz>), then align basecalled reads to the matching reference with 'minimap2 -a -x map-ont' piped into 'samtools sort', and index the sorted BAM with samtools. Run the methylation score calculation with './Galaxy-methyl call-methylation -t 4 -B 4.0M -r <fastq.gz> -b <sorted.bam> -g <reference.fasta>', which executes the HMM step on GPU while launching 4 CPU threads and using a 4.0M batch size. Choose the reference per dataset (hg38 FASTA for human D1-D3/D5-D6, GCA_000005845.2 E. coli K-12 MG1655 for D4). Rationale: Galaxy-methyl parallelizes and optimizes the methylation score calculation on GPU and pipelines the four call-methylation steps, yielding identical log-likelihood ratios to Nanopolish while reducing execution time ~35% versus CPU f5c and 3x-5x versus Nanopolish.

## Related tools

- **Galaxy-methyl** (Runs the GPU-parallelized HMM methylation score calculation via call-methylation after building the signal index) — https://github.com/fengyilin118/Galaxy-methyl
- **f5c** (CPU (OpenMP) baseline re-implementation of call-methylation used for execution-time comparison against the GPU HMM step) — https://github.com/hasindu2008/f5c
- **Nanopolish** (Reference implementation of HMM call-methylation used for correctness verification of Galaxy-methyl log-likelihood ratios) — https://github.com/jts/nanopolish
- **minimap2** (Aligns basecalled ONT reads to the reference genome with -a -x map-ont before methylation calling) — https://github.com/lh3/minimap2
- **samtools** (Sorts minimap2 output and indexes the sorted BAM consumed by call-methylation) — https://www.htslib.org/

## Examples

```
./Galaxy-methyl call-methylation -t 4 -B 4.0M -r FAB41174.fastq.gz -b FAB41174.sorted.bam -g hg38.fasta > methylation_calls.tsv
```

## Evaluation signals

- Per-CpG log-likelihood ratios from Galaxy-methyl are identical to those from Nanopolish call-methylation on the same indexed/aligned inputs (cpg_llr_comparison.tsv shows no flagged disagreements)
- GPU HMM-step execution times show approximately 2x (or greater) speedup over CPU f5c call-methylation, with larger gains on compute-intensive human-genome datasets D1-D3
- Total execution time is reduced ~35% on average versus f5c, and 3x-5x versus Nanopolish
- The sorted BAM is indexed with samtools index before call-methylation, and minimap2 used the map-ont preset
- Index step completes without error, confirming every read id in the FASTQ is linked to FAST5 signal data

## Limitations

- Speedup advantage is largest where the methylation score calculation is compute-intensive (human-genome datasets D1-D3); smaller gains are expected on less compute-intensive datasets such as D4-D6 (E. coli / other references)
- Nanopolish (and hence this Nanopolish-derived HMM workflow) does not support R10.4 flowcells
- Floating-point approximations can cause minor differences versus re-implementations (f5c results are 'almost the same as from nanopolish except a few differences due to floating point approximations')

## Evidence

- [readme] Galaxy-methyl modified f5c to parallelize and optimize the methylation score calculation step on GPU and then pipeline the four steps of the call-methylation module.: "Galaxy-methyl modified f5c to parallelize and optimize the methylation score calculation step on GPU and then pipeline the four steps of the call-methylation module."
- [abstract] Call-methylation module of Nanopolish can detect methylation based on Hidden Markov Model (HMM).: "Call-methylation module of Nanopolish can detect methylation based on Hidden Markov Model (HMM)."
- [methods] create an index file that links read ids with their signal-level data in the FAST5 files: "create an index file that links read ids with their signal-level data in the FAST5 files"
- [methods] The output is piped directly into samtools sort to get a sorted bam file: "The output is piped directly into samtools sort to get a sorted bam file"
- [abstract] Galaxy-methyl can achieve 3×–5× speedup compared with Nanopolish: "Galaxy-methyl can achieve 3×–5× speedup compared with Nanopolish"
