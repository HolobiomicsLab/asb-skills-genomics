---
name: genome-size-based-preset-selection
description: Use when mapping raw nanopore signals (FAST5/POD5/SLOW5/BLOW5) to a reference
  genome with RawHash2/RawHash and you must choose the preset matching the reference
  genome's size and type (viral vs. human-sized).
when_to_use_negative:
- The task is all-vs-all overlap detection between raw signals without a reference
  genome — use Rawsamble's overlapping mode instead of a genome mapping preset.
- The data is legacy r9.4.1 being mapped with UNCALLED, which only supports that chemistry
  and has its own presets; RawHash2 R10-specific flags (--r10) do not apply.
- The input is already basecalled FASTQ/FASTA and you need sequence-level alignment
  — use minimap2 with its own presets (e.g., -x map-ont) rather than raw-signal mapping
  presets.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash2
  role: Raw-signal mapper whose presets (-x viral, -x sensitive, -x fast, --r10) are
    selected based on genome size/chemistry; builds ref.ind and emits modified-PAF
    mappings
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Baseline raw-signal mapper (release v1.0) run with the same references and
    datasets when comparing preset-driven accuracy
  repo: https://github.com/CMU-SAFARI/RawHash
- name: minimap2
  role: Basecalled-read mapper used to generate ground-truth PAF alignments against
    the same reference genome
  repo: https://github.com/lh3/minimap2
- name: UNCALLED
  role: Source of the pafstats utility used to score tool PAFs (precision, recall,
    F1) and of the R10.4.1 k-mer model used with --r10
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Provides the R9.4 segmentation parameters used by RawHash2's signal normalization
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:69043825f00d935fb172a78c03d8c8b07e395ade422fd7c9379ee52373cc5262
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# genome-size-based-preset-selection

## Summary

Select RawHash2's genome-appropriate mapping preset (e.g., -x viral for small viral genomes, -x sensitive for small eukaryotic genomes, -x fast for large genomes such as human) so that hash-based raw-signal mapping remains accurate and fast for the genome at hand.

## When to use

Use when mapping raw nanopore signals (FAST5/POD5/SLOW5/BLOW5) to a reference genome with RawHash2/RawHash and you must choose the preset matching the reference genome's size and type (viral vs. human-sized).

## When NOT to use

- The task is all-vs-all overlap detection between raw signals without a reference genome — use Rawsamble's overlapping mode instead of a genome mapping preset.
- The data is legacy r9.4.1 being mapped with UNCALLED, which only supports that chemistry and has its own presets; RawHash2 R10-specific flags (--r10) do not apply.
- The input is already basecalled FASTQ/FASTA and you need sequence-level alignment — use minimap2 with its own presets (e.g., -x map-ont) rather than raw-signal mapping presets.

## Inputs

- Reference genome in FASTA format (e.g., SARS-CoV-2, green algae, human GRCh38, E. coli)
- Raw nanopore signal reads in FAST5, POD5, SLOW5, or BLOW5 format
- K-mer pore model files (e.g., legacy R9.4 450bps 6-mer template model under extern/kmer_models/legacy; UNCALLED4 R10.4.1 model)

## Outputs

- Raw-signal index file (ref.ind) built from the reference genome
- Mapping alignments in modified PAF format, per dataset, reflecting the chosen preset

## How to apply

Identify the reference genome and the sequencing chemistry of the test dataset, then pick the RawHash2 preset accordingly: use -x viral for small viral genomes (e.g., SARS-CoV-2), -x sensitive for small eukaryotic genomes (e.g., green algae), and -x fast for large genomes (e.g., human NA12878). For newer chemistry, switch the k-mer model instead of the preset: use -x sensitive --r10 with the UNCALLED4 R10.4.1 k-mer model for R10.4 data, since RawHash2 added R10.4 support. Build the raw-signal index (ref.ind) with the legacy R9.4 450bps 6-mer template model from extern/kmer_models/legacy using ~32 threads before mapping. The preset choice governs seeding/sensitivity trade-offs inherent to the hash-based indexing and mapping steps; the outputs are modified-PAF mappings that can be scored with UNCALLED pafstats against minimap2-based ground truth.

## Related tools

- **RawHash2** (Raw-signal mapper whose presets (-x viral, -x sensitive, -x fast, --r10) are selected based on genome size/chemistry; builds ref.ind and emits modified-PAF mappings) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Baseline raw-signal mapper (release v1.0) run with the same references and datasets when comparing preset-driven accuracy) — https://github.com/CMU-SAFARI/RawHash
- **minimap2** (Basecalled-read mapper used to generate ground-truth PAF alignments against the same reference genome) — https://github.com/lh3/minimap2
- **UNCALLED** (Source of the pafstats utility used to score tool PAFs (precision, recall, F1) and of the R10.4.1 k-mer model used with --r10) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Provides the R9.4 segmentation parameters used by RawHash2's signal normalization) — https://github.com/haowenz/sigmap

## Examples

```
rawhash2 -i -r ref.fasta -o ref.ind -t 32 && rawhash2 ref.ind sars_cov2_fast5/ -x viral -o mappings.paf
```

## Evaluation signals

- Per-dataset precision, recall, and F1 are computed for every dataset when tool PAFs are scored against minimap2 ground truth with UNCALLED pafstats.
- RawHash2's F1 improvement over RawHash averages 10.57% and reaches up to 20.25%; reproduced deltas should fall within reasonable agreement of these values.
- The index file ref.ind is built successfully with the genome-appropriate k-mer model (legacy R9.4 450bps 6-mer template for R9.4 data; UNCALLED4 model for R10.4 data).
- Each dataset's mapping run uses the preset matching its genome: -x viral (SARS-CoV-2), -x sensitive (green algae, E. coli R10.4 with --r10), -x fast (human NA12878).

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data; for r10.4.1 data alternative tools or models are needed (RawHash2 addresses this with --r10 and the UNCALLED4 model).
- RawHash2's minimizer seeding mode is around 3X faster but slightly less accurate than the non-minimizer mode, which affects the accuracy-throughput trade-off per preset.
- FAST5 reads compressed with VBZ require the ONT vbz_compression HDF5 plugin to be installed before signal extraction.

## Evidence

- [abstract] abstract: "RawHash2 provides better F1 accuracy (on average by 10.57% and up to 20.25%)"
- [methods] methods: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [readme] readme: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [intro] intro: "support for the R10.4 flow cell version and POD5 and SLOW5 file formats"
