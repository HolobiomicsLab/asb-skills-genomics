---
name: signal-base-alignment-visualization
description: Use when nanopore FAST5/POD5 raw signal data (DNA or RNA) must be aligned
  to basecalls or a reference for visualisation, especially when basecalled reads
  contain methylated bases (e.g. methylatedU basecalled as C) that break pore-model-based
  event alignment.
when_to_use_negative:
- When a fast summary statistic rather than interactive per-read signal visualisation
  is needed — use squigualiser metric or external evaluation instead of plotting.
- When the basecaller BAM lacks move tags (basecalled without --moves_out), since
  reform/realign depend on the move table.
- When working with non-ONT signal data, as squigualiser currently supports only DNA
  and RNA sequencing data from Oxford Nanopore Technologies instruments.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Squigualiser
  role: 'Core toolkit: reform (move table to ss string), realign (signal-to-reference
    via cigar string and move table), and plot/plot_pileup/plot_tracks visualisation'
  repo: https://github.com/hiruna72/squigualiser
- name: guppy (guppy_basecaller)
  role: Basecalling with --moves_out using rna_r9.4.1_70bps_hac_prom.cfg to produce
    the move-table BAM
- name: minimap2
  role: Read alignment to the reference (e.g. -k 10 -s 50 for the GFP mRNA direct
    RNA data)
- name: f5c
  role: Alternative eventalign-based signal-to-reference alignment for comparison
    against move-table realign
- name: nanopolish
  role: Pore model training (round-1 trained model) to improve eventalign-based signal
    alignment for comparison
  repo: https://github.com/jts/nanopolish
- name: blue-crab
  role: POD5 to SLOW5/BLOW5 conversion when signal input is in POD5 format
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae501
    title: Interactive visualization of nanopore sequencing signal data with <i>Squigualiser</i>
  source_content_hash: sha256:cbb01133e44e4be3bf67fdbca6160e05cbbaf467fd4ef0a39bf784c05445522b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hiruna72/squigualiser
  tools:
  - Squigualiser
  - blue-crab
  - f5c
  - guppy (guppy_basecaller)
  - minimap2
  - nanopolish
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# signal-base-alignment-visualization

## Summary

Visualise nanopore raw signal-base alignments (signal-to-read and signal-to-reference) at single-base resolution using Squigualiser, whose realign method derives signal-to-reference alignments from the basecaller's move table and is robust to methylated Us in direct RNA-seq data.

## When to use

Use when nanopore FAST5/POD5 raw signal data (DNA or RNA) must be aligned to basecalls or a reference for visualisation, especially when basecalled reads contain methylated bases (e.g. methylatedU basecalled as C) that break pore-model-based event alignment.

## When NOT to use

- When a fast summary statistic rather than interactive per-read signal visualisation is needed — use squigualiser metric or external evaluation instead of plotting.
- When the basecaller BAM lacks move tags (basecalled without --moves_out), since reform/realign depend on the move table.
- When working with non-ONT signal data, as squigualiser currently supports only DNA and RNA sequencing data from Oxford Nanopore Technologies instruments.

## Inputs

- ONT raw signal data (FAST5 or POD5; POD5 convertible to BLOW5 with blue-crab)
- Basecaller move-table BAM from guppy v6.3.7 (--moves_out), merged with samtools
- Reference FASTA (e.g. synthetic GFP mRNA sequence)
- Read alignment BAM from minimap2
- Optional kmer model files (e.g. from nanoporetech/kmer_models) for eventalign comparisons

## Outputs

- reform.paf — signal-to-read alignment in PAF format derived from the move table
- realign_output.paf — signal-to-reference alignment in PAF format
- Interactive HTML signal browser plots (plot/plot_pileup/plot_tracks) at single-base resolution with signal pileup and parallel tracks

## How to apply

