---
name: sequence-profile-alignment-and-refinement
description: Use when aligning ultra-long (billion-base) nucleotide FASTA datasets
  that were segmented by partial-chain MEM detection and the resulting subset blocks
  must be combined into one seamless multiple sequence alignment.
when_to_use_negative:
- Sequences are short or few, so the overhead of MEM-based segmentation and parallel
  subMSA is unnecessary — a standard MSA tool alone suffices.
- Input is protein sequences — FMAlign2 targets nucleotide (DNA) alignment.
- You need a single fixed alignment backend and cannot supply FASTA-in/FASTA-out commands;
  FMAlign2 requires the MSA tool to accept and emit FASTA.
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: FMAlign2
  role: 'Main driver: segments sequences via suffix-array MEMs, runs parallel subMSA,
    performs sequence-profile alignment and gap-based refinement to concatenate blocks'
  repo: https://github.com/malabz/FMAlign2
- name: MAFFT
  role: Default pluggable MSA backend (-p mafft) for aligning segmented subMSA blocks
    in parallel
  repo: https://mafft.cbrc.jp/alignment/software/
- name: Clustal Omega
  role: Alternative MSA backend selectable with -p clustalo
  repo: http://www.clustal.org/omega/
- name: HAlign3
  role: Alternative MSA backend selectable with -p halign3 (JAR, K-ary suffix-tree
    based center star strategy)
  repo: https://github.com/malabz/HAlign-3
- name: HAlign4
  role: Alternative MSA backend selectable with -p halign4 for aligning very large
    numbers of sequences
  repo: https://github.com/pinglu-zhang/HAlign-4
- name: sp.py
  role: Evaluation script that computes the Sum-of-Pairs score of the final alignment
  repo: https://github.com/malabz/FMAlign2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:99c01eae2192f2d63584e7742907c25fe09333f3259b3885b342e08b42556498
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/malabz/FMAlign2
  tools:
  - Clustal Omega
  - FMAlign2
  - HAlign3
  - HAlign4
  - MAFFT
  - sp.py
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# sequence-profile-alignment-and-refinement

## Summary

FMAlign2's Step 3 module: after partial-chain segmentation of ultra-long nucleotide sequences, segmented subMSA blocks are aligned in parallel by a pluggable backend (MAFFT, Clustal Omega, HAlign3, or HAlign4), and unmatched fragments are stitched into the backbone via sequence-profile alignment with gap-based refinement during block concatenation.

## When to use

Use when aligning ultra-long (billion-base) nucleotide FASTA datasets that were segmented by partial-chain MEM detection and the resulting subset blocks must be combined into one seamless multiple sequence alignment.

## When NOT to use

- Sequences are short or few, so the overhead of MEM-based segmentation and parallel subMSA is unnecessary — a standard MSA tool alone suffices.
- Input is protein sequences — FMAlign2 targets nucleotide (DNA) alignment.
- You need a single fixed alignment backend and cannot supply FASTA-in/FASTA-out commands; FMAlign2 requires the MSA tool to accept and emit FASTA.

## Inputs

- Nucleotide sequence dataset in FASTA format (ultra-long sequences)
- Partial-chains segmentation artifact from the FMAlign2 chain-formation step
- Optional custom MSA command file (FASTA-to-FASTA command template with {input}, {output}, optional {thread} placeholders)

## Outputs

- Aligned multiple sequence alignment in FASTA format (output.fasta)
- SP (Sum-of-Pairs) score printed by sp.py

## How to apply

Load the input nucleotide FASTA plus the partial-chains segmentation artifact from the suffix-array/MEM step, then invoke FMAlign2 with -i input.fasta -o output.fasta. Choose the pluggable MSA backend with -p (default mafft; alternatives clustalo, halign3, halign4, or a path to a custom command file) and tune MEM detection with -l (minimum MEM length, default 30) and -f (filtering mode, default accurate; switch to fast to speed up at the cost of sensitivity). Let the chosen backend align the segmented subMSA blocks in parallel (vertical division strategy), while unmatched fragments are incorporated into the backbone using the sequence-profile alignment strategy. Apply the gap-based refinement during concatenation of the aligned subset blocks to yield the final seamless MSA. Evaluate the result by running sp.py with the scoring scheme --match 0 --mismatch 1 --gap1 2 --gap2 2 to obtain the Sum-of-Pairs score.

## Related tools

- **FMAlign2** (Main driver: segments sequences via suffix-array MEMs, runs parallel subMSA, performs sequence-profile alignment and gap-based refinement to concatenate blocks) — https://github.com/malabz/FMAlign2
- **MAFFT** (Default pluggable MSA backend (-p mafft) for aligning segmented subMSA blocks in parallel) — https://mafft.cbrc.jp/alignment/software/
- **Clustal Omega** (Alternative MSA backend selectable with -p clustalo) — http://www.clustal.org/omega/
- **HAlign3** (Alternative MSA backend selectable with -p halign3 (JAR, K-ary suffix-tree based center star strategy)) — https://github.com/malabz/HAlign-3
- **HAlign4** (Alternative MSA backend selectable with -p halign4 for aligning very large numbers of sequences) — https://github.com/pinglu-zhang/HAlign-4
- **sp.py** (Evaluation script that computes the Sum-of-Pairs score of the final alignment) — https://github.com/malabz/FMAlign2

## Examples

```
FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta -p mafft -l 30 -f accurate
```

## Evaluation signals

- output.fasta exists and parses as a valid aligned FASTA file with all input sequences present and of equal (aligned) length.
- sp.py runs without error and prints a Sum-of-Pairs score using --match 0 --mismatch 1 --gap1 2 --gap2 2; higher SP score indicates better alignment quality.
- Lowering -l (e.g., below 30) should increase the number of MEMs found if segmentation was sparse — if too few MEMs are found, the minimum MEM length was too high.
- Alignment completes within acceptable time on billion-base inputs; switching -f to fast should reduce runtime at the cost of sensitivity, providing a runtime/sensitivity sanity check.
- Unmatched fragments appear incorporated into the backbone with no orphaned segments left out of the concatenated result.

## Limitations

- If too few MEMs are found, alignment quality degrades; remedy by lowering the -l minimum MEM length (e.g., below the default 30).
- MEM detection with -f accurate can be slow; -f fast runs faster but may find fewer MEMs, trading sensitivity for speed.
- If a segment remains too large for the backend, FMAlign2 must recursively apply itself with a reduced MEMs minimum length, limited to two iterations.
- Custom MSA backends must consume and emit FASTA; tools with other I/O formats cannot be integrated without a wrapper.

## Evidence

- [abstract] Sequence-profile alignment and refinement to concatenate subsets into the final result: "Furthermore, sequence-profile alignment and refinement are incorporated to concatenate subsets, yielding the final result seamlessly."
- [abstract] Vertical division enables parallel subMSA: "By using a vertical division strategy, large-scale problem is deconstructed into manageable tasks, enabling parallel execution of subMSA."
- [abstract] Partial-chain MEM segmentation: "FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains."
- [readme] Backend selection and MEM parameters: "-p <method|file>` (default: `mafft`). MSA backend — `mafft`, `clustalo`, `halign3`, or `halign4` — or a path to a custom MSA command file."
- [readme] SP scoring scheme: "python sp.py --input output.fasta --match 0 --mismatch 1 --gap1 2 --gap2 2"
- [readme] Fast mode tradeoff: "-f <mode>` (default: `accurate`). MEM filtering mode; use `fast` to speed up at the cost of sensitivity."
