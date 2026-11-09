---
name: methylation-motif-benchmarking-against-pacbio-reference
description: Use when evaluating how many methylation motifs an ONT-native pipeline
  recovers de novo relative to a PacBio-characterized methylome, e.g. H. pylori J99
  ONT native/WGA data.
when_to_use_negative:
- Input data is not ONT fast5 with a matched WGA control — the sample-compare design
  requires native and WGA multi-fast5 directories plus a reference FASTA
- The goal is methylome discovery on a strain with no prior methylation characterization
  rather than benchmarking detection sensitivity against a known reference
- You need single-base modification calling rather than position-specific methylation
  motif enrichment
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: Tombo
  role: Resquiggle ONT fast5 reads against the reference and run model_sample_compare
    between native and WGA samples for per-base modification statistics
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Convert resquiggled fast5 files to the multi-fast5 format required by the
    downstream motif tools
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: MEME
  role: De novo motif discovery on extracted 12/18/22-nt modified-site context FASTA
    sets using Tombo-documented parameters (baseline comparator)
- name: Snapper
  role: High-sensitivity greedy position-specific motif enrichment run directly on
    sample/control multi-fast5 dirs and the reference FASTA (benchmark comparator)
  repo: DNKonanov/Snapper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:0d626ba491f7c37737b0dfd7d623b4818ed7d4bf0839350dae9c672433c3df26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - MEME
  - Snapper
  - Tombo
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# methylation-motif-benchmarking-against-pacbio-reference

## Summary

Benchmark ONT-based methylation motif detection pipelines (Tombo+MEME baseline vs Snapper) against a PacBio-validated methylome reference, using H. pylori J99 as the calibration strain to quantify de novo motif-enrichment sensitivity.

## When to use

Use when evaluating how many methylation motifs an ONT-native pipeline recovers de novo relative to a PacBio-characterized methylome, e.g. H. pylori J99 ONT native/WGA data.

## When NOT to use

- Input data is not ONT fast5 with a matched WGA control — the sample-compare design requires native and WGA multi-fast5 directories plus a reference FASTA
- The goal is methylome discovery on a strain with no prior methylation characterization rather than benchmarking detection sensitivity against a known reference
- You need single-base modification calling rather than position-specific methylation motif enrichment

## Inputs

- Native ONT fast5 reads (H. pylori J99 sample)
- WGA control ONT fast5 reads (H. pylori J99 WGA control)
- J99 reference genome in FASTA format
- PacBio-derived J99 methylome/motif reference set for benchmarking

## Outputs

- Per-base modification statistics from Tombo model_sample_compare
- FASTA context sets of 12/18/22-nt sequences per strand/contig
- Filtered MEME motif list written to meme_motifs.txt (five motifs for the J99 baseline)
- Snapper motif calls from the sample/control/reference invocation

## How to apply

Resquiggle the J99 native ONT fast5 reads against the J99 reference genome with Tombo, then convert both the native sample and the J99 WGA control fast5 directories to multi-fast5 format using ont_fast5_api. For the Tombo+MEME baseline, run Tombo's model_sample_compare between native and WGA samples to get per-base modification statistics, extract 12-, 18-, and 22-nt read-sequence contexts centered on significantly modified sites per strand/contig as FASTA, run MEME with Tombo-documented parameters on each context set, and filter MEME motifs to genome-highly-represented ones. For comparison, run snapper directly on the sample/control multi-fast5 directories with the reference FASTA (k_size default 15, coverage default 40, ks_t default 3). Judge the baseline by its recovered motif count against the PacBio-derived J99 motif set: the baseline is low-sensitivity, inferring only five genome-highly-represented motifs on J99 (Table 1, ONT+Tombo column) instead of the full J99 methylation motif complement, whereas Snapper claims higher enrichment sensitivity than MEME coupled with Tombo or Nanodisco.

## Related tools

- **Tombo** (Resquiggle ONT fast5 reads against the reference and run model_sample_compare between native and WGA samples for per-base modification statistics) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Convert resquiggled fast5 files to the multi-fast5 format required by the downstream motif tools) — https://github.com/nanoporetech/ont_fast5_api
- **MEME** (De novo motif discovery on extracted 12/18/22-nt modified-site context FASTA sets using Tombo-documented parameters (baseline comparator))
- **Snapper** (High-sensitivity greedy position-specific motif enrichment run directly on sample/control multi-fast5 dirs and the reference FASTA (benchmark comparator)) — DNKonanov/Snapper

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- Baseline Tombo+MEME on J99 should recover five genome-highly-represented motifs (Table 1, ONT+MEME column), demonstrating known low sensitivity vs the PacBio J99 motif set
- Snapper motif calls should exceed MEME-coupled-with-Tombo/Nanodisco sensitivity on J99 and the four external datasets
- Resquiggled fast5 files must convert cleanly to multi-fast5 format with ont_fast5_api for both native and WGA directories
- Output motif list meme_motifs.txt should contain only motifs classified as genome-highly-represented after MEME filtering
- Snapper run should meet the minimum genome coverage depth of 40 (default) for called positions

## Limitations

- The Tombo+MEME baseline has low sensitivity: on H. pylori J99 it infers only five motifs rather than the full J99 methylation motif set
- MEME does not extract all H. pylori methylation sites de novo even with Nanodisco's iterative approach; no existing algorithm is both sensitive and deterministic
- Snapper requires Python 3.7; later versions may be incompatible because of inner biopython dependencies
- Tombo is deprecated/unsupported in favor of Remora by Oxford Nanopore

## Evidence

- [results] The Tombo+MEME baseline showed rather low sensitivity, inferring only five motifs highly represented in the genome for H. pylori J99: [evidence span withheld — anchor status: not_found]
- [abstract] Snapper has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco: "Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments"
- [methods] fast5 files should be resquiggled using Tombo software: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [methods] After resquiggling, fast5 files should be converted to the multi-fast5 format using ont_fast5_api.: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [abstract] MEME does not extract all Helicobacter pylori methylation sites de novo even with Nanodisco's iterative approach: "the MEME suit does not extract all Helicobacter pylori methylation sites de novo even using the iterative approach implemented in the most up-to-date methylation analysis tool Nanodisco"
