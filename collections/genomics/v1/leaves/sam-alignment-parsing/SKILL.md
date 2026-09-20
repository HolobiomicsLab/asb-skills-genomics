---
name: sam-alignment-parsing
description: Use when short reads have been aligned to long reads with BWA-MEM and
  the SAM output must be parsed to locate mismatches/indels, extract PHRED qualities,
  and compute alignment identities that drive base-correction decisions.
when_to_use_negative:
- The correction pipeline is self-correcting (HGAP-style, no short-read data), since
  there are no BWA-MEM SAM alignments of Illumina reads to parse
- The base has already reached at least 90% consensus (η = 0.9) and is eligible for
  HECIL's quick correction instead of the optimization-based stage
- Alignments come from a tool/format other than the BWA-MEM/samtools SAM chain HECIL
  is built around
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: BWA
  role: Aligns short reads to long reads (BWA-MEM) producing the SAM records whose
    mismatches, insertions, and deletions define questionable bases
- name: Samtools
  role: SAM file handling/post-processing in the HECIL pipeline (Align_Corr.sh depends
    on it)
- name: HECIL
  role: 'Consumes the parsed alignments: computes quality/similarity decision weights
    and applies the optimization-based correction to questionable bases'
  repo: https://github.com/NDBL/HECIL
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:0518184477bf5dc913f42d0c589ed45e68a502c3d1dc43f74818477ed3955203
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA
  - BWA MEM
  - BWA-MEM
  - HECIL
  - Samtools
  - TULIP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# sam-alignment-parsing

## Summary

Parses BWA-MEM SAM alignments of Illumina short reads onto PacBio/Oxford Nanopore long reads (HECIL) to flag mismatch/insertion/deletion positions as questionable bases and extract per-read PHRED quality and alignment identity, which serve as decision weights for HECIL's optimization-based correction.

## When to use

Use when short reads have been aligned to long reads with BWA-MEM and the SAM output must be parsed to locate mismatches/indels, extract PHRED qualities, and compute alignment identities that drive base-correction decisions.

## When NOT to use

- The correction pipeline is self-correcting (HGAP-style, no short-read data), since there are no BWA-MEM SAM alignments of Illumina reads to parse
- The base has already reached at least 90% consensus (η = 0.9) and is eligible for HECIL's quick correction instead of the optimization-based stage
- Alignments come from a tool/format other than the BWA-MEM/samtools SAM chain HECIL is built around

## Inputs

- Long reads in FASTA (e.g. LongRead.fa, passed as -l LR)
- Short reads in FASTQ with PHRED quality scores (e.g. ShortRead.fq, passed as -s SR)
- Short-read length parameter (e.g. -len 202)
- BWA-MEM SAM alignment records of short reads onto long reads

## Outputs

- Set of questionable base positions on each long read (SAM mismatch/insertion/deletion sites)
- Per-candidate decision weights: normalized quality weight and normalized similarity weight
- Selected correction per questionable base, written to corrected_bases.tsv
- Corrected long reads in LongRead_Corrected.fasta

## How to apply

Align short reads (FASTQ) to long reads (FASTA) using BWA-MEM (tested on BWA 0.7.15) with samtools (0.1.18) post-processing, under Python 2.7.8 as in the HECIL repository. Parse the SAM records to mark every position with a mismatch, insertion, or deletion on the long read as a questionable base, since these are the bases handed to the optimization-based correction stage. For each questionable base, collect the aligning short reads and compute the normalized quality weight as the short read's PHRED score from the FASTQ divided by the maximum quality among the aligning reads, and the normalized similarity weight as the number of exact matches divided by the short read's length. Compute the cost J_k = 1/2*(quality weight + similarity weight), select the read k* = argmax J_k (breaking ties by highest quality), and replace the questionable base with the winning read's aligned base. Rationale: the paper argues these localized signals (base quality of short reads, mapping identity) are not systematically used by most hybrid correctors, so parsing them out of the alignments is what enables the weighted policy.

## Related tools

- **BWA** (Aligns short reads to long reads (BWA-MEM) producing the SAM records whose mismatches, insertions, and deletions define questionable bases)
- **Samtools** (SAM file handling/post-processing in the HECIL pipeline (Align_Corr.sh depends on it))
- **HECIL** (Consumes the parsed alignments: computes quality/similarity decision weights and applies the optimization-based correction to questionable bases) — https://github.com/NDBL/HECIL

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- Every flagged questionable base corresponds to a SAM-record mismatch, insertion, or deletion between a short read and the long read (no spurious positions)
- Normalized quality weight equals the short read's PHRED score divided by the maximum quality among the aligning reads
- Normalized similarity weight equals the number of exact matches to the long read divided by the short read's length
- Cost J_k is exactly the equal-weight (1/2, 1/2) combination of the two weights, and selection is argmax with highest-quality tie-breaking
- Each corrected base recorded in corrected_bases.tsv equals the aligned base of the selected winning short read

## Limitations

- Dependencies are pinned to old versions (BWA 0.7.15, Samtools 0.1.18, Python 2.7.8); executables may need chmod +x after cloning
- Iterative improvement over successive realignments remains a conjecture with no mathematical proof
- Multi-threading speedup is not implemented in the current version; workloads must be split manually into concurrent tasks

## Evidence

- [methods] For the remaining questionable bases, we employ an optimization-based correction framework.: "For the remaining questionable bases, we employ an optimization-based correction framework."
- [intro] a core algorithm that selects a correction policy by leveraging an optimal combination of decision weights based on base quality and mapping identity of aligned short reads: "a core algorithm that selects a correction policy by leveraging an optimal combination of decision weights based on base quality and mapping identity of aligned short reads"
- [readme] BWA (tested on version 0.7.15), Samtools (tested on version 0.1.18), Python (tested on version 2.7.8): "BWA (tested on version 0.7.15), Samtools (tested on version 0.1.18), Python (tested on version 2.7.8)"
- [methods] we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
