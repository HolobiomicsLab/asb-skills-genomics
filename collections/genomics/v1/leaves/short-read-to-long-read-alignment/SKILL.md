---
name: short-read-to-long-read-alignment
description: Use when a hybrid error-correction task requires mapping short reads
  (e.g. Illumina FASTQ) onto long reads (e.g. PacBio/Sequel FASTA) to localize questionable
  bases before consensus- or optimization-based correction.
when_to_use_negative:
- The long reads are already high quality or the goal is self-correction without short-read
  data (e.g. HGAP-style self-correcting approaches) — this skill is specifically hybrid
  correction driven by aligned short reads
- Input short reads are not from a second-generation technology with per-base quality
  (e.g. 454 pyro-sequencing output), since HECIL's correction weights depend on PHRED-derived
  quality and mapping identity of the aligned short reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: BWA-MEM
  role: Aligns Illumina short reads to the long reads with previously reported parameters;
    re-run for realignment at each iterative-learning step
  repo: https://github.com/NDBL/HECIL
- name: Samtools
  role: Processes the BWA-MEM SAM/BAM output so disagreement positions can be parsed
    from the alignment
  repo: https://github.com/NDBL/HECIL
- name: HECIL
  role: Wraps the alignment via Align_Corr.sh and uses the flagged questionable positions
    as input to Quick Correction (eta = 0.9) and optimization-based correction
  repo: https://github.com/NDBL/HECIL
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_007
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:057ecf597ff45845c1b37b20c8a0f64ebb2bb1550ac8256898468164e19db9a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# short-read-to-long-read-alignment

## Summary

Aligns high-accuracy Illumina short reads onto noisy third-generation long reads with BWA-MEM and marks mismatch, insertion, and deletion positions on the long reads as questionable, providing the evidence layer that HECIL's correction stages consume.

## When to use

Use when a hybrid error-correction task requires mapping short reads (e.g. Illumina FASTQ) onto long reads (e.g. PacBio/Sequel FASTA) to localize questionable bases before consensus- or optimization-based correction.

## When NOT to use

- The long reads are already high quality or the goal is self-correction without short-read data (e.g. HGAP-style self-correcting approaches) — this skill is specifically hybrid correction driven by aligned short reads
- Input short reads are not from a second-generation technology with per-base quality (e.g. 454 pyro-sequencing output), since HECIL's correction weights depend on PHRED-derived quality and mapping identity of the aligned short reads

## Inputs

- Long reads in FASTA format (e.g. 33,360 filtered E. coli PacBio reads, reads shorter than 100 bp excluded; S. cerevisiae S288C 1,758,169 reads; A. funestus flowcell reads)
- Short reads in FASTQ format with known read length (e.g. 22,720,100 E. coli Illumina reads, length 202 passed via -len; 4,503,422 S. cerevisiae short reads)
- BWA index of the long-read FASTA built by BWA-MEM (version 0.7.15)

## Outputs

- SAM/BAM alignment of short reads against long reads, processed with Samtools 0.1.18
- Set of questionable positions (mismatches, insertions, deletions) per long read consumed by the Quick Correction and optimization-based correction modules
- Realigned alignments for each iterative-learning round feeding the confidence threshold recomputation

## How to apply

Run BWA-MEM (tested on version 0.7.15 in the HECIL repository) with the previously reported parameters to align the Illumina short reads against the long reads, then use Samtools (tested on version 0.1.18) to process the resulting SAM/BAM. Parse the alignment and mark every long-read position showing a mismatch, insertion, or deletion relative to the aligned short reads as questionable; only these positions become candidates for HECIL's Quick Correction (eta = 0.9 consensus) and optimization-based correction (J_k = 1/2(w_q + w_s)) stages. In HECIL's iterative learning loop, this alignment step is repeated after each round: high-confidence corrections (alpha > 95 percentile of optimal costs) are fixed, updated long reads are realigned to the short reads, and the confidence threshold mu^(l) is recomputed. The alignment is data-parallel — HECIL's workload was split into 16 concurrent tasks, matching the 16 threads used for baselines like proovread, LoRDEC, and CoLoRMap.

## Related tools

- **BWA-MEM** (Aligns Illumina short reads to the long reads with previously reported parameters; re-run for realignment at each iterative-learning step) — https://github.com/NDBL/HECIL
- **Samtools** (Processes the BWA-MEM SAM/BAM output so disagreement positions can be parsed from the alignment) — https://github.com/NDBL/HECIL
- **HECIL** (Wraps the alignment via Align_Corr.sh and uses the flagged questionable positions as input to Quick Correction (eta = 0.9) and optimization-based correction) — https://github.com/NDBL/HECIL

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- The alignment step is internal to HECIL: verify the output FASTQ/FASTA contains the same number of long reads as the input (e.g. 33,360 for the filtered E. coli set)
- Only long-read positions with mismatches, insertions, or deletions relative to aligned short reads should be flagged questionable; positions meeting the >= 90% (eta = 0.9) consensus may be quick-corrected, all others left untouched by this stage
- After iterative realignment, percent identity and % matched bases against the reference (e.g. E. coli K-12 MG1655 via BLASR) must increase monotonically relative to the uncorrected baseline (e.g. PI 85.0 -> 95.6 at iteration 1 -> 97.2 at iteration 5 for Sequel E. coli)
- Unique k-mer counts (Jellyfish) should decrease monotonically across iterations until the relative improvement falls below epsilon or the maximum n iterations is reached
- Executables must be runnable after cloning: if Align_Corr.sh, bwa, and samtools cannot execute, run chmod +x on them as documented in the README troubleshooting

## Limitations

- BWA-MEM and Samtools versions are pinned by testing (0.7.15 and 0.1.18); other versions are unverified
- Multi-threading speedup of HECIL itself is not implemented — parallelism is only via decomposition into 16 independent data-parallel tasks, at the expense of total execution time
- Iterative improvement is a conjecture without mathematical proof; saturation behavior of repeated realignment is unvalidated
- Python 2.7.8 dependency: the HECIL.py entry point was tested only on that version

## Evidence

- [methods] BWA-MEM alignment marks questionable positions: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
- [methods] Realignment drives iterative learning: "We learn in successive iterations by realigning the updated long reads to the short reads."
- [results] E. coli filtering before alignment: "We filter long reads of E. coli to exclude reads shorter than 100 bp, creating a final set of 33,360"
- [readme] BWA and Samtools dependencies and versions: "BWA (tested on version 0.7.15),
Samtools (tested on version 0.1.18),
Python (tested on version 2.7.8)"
- [readme] Executable permission troubleshooting for the alignment wrapper: "If Align_Corr.sh, bwa, and samtools cannot execute after cloning the repository, run the following commands:
    chmod +x bwa"
