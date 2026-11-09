---
name: synthetic-sequence-design-validation
description: Use when synthetic RNA sequences intended for training or benchmarking
  a base-calling/modification-detection model must be checked for complete k-mer coverage
  and minimized secondary structure before training.
when_to_use_negative:
- Input is already a curated training dataset with documented k-mer coverage — no
  re-validation needed
- Goal is per-read modification prediction quality assessment rather than validation
  of the synthetic training set itself
- The sequences are natural transcripts (e.g. yeast mRNAs), which cannot be k-mer-complete
  by design
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_0097
- http://edamontology.org/topic_0099
tools:
- name: Biopython
  role: Load the synthetic curlcake FASTA files and enumerate sliding-window 5-mers
    for coverage tallies
- name: RNAfold (ViennaRNA)
  role: Compute minimum free energy (MFE) secondary structure of each synthetic sequence
    to confirm secondary-structure minimization
- name: EpiNano
  role: 'Downstream consumer: the validated synthetic m6A-modified/unmodified sequences
    train the EpiNano-SVM m6A models'
  repo: github.com/enovoa/EpiNano
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:a65a29c704163b00eb9f16e65b6e10efe347e124ca8540d23aa27282b87ffaa4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biopython
  - EpiNano
  - RNAfold (ViennaRNA)
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_0097
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# synthetic-sequence-design-validation

## Summary

Verify that synthetic training sequences (e.g. the 'curlcake' sequences used to train EpiNano's m6A SVM models) cover the complete k-mer space — all 1024 possible RNA 5-mers — while keeping RNA secondary structure (minimum free energy) minimized, so that an algorithm trained on them is not biased toward specific sequence contexts.

## When to use

Use when synthetic RNA sequences intended for training or benchmarking a base-calling/modification-detection model must be checked for complete k-mer coverage and minimized secondary structure before training.

## When NOT to use

- Input is already a curated training dataset with documented k-mer coverage — no re-validation needed
- Goal is per-read modification prediction quality assessment rather than validation of the synthetic training set itself
- The sequences are natural transcripts (e.g. yeast mRNAs), which cannot be k-mer-complete by design

## Inputs

- Synthetic curlcake sequence FASTA files (four sequences of 2329, 2543, 2678, and 2795 bp)
- k-mer length parameter (k=5, giving 4^5 = 1024 possible RNA 5-mers)

## Outputs

- curlcake_coverage_report.csv with per-sequence length, distinct 5-mer count, missing 5-mers (if any), and MFE values
- Per-sequence RNAfold minimum free energy (MFE) secondary structure predictions

## How to apply

Load the deposited synthetic curlcake sequence FASTA files (2329, 2543, 2678, and 2795 bp) with Biopython, concatenate them, and enumerate every 5-mer in a sliding window, tallying distinct observed 5-mers against the complete set of 1024 (4^5) possible RNA 5-mers. In the curlcake design each 5-mer occurs a median of 10 times, so report per-sequence sequence length, distinct 5-mer count, and any missing 5-mers. Independently compute the minimum free energy (MFE) secondary structure of each sequence with RNAfold (ViennaRNA) to confirm the design's minimization of RNA secondary structure. Consolidate all metrics into a verification table (curlcake_coverage_report.csv); passing validation means all 1024 5-mers are present at roughly uniform (median ~10) occurrence and MFE values reflect the intended structural minimization.

## Related tools

- **Biopython** (Load the synthetic curlcake FASTA files and enumerate sliding-window 5-mers for coverage tallies)
- **RNAfold (ViennaRNA)** (Compute minimum free energy (MFE) secondary structure of each synthetic sequence to confirm secondary-structure minimization)
- **EpiNano** (Downstream consumer: the validated synthetic m6A-modified/unmodified sequences train the EpiNano-SVM m6A models) — github.com/enovoa/EpiNano

## Examples

```
python -c "from Bio import SeqIO; import itertools; seqs=''.join(str(r.seq) for r in SeqIO.parse('curlcake.fasta','fasta')); kmers={seqs[i:i+5] for i in range(len(seqs)-4)}; print(len(kmers), set(''.join(p) for p in itertools.product('ACGU',repeat=5))-kmers)"
```

## Evaluation signals

- Distinct observed 5-mer count equals 1024 (all possible 4^5 RNA 5-mers) across the concatenated sequences
- Median occurrence of each 5-mer is ~10 in the curlcake design; flag missing 5-mers as validation failures
- RNAfold MFE values per sequence confirm secondary structure is minimized relative to random sequences of equal length
- Consolidated verification table (curlcake_coverage_report.csv) has one row per deposited sequence (2329, 2543, 2678, 2795 bp) with length, 5-mer count, missing 5-mers, and MFE populated

## Limitations

- Coverage and MFE validation only certifies the training sequences themselves; the trained algorithm still cannot predict modifications in individual RNA molecules (it aggregates reads per site) and recovers only ~32% of known sites
- The downstream EpiNano-SVM models are base-caller dependent — SVM predictions are accurate only for data base-called with a matching base-caller version (Albacore 2.1.7 for EpiNano 1.0/1.1; Guppy 3.1.5 for EpiNano 1.2)
- The feature contribution varies across different k-mers, so k-mer-specific training is needed to improve accuracy and reduce false positives; full 5-mer coverage does not remove k-mer-dependent performance variation

## Evidence

- [abstract] Training-sequence k-mer completeness is the design basis: "our algorithm, trained with m 6 A-modified and unmodified synthetic sequences, can predict m 6 A RNA modifications with ~90% accuracy"
- [readme] Synthetic sequences must contain m6A at known sites or unmodified A: [evidence span withheld — anchor status: external_doc]
- [readme] EpiNano-SVM models are base-caller dependent, constraining reuse of the validated training data: [evidence span withheld — anchor status: external_doc]
- [abstract] Workflow step that consumes the validated synthetic sequences: [evidence span withheld — anchor status: not_found]
