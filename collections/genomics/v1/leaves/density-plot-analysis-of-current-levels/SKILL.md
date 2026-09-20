---
name: density-plot-analysis-of-current-levels
description: Use when nanopore signal-to-read or signal-to-reference alignment cannot
  precisely associate signal events with single bases and a chemistry-specific base
  shift must be derived for a pore model or move-table alignment.
when_to_use_negative:
- The data are not nanopore raw signal with an available pore model or move-table-based
  read-signal alignment (e.g. already polished second-generation reads)
- A per-read eventalign such as f5c eventalign already provides refined alignment
  boundaries and no global base shift is needed
- The signal-base alignment is produced by squigualiser realign with a valid move
  table, where no base shift adjustment is required
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: squigualiser
  role: Provides the calculate_offsets utility and --base_shift argument to derive
    and apply the k-mer-to-base shift correction
  repo: https://github.com/hiruna72/squigualiser
- name: Python 3.8
  role: Runtime environment in which squigualiser is developed and installed (pip
    or bioconda)
  repo: https://github.com/hiruna72/squigualiser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae501
    title: Interactive visualization of nanopore sequencing signal data with <i>Squigualiser</i>
  source_content_hash: sha256:7cdc56829853d0d8f6fd78fdca4c858f4ef754e171dc2ef938cfd8b7a5a1e04a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hiruna72/squigualiser
  tools:
  - Python 3.8
  - squigualiser
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# density-plot-analysis-of-current-levels

## Summary

A diagnostic procedure using squigualiser's calculate_offsets utility that derives the k-mer-to-base shift correction for nanopore signal-base alignment by binning k-mer current levels into A/C/G/T densities per base index, identifying the most significant base (MSB), and sweeping sig_move_offset values until a chemistry-specific global base shift is determined.

## When to use

Use when nanopore signal-to-read or signal-to-reference alignment cannot precisely associate signal events with single bases and a chemistry-specific base shift must be derived for a pore model or move-table alignment.

## When NOT to use

- The data are not nanopore raw signal with an available pore model or move-table-based read-signal alignment (e.g. already polished second-generation reads)
- A per-read eventalign such as f5c eventalign already provides refined alignment boundaries and no global base shift is needed
- The signal-base alignment is produced by squigualiser realign with a valid move table, where no base shift adjustment is required

## Inputs

- k-mer pore model file (e.g. pore.model, such as dna_r10.4.1_e8.2_400bps/9mer_levels_v1.txt or r9.4_450bps.nucleotide.5mer.template.model)
- read-signal alignment in PAF format with ss string (reform.paf, produced by squigualiser reform from the basecaller move table)
- raw nanopore signal file in S/BLOW5 format (reads.blow5)
- basecalled sequence file (reads.fastq)

## Outputs

- Most significant base index (base_idx) with its max_diff statistic
- Recommended sig_move_offset and recommended_kmer_length (sig_move_offset + 1)
- Chemistry-specific dataset-level base shift value (e.g. forward -6 / reverse -2 for dna_r10.4.1_e8.2_400bps 9-mer; forward -2 / reverse -3 for dna_r9.4.1_450bps 6-mer)
- Offset density plots written to out.pdf

## How to apply

Load the k-mer pore model (pore.model) and read-signal alignment (reform.paf) with signal (reads.blow5) and sequence (reads.fastq) files into squigualiser's calculate_offsets utility. For each candidate base_index within the kmer_length, bin every k-mer current level into one of four A/C/G/T bins according to the base at that position and draw four density plots per base index (Option 3 mode with --use_model). Compute the max-min difference of medians across the four base densities at each index and take the index with the maximum difference as the most significant base. Sweep sig_move_offset m from 0 to DEFAULT_KMER_LENGTH=6, building a partial k-mer model from the fastq sequence via the move table, and select the first offset at which a clear separation among the density plots appears (e.g. sig_move_offset=2 for guppy_dna_r9.4.1_450bps_fast_prom). Confirm that recommended_sig_move_offset == best_m minus most_significant_base_offset (e.g. 2 == 5 - 3), then set recommended_kmer_length = recommended_sig_move_offset + 1 and apply the resulting base shift globally via --base_shift.

## Related tools

- **squigualiser** (Provides the calculate_offsets utility and --base_shift argument to derive and apply the k-mer-to-base shift correction) — https://github.com/hiruna72/squigualiser
- **Python 3.8** (Runtime environment in which squigualiser is developed and installed (pip or bioconda)) — https://github.com/hiruna72/squigualiser

## Examples

```
squigualiser calculate_offsets --kmer_model pore.model --alignment reform.paf --signal reads.blow5 --sequence reads.fastq --use_model -o out.pdf
```

## Evaluation signals

- The derived most-significant base offset and recommended sig_move_offset reproduce the documented per-chemistry base-shift values (forward -6 / reverse -2 for dna_r10.4.1_e8.2_400bps 9-mer; forward -2 / reverse -3 for dna_r9.4.1_450bps 6-mer)
- The assumption check recommended_sig_move_offset == best_m - most_significant_base_offset holds (e.g. 2 == 5 - 3 for guppy_dna_r9.4.1_450bps_fast_prom)
- Density plots at the chosen sig_move_offset show a clear separation among the four A/C/G/T base densities
- recommended_kmer_length equals recommended_sig_move_offset + 1
- Resulting offset density plots are non-trivially rendered in out.pdf

## Limitations

- The chosen sig_move_offset is protocol/chemistry-specific (e.g. 2 for guppy_dna_r9.4.1_450bps_fast_prom) and the base shift differs between forward and reverse strands of the same chemistry
- The correction applies a single dataset-level base shift, so it cannot fix per-read or per-region alignment errors introduced by indels; refined boundaries require event-level methods like f5c eventalign
- Squigualiser has been tested with Python 3.8.0 (or higher) and Google Chrome is the recommended browser for the generated visualisations

## Evidence

- [methods] calculate_offsets is a utility to compute the most significant base: "`calculate_offsets`:<br/>		 A utility program to calculate the most significant base index given a kmer model or a read - signal alignment."
- [methods] Base shift is applied via --base_shift to plot commands: "User can shift the base sequence to the left by `n` number of bases by providing the argument `--base_shift -n` to `plot` and `plot_pileup` commands."
- [readme] squigualiser visualises raw signal-base alignment: "squigualiser is a tool to Visualise nanopore raw signal-base alignment."
