---
name: iterative-confidence-based-realignment
description: Use when hybrid-correcting PacBio/Sequel long reads with Illumina short
  reads and a single core-correction pass is insufficient; goal is to boost percent
  identity (e.g., E. coli PI 85.0 -> 97.2) via repeated realign-and-learn rounds.
when_to_use_negative:
- A single iteration suffices — HECIL's core algorithm (iteration 1) already generates
  the lowest number of unique k-mers for E. coli (PacBio and Sequel) and S. cerevisiae
  compared to CoLoRMap, so iterative learning is only for squeezing out further gains
  at extra runtime
- Reads are third-generation data without matching short reads — this skill is a hybrid-correction
  loop that requires short-read alignments; self-correcting methods (e.g., HGAP) are
  the alternative
- You need a mathematically guaranteed improvement at every round — the article notes
  iterative improvement is a conjecture with no rigorous proof and that performance
  is speculated to improve only until saturation
edam_operation: http://edamontology.org/operation_3194
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: HECIL
  role: Executes the confidence assignment (alpha > 95 percentile), quick correction
    (eta = 0.9 consensus), and optimization-based correction in each iterative round
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: Aligns short reads to the updated long reads between iterations to re-mark
    questionable positions for correction
- name: BLASR
  role: Aligns each iteration's corrected reads to the reference genome (e.g., E.
    coli K-12 MG1655 or S288C) to compute PI and % matched bases
- name: Jellyfish
  role: Counts unique k-mers after each iteration to monitor the relative reduction
    used in the primary termination criterion
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:a140ecc2519669ee39701a2409e496c729e56cc307302b5852e28b93d22ceb40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BLASR
  - BWA-MEM
  - HECIL
  - Jellyfish
  edam_operation: http://edamontology.org/operation_3194
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# iterative-confidence-based-realignment

## Summary

HECIL's iterative learning loop: high-confidence corrections (alpha > 95 percentile of optimal costs) are fixed, updated long reads are realigned to the short reads with BWA-MEM, and correction is repeated across successive iterations (up to 5, limited by the maximal iterations n) to progressively improve long-read accuracy.

## When to use

Use when hybrid-correcting PacBio/Sequel long reads with Illumina short reads and a single core-correction pass is insufficient; goal is to boost percent identity (e.g., E. coli PI 85.0 -> 97.2) via repeated realign-and-learn rounds.

## When NOT to use

- A single iteration suffices — HECIL's core algorithm (iteration 1) already generates the lowest number of unique k-mers for E. coli (PacBio and Sequel) and S. cerevisiae compared to CoLoRMap, so iterative learning is only for squeezing out further gains at extra runtime
- Reads are third-generation data without matching short reads — this skill is a hybrid-correction loop that requires short-read alignments; self-correcting methods (e.g., HGAP) are the alternative
- You need a mathematically guaranteed improvement at every round — the article notes iterative improvement is a conjecture with no rigorous proof and that performance is speculated to improve only until saturation

## Inputs

- Long reads FASTA (e.g., 33,360 filtered E. coli Sequel reads, or 1,758,169 S. cerevisiae reads)
- Illumina short reads FASTQ (e.g., 22,720,100 E. coli or 4,503,422 S. cerevisiae sequences)
- Short-read mean length (e.g., -len 202 for the HECIL.py CLI)
- Optional per-iteration outputs from the previous HECIL round (updated long reads to realign)

## Outputs

- Corrected long reads per iteration (LongRead_Corrected.fasta containing corrected and uncorrected reads)
- Per-iteration alignment metrics: percent identity (PI), % matched bases, number of aligned reads (e.g., pi_matched_bases_progression.csv with six rows, uncorrected through iteration 5)
- Unique k-mer counts per iteration (via Jellyfish) for termination-criterion monitoring

## How to apply

After HECIL's core pass, assign confidence by computing the alpha > 95 percentile of the optimal correction costs J_k = 1/2(w_q^k + w_s^k) (normalized PHRED-quality weight plus alignment-identity similarity weight), fixing only those high-confidence corrections. Realign the updated long reads to the short reads using BWA-MEM and repeat the quick-correction (eta = 0.9 consensus) and optimization-based correction steps, marking mismatched/inserted/deleted positions as questionable each round. Run four additional rounds after the core pass so corrections exist for iterations 1 through 5, using the maximal allowable iterations n as the secondary termination criterion (the paper's primary epsilon criterion uses the relative reduction in unique k-mers between successive iterations). At each iteration, evaluate progress by aligning uncorrected and corrected reads to the reference (e.g., BLASR against E. coli K-12 MG1655) and computing PI and % matched bases; e.g., Sequel E. coli PI rises from 85.0 to 95.6 at iteration 1 and 97.2 at iteration 5.

## Related tools

- **HECIL** (Executes the confidence assignment (alpha > 95 percentile), quick correction (eta = 0.9 consensus), and optimization-based correction in each iterative round) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Aligns short reads to the updated long reads between iterations to re-mark questionable positions for correction)
- **BLASR** (Aligns each iteration's corrected reads to the reference genome (e.g., E. coli K-12 MG1655 or S288C) to compute PI and % matched bases)
- **Jellyfish** (Counts unique k-mers after each iteration to monitor the relative reduction used in the primary termination criterion)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- PI and % matched bases increase monotonically from uncorrected baseline through iteration 5 (e.g., Sequel E. coli: 85.0/85.1 -> 95.6/95.6 -> 97.2/97.1)
- S. cerevisiae reproduction targets: PI 93.8 -> 98.6 (iter 1) -> 98.9 (iter 5); % matched bases 78.8 -> 85.6 -> 86.5; 346,242 aligned reads
- Unique k-mer count (Jellyfish) decreases or its relative reduction falls below epsilon between successive iterations
- Only corrections with optimal cost above the alpha > 95 percentile threshold are fixed in each round; low-confidence sites remain revisitable
- Output FASTA contains both corrected and uncorrected long reads (LongRead_Corrected.fasta) as documented in the README

## Limitations

- Iterative improvement is only a conjecture; no rigorous mathematical framework exists, and it may require unrealistic stochastic assumptions
- Saturation behavior of the iterative procedure is unvalidated speculation
- The current HECIL version has no multi-threading speedup; workload must be split into 16 concurrent data-parallel tasks run in parallel
- Maximal number of iterations n is only a secondary termination criterion; the primary epsilon criterion on unique k-mer reduction may trigger earlier
- Iterative learning increases total execution time relative to a single core-algorithm pass

## Evidence

- [methods] Here we select α > 95 so that a small percentage of the optimal corrections are considered to be of high confidence.: "Here we select α > 95 so that a small percentage of the optimal corrections are considered to be of high confidence."
- [methods] We learn in successive iterations by realigning the updated long reads to the short reads.: "We learn in successive iterations by realigning the updated long reads to the short reads."
- [methods] we also recommend selecting a secondary termination criterion: the maximal number n of allowable iterations for iterative learning.: "we also recommend selecting a secondary termination criterion: the maximal number n of allowable iterations for iterative learning."
- [discussion] The current version of HECIL allows decomposition of the workload into independent data-parallel tasks that can be executed simultaneously.: "The current version of HECIL allows decomposition of the workload into independent data-parallel tasks that can be executed simultaneously."
- [readme] The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta: "The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta"
