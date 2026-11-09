---
name: quality-weighted-consensus-correction
description: Use when high-error long reads (PacBio or Oxford Nanopore) must be hybrid-corrected
  with quality-bearing Illumina short reads, and mismatch/indel positions on the long
  reads can be flagged as questionable sites.
when_to_use_negative:
- Only long reads are available with no short-read companion data (e.g., self-correction
  settings like HGAP), since HECIL is a hybrid method depending on Illumina alignments
- Short reads lack PHRED base qualities, because the optimization weights require
  quality-derived weights w_q
- The long reads are already corrected or error-free, so marking and correcting questionable
  bases would be moot
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3168
tools:
- name: HECIL
  role: Implements quick correction (eta = 0.9), optimization-based correction (J_k
    = 1/2*(w_q + w_s)), and confidence assignment/realignment iterations
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: Aligns Illumina short reads to long reads so mismatch, insertion, and deletion
    positions can be marked questionable
  repo: https://github.com/NDBL/HECIL
- name: Samtools
  role: Bundled dependency of HECIL for handling the BWA-MEM alignments used to derive
    correction weights
  repo: https://github.com/NDBL/HECIL
- name: Jellyfish
  role: Counts unique k-mers after each correction to monitor improvement and drive
    the epsilon termination criterion
- name: BLASR
  role: Aligns corrected long reads to a reference genome to compute PI and percent
    matched bases for validation
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:90820d4ca7de958a03033231e2a05a875986833a72437960e04b4b001a53d27e
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
  - PacBioToCA
  - QUAST
  - Samtools
  - proovread
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# quality-weighted-consensus-correction

## Summary

HECIL's hybrid long-read correction policy: align Illumina short reads to erroneous long reads (PacBio error rates up to 20%, Oxford Nanopore up to 35%), mark mismatch/insertion/deletion positions as questionable, then correct via quick consensus (eta = 0.9) or an optimization over a convex combination of normalized base-quality and alignment-identity weights, J = 0.5*(w_q + w_s).

## When to use

Use when high-error long reads (PacBio or Oxford Nanopore) must be hybrid-corrected with quality-bearing Illumina short reads, and mismatch/indel positions on the long reads can be flagged as questionable sites.

## When NOT to use

- Only long reads are available with no short-read companion data (e.g., self-correction settings like HGAP), since HECIL is a hybrid method depending on Illumina alignments
- Short reads lack PHRED base qualities, because the optimization weights require quality-derived weights w_q
- The long reads are already corrected or error-free, so marking and correcting questionable bases would be moot

## Inputs

- Erroneous long reads (e.g., PacBio/Sequel FASTA), E. coli K-12 MG1655 filtered to exclude reads < 100 bp (33,360 reads)
- High-quality Illumina short reads with PHRED base qualities (FASTQ; e.g., 22,720,100 E. coli sequences, 37,797,235 A. funestus reads)
- Short read length parameter (-len, e.g. 202) used by HECIL.py
- Optional reference genome for alignment-based validation (e.g., Anopheles funestus Fumoz, GenBank GCA_000349085.1)

## Outputs

- Corrected long-read FASTA containing all (corrected and uncorrected) reads, saved by HECIL as LongRead_Corrected.fasta
- Per-position correction decisions with confidence classes (high-confidence at alpha > 95 percentile of optimal costs) for subsequent realignment iterations
- Corrected read sets per iteration (e.g., iteration 1 and iteration 5) feeding Jellyfish k-mer counts and BLASR PI / % matched bases metrics

## How to apply

First mark as questionable every position on a long read where aligned short reads disagree (mismatches, insertions, deletions) after BWA-MEM alignment. Apply quick correction: replace a questionable base only when at least 90% of aligned short reads reach consensus at that position (eta = 0.9). For remaining questionable bases, run optimization-based correction: normalize PHRED-derived quality weights and alignment-identity weights (exact matches / short read length), then replace each base with that of the short read maximizing the convex-combination cost J_k = 1/2*(w_q + w_s), breaking ties by highest base quality. Optionally extend with iterative learning: assign high-confidence corrections at the alpha > 95 percentile of optimal costs, fix those sites, realign updated long reads to short reads each iteration, and terminate when the relative improvement in Jellyfish unique k-mers falls below epsilon or a maximal iteration count n is reached. As a validation layer, compute PI and percent matched bases by aligning corrected long reads to a reference (e.g., BLASR against the A. funestus Fumoz reference GCA_000349085.1).

## Related tools

- **HECIL** (Implements quick correction (eta = 0.9), optimization-based correction (J_k = 1/2*(w_q + w_s)), and confidence assignment/realignment iterations) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Aligns Illumina short reads to long reads so mismatch, insertion, and deletion positions can be marked questionable) — https://github.com/NDBL/HECIL
- **Samtools** (Bundled dependency of HECIL for handling the BWA-MEM alignments used to derive correction weights) — https://github.com/NDBL/HECIL
- **Jellyfish** (Counts unique k-mers after each correction to monitor improvement and drive the epsilon termination criterion)
- **BLASR** (Aligns corrected long reads to a reference genome to compute PI and percent matched bases for validation)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- Quick correction fires only when >= 90% of aligned short reads agree at a questionable position (eta = 0.9)
- Unique k-mer counts (Jellyfish) decrease monotonically across iterations until relative improvement drops below epsilon
- PI rises toward reported values (e.g., PI from 94.5 to 98.5 on A. funestus flowcell 1) and matched-base improvement matches reported deltas (e.g., 87.5% matched bases with 3.8% improvement on flowcell 16 at iteration 5)
- High-confidence sites are restricted to the alpha > 95 percentile of optimal correction costs and are fixed across subsequent realignments
- Output FASTA retains all input long reads (corrected plus uncorrected), with E. coli input matching the 33,360 filtered reads

## Limitations

- Iterative improvement is a conjecture with no rigorous mathematical proof; a proof likely requires possibly unrealistic stochastic assumptions
- Saturation behavior of the iterative procedure (e.g., continued improvement in unique k-mers) is unvalidated speculation
- Multi-threading is not implemented internally; parallelism is achieved only by manually splitting the workload into independent data-parallel tasks (e.g., 16 concurrent tasks)
- README requires executable permissions after cloning: chmod +x on bwa, samtools, and Align_Corr.sh; tested only on BWA 0.7.15, Samtools 0.1.18, and Python 2.7.8

## Evidence

- [methods] we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
- [methods] For the remaining questionable bases, we employ an optimization-based correction framework.: "For the remaining questionable bases, we employ an optimization-based correction framework."
- [methods] Here we select α > 95 so that a small percentage of the optimal corrections are considered to be of high confidence.: "Here we select α > 95 so that a small percentage of the optimal corrections are considered to be of high confidence."
- [intro] a core algorithm that selects a correction policy by leveraging an optimal combination of decision weights based on base quality and mapping identity of aligned short reads: "a core algorithm that selects a correction policy by leveraging an optimal combination of decision weights based on base quality and mapping identity of aligned short reads"
- [readme] usage: python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE]: "usage: python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE]"
