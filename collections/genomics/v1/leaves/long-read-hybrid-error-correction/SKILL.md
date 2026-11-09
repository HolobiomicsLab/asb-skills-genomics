---
name: long-read-hybrid-error-correction
description: Use when noisy long reads (e.g. PacBio E. coli, S. cerevisiae, or heterozygous
  A. funestus flowcell data) must be corrected with matched Illumina short reads,
  and alignment/assembly-based correction quality metrics are needed.
when_to_use_negative:
- No matching short reads are available — HECIL is a hybrid corrector requiring aligned
  Illumina reads (self-correcting methods like HGAP are a different approach)
- The long reads are already high quality or have been corrected (e.g. Canu-corrected
  reads are only further improved as a downstream assembly-refinement option)
- The goal is de novo assembly only without correction — assembly alone with Canu
  does not substitute for correction and evaluation
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HECIL
  role: Hybrid error correction framework (quick + optimization-based correction with
    iterative learning); run as python HECIL.py
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: Aligns short reads to long reads to mark questionable positions and realign
    updated long reads each iteration
- name: Samtools
  role: Dependency for alignment post-processing in the HECIL pipeline
- name: Jellyfish
  role: Counts unique k-mers of corrected long reads after each correction/iteration
    for termination and comparison
- name: BLASR
  role: Aligns corrected long reads to the reference genome to compute PI, % matched
    bases, and number of aligned reads
- name: Canu
  role: De novo assembly of corrected long reads for assembly-based evaluation
- name: QUAST
  role: Evaluates assembly quality (e.g. N50) of corrected-read assemblies
- name: MUMmer dnadiff
  role: Measures assembly accuracy by aligning assembled genome to the reference
- name: LSC
  role: Hybrid correction baseline for comparison
- name: PacBioToCA
  role: Hybrid correction baseline for comparison
- name: LoRDEC
  role: Hybrid correction baseline run with 16 threads
- name: proovread
  role: Hybrid correction baseline run with 16 threads
- name: CoLoRMap
  role: Hybrid correction baseline for comparison
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:0ac5125548823262777c2cf7c97dc1a811d91fd1908ce7c8f1dd9e698eaceca7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BLASR
  - BWA-MEM
  - Canu
  - CoLoRMap
  - HECIL
  - Jellyfish
  - LSC
  - LoRDEC
  - MUMmer dnadiff
  - PacBioToCA
  - QUAST
  - Samtools
  - proovread
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# long-read-hybrid-error-correction

## Summary

Hybrid error correction of noisy third-generation long reads (PacBio/Sequel, Oxford Nanopore) using high-quality Illumina short reads, implemented as HECIL: quick consensus correction plus optimization-based correction, optionally enhanced by an iterative learning loop that realigns and re-corrects across iterations.

## When to use

Use when noisy long reads (e.g. PacBio E. coli, S. cerevisiae, or heterozygous A. funestus flowcell data) must be corrected with matched Illumina short reads, and alignment/assembly-based correction quality metrics are needed.

## When NOT to use

- No matching short reads are available — HECIL is a hybrid corrector requiring aligned Illumina reads (self-correcting methods like HGAP are a different approach)
- The long reads are already high quality or have been corrected (e.g. Canu-corrected reads are only further improved as a downstream assembly-refinement option)
- The goal is de novo assembly only without correction — assembly alone with Canu does not substitute for correction and evaluation

## Inputs

- Long noisy reads in FASTA format (e.g. E. coli PacBio/Sequel reads filtered to >100 bp, 33,360 reads; S. cerevisiae 1,758,169 reads; A. funestus flowcell reads)
- Matching Illumina short reads in FASTQ format with PHRED quality scores (e.g. 22,720,100 E. coli sequences; 4,503,422 S. cerevisiae reads)
- Reference genome for evaluation (E. coli K-12 MG1655; S. cerevisiae S288C 12.2 Mbp; A. funestus Fumoz GenBank GCA_000349085.1)

## Outputs

- Corrected long reads in FASTA (LongRead_Corrected.fasta), containing all corrected and uncorrected long reads
- Per-iteration metrics: percent identity (PI), % matched bases, number of aligned reads (BLASR), unique k-mer counts (Jellyfish)
- Assembly-based metrics from Canu + QUAST (N50) and MUMmer dnadiff assembly accuracy
- Tabulated benchmark CSVs comparing HECIL iterations to baselines (e.g. hecil_ecoli_benchmark.csv, pi_matched_bases_progression.csv, yeast_correction_metrics.csv, hecil_afunestus_flowcell_metrics.csv)

