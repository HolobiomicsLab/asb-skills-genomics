---
name: methylation-hmm-score-computation
description: Use when implementing or extending the GPU methylation score calculation
  step of Galaxy-methyl/f5c call-methylation, e.g. writing or tuning the CUDA HMM
  Fill_kernel over FAST5 signal-level data from basecalled Nanopore reads.
when_to_use_negative:
- Flowcell/chemistry without signal-level support — Nanopolish-based methylation calling
  does not support R10.4 flowcells
- The goal is variant/consensus calling or event alignment rather than detecting methylated
  bases from raw signals
- Running on CPU-only hosts without CUDA library support, since the score calculation
  step depends on `make cuda=1 CUDA_LIB=...`
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Galaxy-methyl
  role: Hosts the call-methylation module whose methylation score calculation (HMM
    Fill_kernel) is GPU-parallelized; provides index and call-methylation commands
  repo: https://github.com/fengyilin118/Galaxy-methyl
- name: f5c
  role: Optimised re-implementation of Nanopolish call-methylation that Galaxy-methyl
    modified for GPU acceleration of the score calculation step
  repo: https://github.com/hasindu2008/f5c
- name: Nanopolish
  role: Original software whose call-methylation module detects methylation via HMM;
    upstream of f5c and Galaxy-methyl
  repo: https://github.com/jts/nanopolish
- name: minimap2
  role: Aligns basecalled ONT reads (-ax map-ont) to the reference genome, producing
    SAM piped into samtools sort before call-methylation
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the alignment BAM consumed by call-methylation
  repo: https://www.htslib.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:aabdc6dff320eaf9b6d253b2a7b31807dd53cf21d3104fed6954e561273fee5e
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# methylation-hmm-score-computation

## Summary

GPU-parallel computation of the HMM methylation score matrix in Galaxy-methyl's call-methylation module, using a CUDA Fill_kernel (Algorithm 2) that assigns one block per k-mer, fills block-independent probabilities in parallel, and fills k-mer-skip probabilities serially.

## When to use

Use when implementing or extending the GPU methylation score calculation step of Galaxy-methyl/f5c call-methylation, e.g. writing or tuning the CUDA HMM Fill_kernel over FAST5 signal-level data from basecalled Nanopore reads.

## When NOT to use

- Flowcell/chemistry without signal-level support — Nanopolish-based methylation calling does not support R10.4 flowcells
- The goal is variant/consensus calling or event alignment rather than detecting methylated bases from raw signals
- Running on CPU-only hosts without CUDA library support, since the score calculation step depends on `make cuda=1 CUDA_LIB=...`

## Inputs

- D4 FAST5 raw signal files from Oxford Nanopore basecalled reads
- Basecalled FASTQ reads
- Sorted, indexed BAM from minimap2 -a -x map-ont alignment
- Reference genome FASTA (e.g. Escherichia coli str. k_12 substr. MG1655 GCA_000005845.2)

## Outputs

- Populated methylation_calls.tsv table of methylated-base calls
- HMM probability matrix entries (PSR9_MATCH, PSR9_BAD_EVEN, PSR9_KMER_SKIP) filled on GPU

## How to apply

First index the FAST5 signal data against the basecalled reads (Galaxy-methyl index) and align reads to the reference with minimap2 -a -x map-ont, piping into samtools sort and indexing the sorted bam. Then implement the CUDA Fill_kernel per Algorithm 2: one thread per k-mer block of the HMM probability matrix; PSR9_MATCH and PSR9_BAD_EVEN emission/transition probabilities are computed in parallel across blocks because they are block-independent, while PSR9_KMER_SKIP is filled serially by the first thread of each group due to dependence on the previous block, with add_log applied at the end. Build with `make cuda=1 CUDA_LIB=/path/to/cuda/library/`, then run call-methylation with CPU threads (e.g. -t 4) and a GPU batch setting (e.g. -B 4.0M) on the sorted bam and reference (e.g. Escherichia coli str. k_12 substr. MG1655 GCA_000005845.2). This step targets the methylation score calculation that Galaxy-methyl parallelized on top of f5c's call-methylation module, after which the module's four steps are pipelined.

## Related tools

- **Galaxy-methyl** (Hosts the call-methylation module whose methylation score calculation (HMM Fill_kernel) is GPU-parallelized; provides index and call-methylation commands) — https://github.com/fengyilin118/Galaxy-methyl
- **f5c** (Optimised re-implementation of Nanopolish call-methylation that Galaxy-methyl modified for GPU acceleration of the score calculation step) — https://github.com/hasindu2008/f5c
- **Nanopolish** (Original software whose call-methylation module detects methylation via HMM; upstream of f5c and Galaxy-methyl) — https://github.com/jts/nanopolish
- **minimap2** (Aligns basecalled ONT reads (-ax map-ont) to the reference genome, producing SAM piped into samtools sort before call-methylation) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the alignment BAM consumed by call-methylation) — https://www.htslib.org/

## Examples

```
git clone https://github.com/fengyilin118/Galaxy-methyl.git; cd Galaxy-methyl; make cuda=1 CUDA_LIB=/path/to/cuda/library/; ./Galaxy-methyl call-methylation -t 4 -B 4.0M > methylation_calls.tsv
```

## Evaluation signals

- The modified Galaxy-methyl build containing the CUDA kernel compiles with `make cuda=1 CUDA_LIB=/path/to/cuda/library/`
- The ./Galaxy-methyl call-methylation run (e.g. -t 4 -B 4.0M on the D4 sorted bam and E. coli GCA_000005845.2 reference) completes end-to-end
- A populated methylation_calls.tsv file is produced
- Results are consistent with Nanopolish/f5c call-methylation output up to floating-point differences (as expected for the f5c re-implementation lineage)
- The kernel's block-independence invariant holds: PSR9_MATCH and PSR9_BAD_EVEN are parallel across blocks while only PSR9_KMER_SKIP is serial within each group

## Limitations

- PSR9_KMER_SKIP depends on the previous block and therefore must be filled serially by the first thread of each group, limiting parallelism of that portion
- Nanopolish-based methylation calling does not support R10.4 flowcells; f5c R10.4.1 support requires specifying --pore r10 for FAST5 input
- Speedup claims (3x-5x over Nanopolish, 35% total-time reduction vs f5c) were measured on specific datasets (PRJEB23027, PRJEB13021 with hg38 and E. coli references), not universally

## Evidence

- [abstract] We propose Galaxy-methyl, which parallelizes and optimizes the methylation score calculation step on GPU: "We propose Galaxy-methyl, which parallelizes and optimizes the methylation score calculation step on GPU"
- [abstract] Call-methylation module of Nanopolish can detect methylation based on Hidden Markov Model (HMM).: "Call-methylation module of Nanopolish can detect methylation based on Hidden Markov Model (HMM)."
- [methods] create an index file that links read ids with their signal-level data in the FAST5 files: "create an index file that links read ids with their signal-level data in the FAST5 files"
- [methods] The output is piped directly into samtools sort to get a sorted bam file: "The output is piped directly into samtools sort to get a sorted bam file"
- [readme] Galaxy-methyl modified *f5c* to parallelize and optimize the methylation score calculation step on GPU and then pipeline the four steps of the call-methylation module.: "Galaxy-methyl modified *f5c* to parallelize and optimize the methylation score calculation step on GPU and then pipeline the four steps of the call-methylation module."
