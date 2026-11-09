---
name: probability-divergence-sequence-scoring
description: Use when comparing short DNA barcode sequences under nanopore-style error
  models (substitutions, insertions, deletions) — e.
when_to_use_negative:
- Reads whose flanking sequences are similar to the barcode itself — PRO cannot accurately
  extract the corrupted barcode and underperforms Guppy in that case
- Barcodes of unequal length or non-ONT error profiles where the pm/pd/ps/pi decomposition
  does not apply
- PacBio sequencing data — PRO is proposed for both ONT and PacBio but no PacBio tests
  have been performed
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: PRO
  role: Implements probability divergence, farthest-point-sampling barcode selection
    (pro_FPS_select.py), and PRO-dex demultiplexing
  repo: https://github.com/renztr/PRO.git
- name: edlib 1.3.9
  role: Semiglobal alignment of flanking sequences to reads to extract the corrupted
    barcode before divergence scoring
- name: deepsimulator 1.5
  role: Simulates ONT sequencing of ligated barcode libraries to generate corrupted
    barcodes for validation
  repo: https://github.com/liyu95/DeepSimulator
- name: tqdm
  role: Progress tracking in the PRO Python scripts
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_001
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:7c13678697ceb68dedf208d4a22d34f0d643b058779e422d3a9d2a7edf7b50c4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/rztongr/PRO
  repo_url: https://github.com/renztr/PRO.git
  tools:
  - PRO
  - Python
  - deepsimulator 1.5
  - edlib 1.3.9
  - tqdm
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# probability-divergence-sequence-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores dissimilarity between two short, equal-length DNA sequences using PRO's probability divergence, ProDiv(X,Y)=pm^m·pd^d·(1/3·ps)^s·(1/4·pi)^i, a probabilistic alternative to edit distance for selecting distinguishable barcodes and demultiplexing corrupted nanopore barcode reads.

## When to use

Use when comparing short DNA barcode sequences under nanopore-style error models (substitutions, insertions, deletions) — e.g., selecting a maximally distinguishable barcode kit or assigning corrupted barcodes extracted from ONT reads — where edit distance is the default but a probabilistic error-weighted score is warranted.

## When NOT to use

- Reads whose flanking sequences are similar to the barcode itself — PRO cannot accurately extract the corrupted barcode and underperforms Guppy in that case
- Barcodes of unequal length or non-ONT error profiles where the pm/pd/ps/pi decomposition does not apply
- PacBio sequencing data — PRO is proposed for both ONT and PacBio but no PacBio tests have been performed

## Inputs

- Equal-length candidate DNA barcode sequences (e.g., preprocessed 4^n k-mer space hashed to 64-bit integers, or select_barcode.txt)
- Corrupted barcode sequences s' extracted from simulated or real ONT reads (e.g., deepsimulator 1.5 FASTQ output)
- Flanking-sequence FASTQ file used in library preparation
- Sequencing error rate p and tolerated error count k for threshold conversion

## Outputs

- Pairwise probability-divergence scores between barcode sequences
- Threshold-filtered candidate set (candidates with ProDiv(s, seq1) <= t)
- Per-read barcode assignments from PRO-dex and demultiplexing accuracy (e.g., accuracy_comparison.tsv, fps_vs_random_comparison.tsv)
- select_barcode.txt with the FPS-selected barcode kit (e.g., 2,292 barcodes at 24 bp)

## How to apply

For two equal-length sequences X and Y differing by m matches, d deletions, s substitutions, and i insertions, compute ProDiv(X,Y)=pm^m·pd^d·(1/3·ps)^s·(1/4·pi)^i, where p is the sequencing error rate; lower ProDiv means the pair is less distinguishable under ONT errors. For barcode generation, convert the number of tolerated edit errors k and error rate p into a divergence threshold t (insertion worst case p'=(1-p)^n·(1/4·pi)^k for length n) and use the score as the weight in farthest-point sampling, discarding candidate s when ProDiv(s, seq1)>t. For demultiplexing (PRO-dex), first extract the corrupted barcode s' from each read by aligning the rear top/bottom flanking sequences with edlib against the narrowed ranges sequence'[35:77] and sequence'[35+startf:120], then assign s' to the barcode s0 with maximal pro(s0,s')=max{pro(s,s'), s in {barcode}}. Judge correctness by comparing accuracy against an edit-distance assignment run on the identical barcode set and reads.

## Related tools

- **PRO** (Implements probability divergence, farthest-point-sampling barcode selection (pro_FPS_select.py), and PRO-dex demultiplexing) — https://github.com/renztr/PRO.git
- **edlib 1.3.9** (Semiglobal alignment of flanking sequences to reads to extract the corrupted barcode before divergence scoring)
- **deepsimulator 1.5** (Simulates ONT sequencing of ligated barcode libraries to generate corrupted barcodes for validation) — https://github.com/liyu95/DeepSimulator
- **tqdm** (Progress tracking in the PRO Python scripts)

## Examples

```
python pro_FPS_select.py -l 10 -th 2 -a 0.88 -o examples/testoutput/k_mer_select -q 10000
```

## Evaluation signals

- The probability-divergence demultiplexing column shows higher accuracy than the edit-distance column on the same extracted corrupted barcodes (accuracy_comparison.tsv)
- FPS-selected sets have lower mean pairwise ProDiv and higher demultiplexing accuracy than size-matched random sets at 15, 20, and 24 bp (fps_vs_random_comparison.tsv)
- All pairwise ProDiv values within the generated kit are <= threshold t derived from p and k
- Kit size and length match expectations (e.g., exactly 2,292 barcodes of 24 nt in select_barcode.txt)
- PRO's demultiplexing accuracy reaches ~98.29% for a kit of size 2,922 and is comparable to Guppy when kit sizes match official ONT kits

## Limitations

- When flanking sequences are similar to the barcode, PRO cannot accurately extract the corrupted barcode and underperforms Guppy (observed on B1–B3)
- ONT's design strategy for B1–B3 does not ensure large enough probability divergence between barcodes, complicating comparisons
- PRO is claimed applicable to ONT and PacBio data, but no tests on PacBio data have been performed

## Evidence
