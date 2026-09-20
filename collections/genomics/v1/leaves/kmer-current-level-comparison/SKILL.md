---
name: kmer-current-level-comparison
description: Use when two homologous direct RNA nanopore datasets (e.g. methylatedU
  SRR22888950 vs unmethylated SRR22888949 GFP mRNA) have been realigned into signal
  pileups and you need to quantify per-k-mer current-level shifts attributable to
  base modifications.
when_to_use_negative:
- Input reads have not been signal-realigned to a reference (no move-table-based realign),
  since raw basecalled sequences cannot be compared at k-mer current level; note the
  basecaller calls methylated Us as C, so sequence-level comparison alone is misleading
- The comparison involves DNA r10.4.1 data or chemistries other than rna_r9.4.1_70bps,
  for which the base shift and pileup profile used here do not apply
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Squigualiser
  role: realign and plot_pileup/plot_tracks to generate base-aligned signal pileups
    for both datasets
  repo: https://github.com/hiruna72/squigualiser
- name: guppy (guppy_basecaller)
  role: basecalling with rna_r9.4.1_70bps_hac_prom.cfg; the move table enables methylated-U-robust
    realignment
- name: minimap2
  role: read alignment of basecalled reads to the reference before signal realignment
- name: Python 3.8
  role: compute per-k-mer median pA levels, aggregate differences, and plot the comparison
    figure
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae501
    title: Interactive visualization of nanopore sequencing signal data with <i>Squigualiser</i>
  source_content_hash: sha256:ba87c3941ba865993f3fa3c429fca6db719577e77efa3703525887b8a1c0b2af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hiruna72/squigualiser
  tools:
  - Python 3.8
  - Squigualiser
  - guppy (guppy_basecaller)
  - minimap2
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# kmer-current-level-comparison

## Summary

Compares per-k-mer median current levels (pA) between squigualiser-generated reference-signal pileups of methylated versus unmethylated direct RNA-seq datasets to detect the signal-level signature of methylated U, which drops below the corresponding unmethylated k-mer level and is amplified when multiple methylated Us occur together.

## When to use

Use when two homologous direct RNA nanopore datasets (e.g. methylatedU SRR22888950 vs unmethylated SRR22888949 GFP mRNA) have been realigned into signal pileups and you need to quantify per-k-mer current-level shifts attributable to base modifications.

## When NOT to use

- Input reads have not been signal-realigned to a reference (no move-table-based realign), since raw basecalled sequences cannot be compared at k-mer current level; note the basecaller calls methylated Us as C, so sequence-level comparison alone is misleading
- The comparison involves DNA r10.4.1 data or chemistries other than rna_r9.4.1_70bps, for which the base shift and pileup profile used here do not apply

## Inputs

- squigualiser-generated reference-signal pileup for unmethylated GFP mRNA direct RNA-seq (SRR22888949)
- squigualiser-generated reference-signal pileup for methylatedU GFP mRNA direct RNA-seq (SRR22888950)
- Reference transcriptome/k-mer definitions containing U positions
- rna_r9.4.1_70bps squigualiser profile with base shift -3

## Outputs

- Per-k-mer median current-level (pA) values from both pileups with per-k-mer differences
- Aggregated current-level differences grouped by number of methylated Us per k-mer
- Figure comparing per-k-mer current levels across tracks (e.g. current_level_differences.png)

## How to apply

Generate reference-signal pileups for both datasets with squigualiser realign followed by plot_pileup/plot_tracks using the rna_r9.4.1_70bps profile with a base shift of -3, since squigualiser's realign method is robust to methylated Us because it uses the basecaller's move table. Load the pileup tracks over homologous transcriptome regions and, for each reference k-mer containing U, compute the median picoampere signal level from the unmethylated pileup (SRR22888949) and the methylatedU pileup (SRR22888950), recording the per-k-mer difference with Python 3.8. Aggregate the differences by the number of methylated Us per k-mer to test whether k-mers with several methylated Us show a larger drop. Plot the per-k-mer current-level comparisons across both tracks and save the figure (e.g. current_level_differences.png).

## Related tools

- **Squigualiser** (realign and plot_pileup/plot_tracks to generate base-aligned signal pileups for both datasets) — https://github.com/hiruna72/squigualiser
- **guppy (guppy_basecaller)** (basecalling with rna_r9.4.1_70bps_hac_prom.cfg; the move table enables methylated-U-robust realignment)
- **minimap2** (read alignment of basecalled reads to the reference before signal realignment)
- **Python 3.8** (compute per-k-mer median pA levels, aggregate differences, and plot the comparison figure)

## Examples

```
squigualiser plot_pileup --signal_file reads.blow5 --strand_list pass_bam.bam --reference transcript.fa --profile rna_r9.4.1_70bps --base_shift -3 --tag_name methylatedU --output_dir pileup_m6A
```

## Evaluation signals

- MethylatedU track shows a consistent current-level reduction at U-containing k-mers relative to the unmethylated track
- The drop in current level is amplified at k-mers containing multiple methylated Us
- Per-k-mer differences reproduce the paper's reported observation that 'the current level of a kmers with methylated U drops below the current level of the corresponding unmethylated kmer'
- Pileups were generated with the rna_r9.4.1_70bps profile and base shift -3, confirmed by inspecting the plot_pileup/plot_tracks output

## Limitations

- The basecaller basecalled almost all methylated Us as C, so k-mer identity in basecalled sequence space does not reflect the modification; comparison must be done on reference-anchored signal, not basecalls
- Synthetically methylated U has very different signal nature: basecalling yielded only 62% pass reads for the methylatedU dataset versus 92% for the unmethylated dataset, so coverage differs between tracks

## Evidence

- [methods] the current level of a kmers with methylated U drops below the current level of the corresponding unmethylated kmer.: "the current level of a kmers with methylated U drops below the current level of the corresponding unmethylated kmer."
- [methods] Squigualiser's realign method is robust to the methylated Us as it uses the basecaller's move table: "Squigualiser's [realign method](realign.md) is robust to the methylated Us as it uses the basecaller's move table"
- [methods] bascalling reported 92% and 62% pass reads percentage for the unmethylated and methylatedU datasets respectively.: "bascalling reported 92% and 62% pass reads percentage for the unmethylated and methylatedU datasets respectively."
- [methods] User can shift the base sequence to the left by `n` number of bases by providing the argument `--base_shift -n` to `plot` and `plot_pileup` commands.: "User can shift the base sequence to the left by `n` number of bases by providing the argument `--base_shift -n` to `plot` and `plot_pileup` commands."
- [readme] squigualiser is a tool to Visualise nanopore raw signal-base alignment.: "squigualiser is a tool to Visualise nanopore raw signal-base alignment."