Basecall the ONT FAST5 data with guppy_basecaller v6.3.7 using rna_r9.4.1_70bps_hac_prom.cfg with --moves_out, then merge passed BAMs with samtools merge into a single move-table BAM. Align reads to the reference with minimap2 (for the methylatedU GFP mRNA data, tweaked parameters -k 10 -s 50 raised mapping to 97%). Run 'squigualiser reform' with --rna and the rna_r9.4.1_70bps 5_mer profile values (kmer_length 5, sig_move_offset) to convert the basecaller's move table to an ss string and write the read-signal alignment to reform.paf; then run 'squigualiser realign' with --rna, the minimap2 BAM and reform.paf to produce the signal-to-reference alignment realign_output.paf. Rationale: the move-table-based realign is robust to methylated Us because signals align via the move table regardless of U-to-C basecalls, unlike f5c eventalign/nanopolish approaches that depend on the pore model (round-1 trained nanopolish models improve over the original pore model). Finally plot with 'squigualiser plot' (or plot_pileup/plot_tracks) over the region of interest and inspect the interactive HTML output in Google Chrome.

## Related tools

- **Squigualiser** (Core toolkit: reform (move table to ss string), realign (signal-to-reference via cigar string and move table), and plot/plot_pileup/plot_tracks visualisation) — https://github.com/hiruna72/squigualiser
- **guppy (guppy_basecaller)** (Basecalling with --moves_out using rna_r9.4.1_70bps_hac_prom.cfg to produce the move-table BAM)
- **minimap2** (Read alignment to the reference (e.g. -k 10 -s 50 for the GFP mRNA direct RNA data))
- **f5c** (Alternative eventalign-based signal-to-reference alignment for comparison against move-table realign)
- **nanopolish** (Pore model training (round-1 trained model) to improve eventalign-based signal alignment for comparison) — https://github.com/jts/nanopolish
- **blue-crab** (POD5 to SLOW5/BLOW5 conversion when signal input is in POD5 format)

## Examples

```
squigualiser realign --rna -f ref.fasta -s reads.blow5 -a reads.bam -r reform.paf -o realign_output.paf; squigualiser plot -f ref.fasta -s reads.blow5 -a realign_output.paf --region gfp:1-500 -o dir_out
```

## Evaluation signals

- reform.paf and realign_output.paf exist and are in valid PAF format
- Plots reproduce the reported behaviour that realign remains robust to methylated U: signals align via the move table regardless of U-to-C basecalls
- f5c eventalign using the round-1 nanopolish-trained model yields visibly better signal alignment than the original pore model (as in the paper's Fig. 3 and Fig. 6 comparisons)
- Read mapping to the synthetic GFP mRNA reference reaches ~97% with minimap2 -k 10 -s 50
- Interactive HTML output opens in Google Chrome and supports navigation across genome/transcriptome regions with signal pileup and parallel tracks

## Limitations

- Basecaller basecalled almost all methylated Us as C; methylatedU datasets yield only 62% pass reads vs 92% for unmethylated, reflecting very different signal nature
- Current level of kmers with methylated U drops below the corresponding unmethylated kmer level, more pronounced when several methylated Us occur together — affecting pore-model-based alignments
- Signal alignment accuracy from nanopolish pore model training did not improve after the first two rounds (round-1 model is the useful output)
- Among alignment methods, f5c eventalign has the best refined boundaries; nanopolish signal projection introduces the highest signal shape distortion
- Google Chrome is the recommended web browser for visualising the plots

## Evidence

- [methods] Squigualiser's [realign method](realign.md) is robust to the methylated Us as it uses the basecaller's move table: "Squigualiser's [realign method](realign.md) is robust to the methylated Us as it uses the basecaller's move table"
- [methods] realign subtool description: "`realign`:<br/>         Realign signal to reference using [cigar string](https://samtools.github.io/hts-specs/SAMv1.pdf) and the move table."
- [methods] reform subtool description: "`reform`:<br/>         Convert basecaller's move table to [ss string](https://hasindu2008.github.io/f5c/docs/output#ss-tag) format."
- [readme] README quickstart pileup command: [evidence span withheld — anchor status: external_doc]
- [intro] squigualiser purpose: "squigualiser is a tool to Visualise nanopore raw signal-base alignment."
- [methods] basecalling pass-read rates: "bascalling reported 92% and 62% pass reads percentage for the unmethylated and methylatedU datasets respectively."
