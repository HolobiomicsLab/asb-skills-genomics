---
name: wga-control-differential-signal-analysis
description: Use when you have paired native and WGA whole-genome ONT read sets (FAST5/multi-fast5)
  plus a reference FASTA and want de novo methylation motif detection.
when_to_use_negative:
- No WGA (unmethylated) control reads are available — the differential sample-vs-control
  contrast requires a matched control FAST5 directory
- Input data are already basecalled and modified-base-annotated (e.g. Remora/Dorado
  modBAM output) rather than raw FAST5 signal, since the workflow is anchored on Tombo
  resquiggling
- The goal is per-read modified-base calling at specific sites rather than de novo
  consensus motif discovery
edam_operation: http://edamontology.org/operation_0427
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: Snapper
  role: Performs the greedy position-specific motif enrichment comparing native sample
    against WGA control signal; installed as the pip package snapper-ont
  repo: https://github.com/DNKonanov/Snapper
- name: Tombo
  role: Resquiggles raw FAST5 files against the reference genome before Snapper analysis
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled FAST5 files into the multi-fast5 format required by Snapper
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: snapper-ont
  role: Pip distribution of Snapper used for installation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:60692aaf609cf95a9a16932748d7ee77d54e6d58c4012ce861c5e38284e6d908
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DNKonanov/Snapper
  tools:
  - Python
  - Snapper
  - Tombo
  - ont_fast5_api
  - pip
  - snapper-ont
  edam_operation: http://edamontology.org/operation_0427
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# wga-control-differential-signal-analysis

## Summary

Detect methylation motifs by contrasting Oxford Nanopore signal distributions of a native DNA sample against a matched WGA (whole-genome amplified, unmethylated) control, using Snapper's greedy position-specific motif enrichment.

## When to use

Use when you have paired native and WGA whole-genome ONT read sets (FAST5/multi-fast5) plus a reference FASTA and want de novo methylation motif detection.

## When NOT to use

- No WGA (unmethylated) control reads are available — the differential sample-vs-control contrast requires a matched control FAST5 directory
- Input data are already basecalled and modified-base-annotated (e.g. Remora/Dorado modBAM output) rather than raw FAST5 signal, since the workflow is anchored on Tombo resquiggling
- The goal is per-read modified-base calling at specific sites rather than de novo consensus motif discovery

## Inputs

- Native (methylated) whole-genome ONT FAST5 read files for the sample
- WGA (whole-genome amplified) whole-genome ONT FAST5 read files as unmethylated control
- Reference genome in FASTA format
- Tombo-resquiggled, multi-fast5-converted read directories for both sample and control

## Outputs

- final_motifs_[strand]_[contig_name].fasta files per contig and strand
- Per-strain methylation motif tables for comparison against REBASE references

## How to apply

Resquiggle the raw FAST5 files against the reference genome using Tombo, then convert them to multi-fast5 format with ont_fast5_api. Run snapper, supplying the native multi-fast5 directory as -sample_fast5dir and the WGA multi-fast5 directory as -control_fast5dir, together with the reference FASTA. Use default parameters (ks_t = 3, coverage = 40, threads = 8, k_size = 15, long_k_size = 29, min_conf = 100) unless coverage or motif length constraints require adjustment. Collect the final_motifs_[strand]_[contig_name].fasta outputs into per-strain motif tables and compare detected motifs against REBASE reference methyltransferase specificities to validate the run.

## Related tools

- **Snapper** (Performs the greedy position-specific motif enrichment comparing native sample against WGA control signal; installed as the pip package snapper-ont) — https://github.com/DNKonanov/Snapper
- **Tombo** (Resquiggles raw FAST5 files against the reference genome before Snapper analysis) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled FAST5 files into the multi-fast5 format required by Snapper) — https://github.com/nanoporetech/ont_fast5_api
- **snapper-ont** (Pip distribution of Snapper used for installation)

## Examples

```
snapper -sample_fast5dir sample_multi/ -control_fast5dir wga_multi/ -reference genome.fasta
```

## Evaluation signals

- Per-strain motif tables recover all expected REBASE reference methylation motifs
- The only deviation from REBASE is the degenerate motif CACNNNNNRTAAA resolved to the concrete instance CACNNNNNATAAA
- Novel motifs CCAG/TGGCCA in T. phaeum DSM 12270 and WGGCCW in C. perfringens ATCC 13124 are detected in SRP219538 validation runs
- Every run completes for both sample and control FAST5 directories with sufficient genome coverage (default minimum coverage depth 40)

## Limitations

- Requires Python 3.7; later versions might be incompatible because of inner biopython dependencies
- k_size must be odd and not less than 11
- No methylation motif detection algorithms previously combined sufficient sensitivity with deterministic results, motivating Snapper's greedy approach
- Tombo is deprecated by Oxford Nanopore in favor of Remora, which may affect long-term maintainability of the resquiggling step

## Evidence

- [readme] Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified.: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
- [methods] Resquiggle fast5 files using Tombo: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [methods] Run snapper with sample and control fast5 dirs: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [abstract] Snapper has higher enrichment sensitivity than MEME with Tombo or Nanodisco: "Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments"
- [abstract] Greedy motif selection algorithm: "We present Snapper, a new highly sensitive approach, to extract methylation motif sequences based on a greedy motif selection algorithm."
