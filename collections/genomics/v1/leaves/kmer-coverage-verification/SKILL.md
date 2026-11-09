---
name: kmer-coverage-verification
description: 'Use when auditing synthetic training sequences for k-mer-based RNA modification
  models (e.g., EpiNano m6A SVM): check FASTA inputs enumerate every 5-mer and confirm
  secondary-structure minimization via MFE folding.'
when_to_use_negative:
- Input is already a per-k-mer base-calling error feature table (EpiNano output) —
  use EpiNano-Error/SVM prediction instead of coverage auditing.
- Sequences are natural transcripts rather than designed synthetic training substrates,
  where exhaustive 5-mer coverage is not a design goal.
- The task is m6A site prediction on new nanopore reads; this skill only validates
  the training-sequence design, not the prediction itself.
tools:
- name: Biopython
  role: Parses the synthetic Curlcake FASTA sequences and enumerates 5-mers via sliding
    window
- name: RNAfold (ViennaRNA)
  role: Computes per-sequence minimum free energy (MFE) secondary structure to confirm
    Curlcake minimization
- name: EpiNano
  role: 'Downstream consumer: its m6A SVM models are trained on the synthetic sequences
    whose 5-mer coverage this skill verifies'
  repo: https://github.com/novoalab/EpiNano
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:759d47462c1e679593f77875a3dcbcc56a909178fc71267b4edb0b3be2a18cf9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - Biopython
  - EpiNano
  - RNAfold (ViennaRNA)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# kmer-coverage-verification

## Summary

Verifies that a set of synthetic RNA training sequences (e.g., Curlcake-designed sequences used to train EpiNano's m6A models) covers all possible 4^5 = 1024 RNA 5-mers while keeping predicted RNA secondary structure minimal, so that k-mer-specific base-calling 'error' features are well represented during model training.

## When to use

Use when auditing synthetic training sequences for k-mer-based RNA modification models (e.g., EpiNano m6A SVM): check FASTA inputs enumerate every 5-mer and confirm secondary-structure minimization via MFE folding.

## When NOT to use

- Input is already a per-k-mer base-calling error feature table (EpiNano output) — use EpiNano-Error/SVM prediction instead of coverage auditing.
- Sequences are natural transcripts rather than designed synthetic training substrates, where exhaustive 5-mer coverage is not a design goal.
- The task is m6A site prediction on new nanopore reads; this skill only validates the training-sequence design, not the prediction itself.

## Inputs

- Synthetic Curlcake sequence FASTA files (four deposited sequences: 2329, 2543, 2678, and 2795 bp)
- Reference set of all 1024 possible RNA 5-mers (4^5)

## Outputs

- Per-sequence coverage table with sequence length, distinct 5-mer count, and missing 5-mers
- Per-sequence RNAfold minimum free energy (MFE) values
- Consolidated CSV report (curlcake_coverage_report.csv)

## How to apply

Load the deposited synthetic Curlcake FASTA files (four sequences of 2329, 2543, 2678, and 2795 bp) with Biopython and concatenate them. Slide a 5-nt window across the concatenated sequences, tally the distinct 5-mers observed, and compare against the complete 1024 (4^5) set of possible RNA 5-mers; the paper's design targeted a median occurrence of each 5-mer of 10. Compute the minimum free energy (MFE) secondary structure of each sequence with RNAfold (ViennaRNA) to confirm the Curlcake-designed minimization of secondary structure. Consolidate per-sequence length, distinct 5-mer count, missing 5-mers (if any), and MFE values into a single report table (e.g., curlcake_coverage_report.csv). This matters because the EpiNano models were trained on modified/unmodified synthetic sequences, and the article notes that feature contributions vary across different k-mers, so gaps in 5-mer coverage would degrade k-mer-specific accuracy.

## Related tools

- **Biopython** (Parses the synthetic Curlcake FASTA sequences and enumerates 5-mers via sliding window)
- **RNAfold (ViennaRNA)** (Computes per-sequence minimum free energy (MFE) secondary structure to confirm Curlcake minimization)
- **EpiNano** (Downstream consumer: its m6A SVM models are trained on the synthetic sequences whose 5-mer coverage this skill verifies) — https://github.com/novoalab/EpiNano

## Examples

```
python -c "from Bio import SeqIO; s=''.join(str(r.seq) for r in SeqIO.parse('curlcake_seqs.fasta','fasta')); print(len({s[i:i+5] for i in range(len(s)-4)}))" ; RNAfold < curlcake_1.fasta
```

## Evaluation signals

- The set of distinct 5-mers observed across the concatenated sequences equals all 1024 (4^5) possible RNA 5-mers with no missing k-mers.
- Median occurrence of each 5-mer across the synthetic sequences is approximately 10, matching the reported design.
- RNAfold MFE values per sequence are low (structure-minimized), consistent with Curlcake design intent.
- The consolidated CSV contains one row per input sequence with non-empty length, distinct 5-mer count, and MFE fields, and the four sequence lengths are 2329, 2543, 2678, and 2795 bp.

## Limitations

- Coverage is assessed over concatenated sequences, so individual sequence files may not each contain every 5-mer.
- The article notes that 'the contribution of each feature varies across different k-mers', so full 5-mer coverage alone does not guarantee uniform model accuracy across k-mers; k-mer-specific training was recommended.
- EpiNano, the downstream model, aggregates information across reads at a site rather than per-read, so sequence-level coverage checks do not address single-read prediction quality.

## Evidence

- [abstract] our algorithm, trained with m 6 A-modified and unmodified synthetic sequences, can predict m 6 A RNA modifications with ~90% accuracy: "our algorithm, trained with m 6 A-modified and unmodified synthetic sequences, can predict m 6 A RNA modifications with ~90% accuracy"
- [readme] The available m6A SVM models has been trained and tested upon a set of 'unmodified' and 'modified' sequences containing m6A at known sites or A.: [evidence span withheld — anchor status: external_doc]
- [discussion] the contribution of each feature varies across different k-mers: "the contribution of each feature varies across different k-mers"
