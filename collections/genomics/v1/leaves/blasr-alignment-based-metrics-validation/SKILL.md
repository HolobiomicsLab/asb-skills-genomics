---
name: blasr-alignment-based-metrics-validation
description: Use when you have hybrid long-read correction results (e.g., HECIL output
  at iteration 1 and 5) plus a reference genome and need alignment-based accuracy
  metrics like PI and % matched bases to validate improvement over uncorrected reads.
when_to_use_negative:
- Input data lacks a reference genome or curated alignments, so alignment-based validation
  metrics (PI, % matched bases) cannot be computed
- The task is self-correction of long reads without short reads (e.g., HGAP-style);
  HECIL is a hybrid method that requires Illumina short reads
- Evaluating assemblies rather than read-level correction accuracy — use the paper's
  complementary QUAST/MUMmer dnadiff workflow instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: HECIL
  role: Hybrid error corrector producing the corrected long-read sets (iterations
    1 and 5) that are validated by BLASR alignment
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: Aligns Illumina short reads to long reads inside HECIL to locate questionable
    bases prior to correction
- name: BLASR
  role: Aligns uncorrected and corrected long reads to the reference genome to compute
    PI, % matched bases, and aligned-read counts
- name: samtools
  role: Dependency of the HECIL pipeline for handling short-read alignments
  repo: https://github.com/NDBL/HECIL
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:3694248fe471ae02d1c96febbc9973576931841b3b9d77b5a537eaa2ee2483a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BLASR
  - BWA-MEM
  - HECIL
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# blasr-alignment-based-metrics-validation

## Summary

Validation workflow from the HECIL paper (Sci Rep 2018) in which uncorrected and iteratively corrected PacBio-style long reads are aligned to a reference genome with BLASR to compute percent identity (PI), % matched bases, and number of aligned reads, demonstrating that hybrid correction improves read accuracy.

## When to use

Use when you have hybrid long-read correction results (e.g., HECIL output at iteration 1 and 5) plus a reference genome and need alignment-based accuracy metrics like PI and % matched bases to validate improvement over uncorrected reads.

## When NOT to use

- Input data lacks a reference genome or curated alignments, so alignment-based validation metrics (PI, % matched bases) cannot be computed
- The task is self-correction of long reads without short reads (e.g., HGAP-style); HECIL is a hybrid method that requires Illumina short reads
- Evaluating assemblies rather than read-level correction accuracy — use the paper's complementary QUAST/MUMmer dnadiff workflow instead

## Inputs

- Uncorrected long reads (FASTA/FASTQ; e.g., S. cerevisiae S288C, 1,758,169 PacBio-style reads)
- Illumina short reads (FASTQ; e.g., 4,503,422 short reads)
- Reference genome (e.g., S. cerevisiae S288C, 12.2 Mbp)
- Corrected long-read sets from HECIL iteration 1 and iteration 5 (LongRead_Corrected.fasta)

## Outputs

- BLASR alignments of uncorrected and corrected long reads against the reference genome
- Per-condition metrics table (e.g., yeast_correction_metrics.csv) with PI, % matched bases, and # aligned reads for uncorrected / iteration 1 / iteration 5

## How to apply

Run the hybrid corrector (HECIL) on the long reads and short reads — for S. cerevisiae S288C this is 1,758,169 long reads and 4,503,422 short reads — where BWA-MEM aligns short reads to long reads, mismatch/insertion/deletion positions are marked questionable, quick correction applies when ≥90% consensus (η = 0.9) is reached, and optimization-based correction (convex combination of normalized quality and similarity weights) handles the rest; the iterative loop uses an α > 95 percentile confidence threshold and realignment until the relative reduction in unique k-mers falls below ε or n maximal iterations is reached. Save corrected reads at iteration 1 and iteration 5 (HECIL writes all corrected and uncorrected long reads to LongRead_Corrected.fasta). Align the uncorrected reads and each corrected iteration to the S288C reference genome (12.2 Mbp) using BLASR. From the BLASR alignments, compute percent identity (PI), % matched bases, and number of aligned reads per condition, then assemble the 3×3 metric grid (e.g., yeast_correction_metrics.csv). Validation succeeds if PI and % matched bases increase monotonically from uncorrected to iteration 5 and match the paper's reported values.

## Related tools

- **HECIL** (Hybrid error corrector producing the corrected long-read sets (iterations 1 and 5) that are validated by BLASR alignment) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Aligns Illumina short reads to long reads inside HECIL to locate questionable bases prior to correction)
- **BLASR** (Aligns uncorrected and corrected long reads to the reference genome to compute PI, % matched bases, and aligned-read counts)
- **samtools** (Dependency of the HECIL pipeline for handling short-read alignments) — https://github.com/NDBL/HECIL

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- PI improves from 93.8 (uncorrected) to 98.6 (HECIL iteration 1) and 98.9 (iteration 5) on the S. cerevisiae dataset
- % matched bases improves from 78.8 (original) to 85.6 (iteration 1) and 86.5 (iteration 5)
- Number of aligned reads matches the paper's reported value (e.g., 346,242 for the uncorrected S. cerevisiae condition)
- Corrected iterations must outperform the uncorrected reads on all three metrics, with iteration 5 ≥ iteration 1
- The metrics grid contains exactly nine values (three conditions × three metrics) matching the reported table

## Limitations

- Iterative improvement is a conjecture without mathematical proof; the paper notes a rigorous framework for proving iterative improvement likely cannot be made without unrealistic stochastic assumptions
- Saturation behavior of the iterative procedure is unvalidated speculation; the authors only speculate metrics improve until saturation
- HECIL's iterative learning adds total execution time, and multi-threading speedup was not implemented (workload was split into 16 concurrent parallel tasks in the study)

## Evidence

- [results] aligning corrected long reads to reference genome with BLASR: "we align corrected long reads to its reference genome using BLASR21"
- [results] S. cerevisiae dataset size: "we use 1,758,169 long reads with 4,503,422 short reads"
- [results] S288C reference genome size: "The reference genome of strain S288C is 12.2 Mbp in size"
- [methods] Quick correction consensus threshold: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
- [methods] Confidence threshold for iterative learning: "Here we select α > 95 so that a small percentage of the optimal corrections are considered to be of high confidence."
- [readme] HECIL output file location: "The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta"
