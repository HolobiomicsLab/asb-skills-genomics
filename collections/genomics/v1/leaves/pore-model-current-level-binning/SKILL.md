---
name: pore-model-current-level-binning
description: Use when nanopore signal-to-read or signal-to-reference alignments (e.g.
  move-table based plots from reads.blow5 + move tables) need a precise base shift,
  or when validating a k-mer pore model's per-base current-level separation.
when_to_use_negative:
- The signal alignment boundaries are already refined and validated (e.g. by f5c eventalign),
  so no base shift correction is needed.
- No k-mer pore model or move-table alignment exists for the chemistry — the binning
  requires either a pore model file or a read–signal alignment.
- You only need interactive visualisation of an already-aligned read (use squigualiser
  plot/plot_pileup directly instead of offset calculation).
edam_operation: http://edamontology.org/operation_3443
tools:
- name: squigualiser
  role: Provides the calculate_offsets utility that bins k-mer current levels per
    base index, plots offset densities, and computes the most significant base index
    and recommended sig_move_offset
  repo: https://github.com/hiruna72/squigualiser
- name: Python 3.8
  role: Runtime environment in which squigualiser (including calculate_offsets) is
    developed and installed
  repo: https://github.com/hiruna72/squigualiser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae501
    title: Interactive visualization of nanopore sequencing signal data with <i>Squigualiser</i>
  source_content_hash: sha256:fe382bbe27daf459142e9bb8e5c593017e288d3e95ed4b974acd73c2b88c4b95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hiruna72/squigualiser
  tools:
  - Python 3.8
  - squigualiser
  edam_operation: http://edamontology.org/operation_3443
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# pore-model-current-level-binning

## Summary

Derives a chemistry-specific k-mer-to-base shift correction by binning k-mer current levels into A/C/G/T density plots per candidate base index, identifying the most significant base, and sweeping sig_move_offset to find where base-specific current-level separation emerges. This corrects the systematic offset between signal events and single bases that existing nanopore signal alignment methods leave uncorrected.

## When to use

Use when nanopore signal-to-read or signal-to-reference alignments (e.g. move-table based plots from reads.blow5 + move tables) need a precise base shift, or when validating a k-mer pore model's per-base current-level separation.

## When NOT to use

- The signal alignment boundaries are already refined and validated (e.g. by f5c eventalign), so no base shift correction is needed.
- No k-mer pore model or move-table alignment exists for the chemistry — the binning requires either a pore model file or a read–signal alignment.
- You only need interactive visualisation of an already-aligned read (use squigualiser plot/plot_pileup directly instead of offset calculation).

## Inputs

- k-mer pore model file (pore.model, e.g. r9.4_450bps 5-mer or dna_r10.4.1_e8.2_400bps 9-mer levels)
- read–signal alignment in reform.paf format (from move table to ss string conversion)
- raw nanopore signal file (reads.blow5)
- basecalled sequence file (reads.fastq)

## Outputs

- most significant base index (base_idx) and its offset (most_significant_base_offset)
- recommended sig_move_offset and recommended_kmer_length (sig_move_offset + 1)
- dataset-level base-shift value applied via --base_shift
- offset density plots written to out.pdf

## How to apply

Load the k-mer pore model file (pore.model) and the read–signal alignment (reform.paf) together with the raw signal (reads.blow5) and sequence (reads.fastq) files into squigualiser's calculate_offsets utility. For each candidate base_index within the kmer_length, bin every k-mer current level into one of four A/C/G/T bins according to the base at that position and draw four density plots per base index (Option 3 mode with --use_model). Compute the difference between the maximum and minimum medians across the four base densities at each base index; the base index with the maximum difference (max_diff) is the most significant base. Then sweep sig_move_offset m from 0 to DEFAULT_KMER_LENGTH = 6, building a partial k-mer model from the FASTQ sequence with current levels pointed to by the move table, and select the first offset at which density plots clearly separate (e.g. sig_move_offset = 2 for guppy_dna_r9.4.1_450bps_fast_prom). Confirm recommended_sig_move_offset == best_m - most_significant_base_offset (e.g. 2 == 5 - 3), set recommended_kmer_length = recommended_sig_move_offset + 1, and validate the resulting base shift against the documented per-chemistry table before applying it globally with --base_shift.

## Related tools

- **squigualiser** (Provides the calculate_offsets utility that bins k-mer current levels per base index, plots offset densities, and computes the most significant base index and recommended sig_move_offset) — https://github.com/hiruna72/squigualiser
- **Python 3.8** (Runtime environment in which squigualiser (including calculate_offsets) is developed and installed) — https://github.com/hiruna72/squigualiser

## Examples

```
squigualiser calculate_offsets --kmer_model pore.model --alignment_reform reform.paf --signal_file reads.blow5 --sequence_file reads.fastq --mode 3 --use_model --output_offset_pdf out.pdf
```

## Evaluation signals

- The assumption check holds: recommended_sig_move_offset == best_m - most_significant_base_offset (e.g. 2 == 5 - 3 for guppy_dna_r9.4.1_450bps_fast_prom).
- The derived base shift reproduces the documented per-chemistry values: dna_r10.4.1_e8.2_400bps 9-mer forward -6 / reverse -2; dna_r9.4.1_450bps 6-mer forward -2 / reverse -3.
- At the chosen sig_move_offset (e.g. 2), the four A/C/G/T density plots per base index show clear separation, and the base index with maximum median spread (max_diff) is stable.
- recommended_kmer_length equals recommended_sig_move_offset + 1, consistent with the k-mer length of the loaded pore model.
- Applying the shift with --base_shift to plot/plot_pileup output visually aligns signal events with the correct bases.

## Limitations

- The offset is protocol- and chemistry-specific: a shift derived for one model (e.g. dna_r10.4.1_e8.2_400bps) cannot be reused for another (e.g. dna_r9.4.1_450bps), and forward vs reverse strands have different shifts.
- Modified bases distort the model: current levels of k-mers containing methylated U drop below the corresponding unmethylated k-mer levels, more pronounced when several methylated Us occur together, which can confound level binning.
- Among signal alignment methods, nanopolish signal projection introduces the highest signal shape distortion, so shift validation is best performed against cleaner alignments such as f5c eventalign.

## Evidence

- [methods] calculate_offsets is a utility for base index significance: "`calculate_offsets`:
		 A utility program to calculate the most significant base index given a kmer model or a read - signal alignment."
- [methods] base shift argument for plotting: "User can shift the base sequence to the left by `n` number of bases by providing the argument `--base_shift -n` to `plot` and `plot_pileup` commands."
- [readme] tool purpose statement: "squigualiser is a tool to Visualise nanopore raw signal-base alignment."
- [abstract] toolkit scope: "Squigualiser (Squiggle visualiser) is a toolkit for intuitive, interactive visualization of sequence-aligned signal data"