## How to apply

Align short reads to long reads with BWA-MEM and mark mismatch, insertion, and deletion positions on the long reads as questionable. Apply quick correction only where short-read consensus reaches at least η = 0.9; for remaining questionable bases, select the aligned short read maximizing the convex-combination cost J_k = ½(ω̂_q^k + ω̂_s^k) of the normalized PHRED quality weight and the alignment-identity (exact matches / short read length) similarity weight, breaking ties by highest quality. For iterative learning, fix high-confidence corrections above the α > 95 percentile of optimal costs, realign the updated long reads to the short reads each iteration, and terminate when the relative reduction in unique k-mers (Jellyfish) falls below ε or the maximal allowed iterations n is reached (e.g. iteration 5). Evaluate outputs with BLASR-derived PI and % matched bases against a reference genome, unique k-mer counts, and Canu/QUAST/MUMmer dnadiff assembly metrics.

## Related tools

- **HECIL** (Hybrid error correction framework (quick + optimization-based correction with iterative learning); run as python HECIL.py) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Aligns short reads to long reads to mark questionable positions and realign updated long reads each iteration)
- **Samtools** (Dependency for alignment post-processing in the HECIL pipeline)
- **Jellyfish** (Counts unique k-mers of corrected long reads after each correction/iteration for termination and comparison)
- **BLASR** (Aligns corrected long reads to the reference genome to compute PI, % matched bases, and number of aligned reads)
- **Canu** (De novo assembly of corrected long reads for assembly-based evaluation)
- **QUAST** (Evaluates assembly quality (e.g. N50) of corrected-read assemblies)
- **MUMmer dnadiff** (Measures assembly accuracy by aligning assembled genome to the reference)
- **LSC** (Hybrid correction baseline for comparison)
- **PacBioToCA** (Hybrid correction baseline for comparison)
- **LoRDEC** (Hybrid correction baseline run with 16 threads)
- **proovread** (Hybrid correction baseline run with 16 threads)
- **CoLoRMap** (Hybrid correction baseline for comparison)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- PI and % matched bases increase monotonically relative to the uncorrected baseline across iterations (e.g. Sequel E. coli: PI 85.0 → 95.6 at iteration 1 → 97.2 at iteration 5; % matched 85.1 → 95.6 → 97.1)
- Unique k-mer counts decrease monotonically across iterations until the relative improvement falls below ε, or the iteration cap n is reached
- Reproduced S. cerevisiae values match reported metrics: % matched 78.8 → 85.6 (iter 1) → 86.5 (iter 5); PI 93.8 → 98.6 → 98.9 with 346,242 aligned reads
- Iterative learning consistently improves all evaluation metrics, especially on heterozygous A. funestus data (e.g. flowcell 1 PI 94.5 → 98.5; flowcell 16 87.5% matched bases with 3.8% improvement at iteration 5)
- Output FASTA contains all corrected and uncorrected long reads, and the cost function is the equal-weight (½, ½) convex combination of normalized quality and similarity weights with highest-quality tie-breaking

## Limitations

- Iterative improvement is only a conjecture with no mathematical proof; a rigorous framework for proving it likely requires unrealistic stochastic assumptions
- Saturation behavior of the iterative procedure (metrics improving until saturation) is unvalidated speculation
- Multi-threading speedup is not implemented in the current version; parallelism is achieved only by splitting the workload into 16 independent data-parallel tasks, and iterative learning incurs extra total execution time
- Dependencies are pinned to old versions (BWA 0.7.15, samtools 0.1.18, Python 2.7.8), and cloned scripts may need chmod +x to execute

## Evidence

- [methods] Quick correction threshold: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
- [methods] Confidence percentile: "Here we select α > 95 so that a small percentage of the optimal corrections are considered to be of high confidence."
- [methods] Realignment step: "We learn in successive iterations by realigning the updated long reads to the short reads."
- [results] k-mer termination criterion: "We employ the widely-used k-mer counting tool Jellyfish19 to compute the number of unique k-mers obtained after each correction algorithm"
- [readme] README invocation: "Example command: python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out"
- [results] E. coli filtering: "We filter long reads of E. coli to exclude reads shorter than 100 bp, creating a final set of 33,360"
