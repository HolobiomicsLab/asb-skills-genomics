---
name: kolmogorov-smirnov-two-sample-testing
description: Use when comparing raw nanopore signal level distributions (native vs
  WGA control) at reference k-mer positions to flag k-mers whose signal shifts suggest
  a modified base.
when_to_use_negative:
- No matched WGA control sample exists, since the test requires paired native/control
  signal distributions
- Genome coverage is below the minimal coverage threshold (default 40), making balanced
  sampling unreliable
- The goal is basecalling-level modification calling per read rather than motif-level
  detection from aggregated signal distributions
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: Snapper
  role: Implements the per-k-mer balanced KS-test signal comparison between native
    and WGA samples; installed as pip package snapper-ont
  repo: https://github.com/DNKonanov/Snapper
- name: Tombo
  role: Resquiggles fast5 files to align raw nanopore signal to the reference before
    signal extraction
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files to multi-fast5 format for efficient
    h5py access
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: scipy
  role: Provides scipy.stats.ks_2samp for the per-k-mer two-sample Kolmogorov–Smirnov
    test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:279afa97b3b672217d0f211fd90e4aadf89526f28c72d613793b6e688f270859
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DNKonanov/Snapper
  tools:
  - Python
  - Snapper
  - Tombo
  - ont_fast5_api
  - scipy
  - snapper-ont
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# kolmogorov-smirnov-two-sample-testing

## Summary

Two-sample Kolmogorov–Smirnov testing as used by Snapper to compare per-k-mer normalized signal distributions between native and WGA control Oxford Nanopore samples, producing a -log p-value used to select candidate methylation k-mers.

## When to use

Use when comparing raw nanopore signal level distributions (native vs WGA control) at reference k-mer positions to flag k-mers whose signal shifts suggest a modified base.

## When NOT to use

- No matched WGA control sample exists, since the test requires paired native/control signal distributions
- Genome coverage is below the minimal coverage threshold (default 40), making balanced sampling unreliable
- The goal is basecalling-level modification calling per read rather than motif-level detection from aggregated signal distributions

## Inputs

- Resquiggled multi-fast5 files for the native sample (Tombo event data)
- Resquiggled multi-fast5 files for the WGA control sample
- Reference genome in FASTA format

## Outputs

- seq_set of candidate modified k-mers written to passed_motifs_[strand]_[contig_name].fasta
- Per-k-mer two-sample KS-test statistics and -log p-values

## How to apply

After Tombo resquiggling and conversion to multi-fast5 via ont_fast5_api, read event data with h5py and key each read's normalized signal events by genome position and strand. Enumerate all odd-sized k-mers (k_size default 15, minimum 11) of the reference genome for both strands and aggregate per-k-mer signal values into a hash table. Draw equal-size (N=200) balanced signal samples from native and WGA samples for each k-mer to avoid coverage bias, then run scipy.stats.ks_2samp per k-mer. Retain candidates whose -log KS-test p-value exceeds ks_t (default 3) with confidence above min_conf (default 100).

## Related tools

- **Snapper** (Implements the per-k-mer balanced KS-test signal comparison between native and WGA samples; installed as pip package snapper-ont) — https://github.com/DNKonanov/Snapper
- **Tombo** (Resquiggles fast5 files to align raw nanopore signal to the reference before signal extraction) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files to multi-fast5 format for efficient h5py access) — https://github.com/nanoporetech/ont_fast5_api
- **scipy** (Provides scipy.stats.ks_2samp for the per-k-mer two-sample Kolmogorov–Smirnov test)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- passed_motifs_[strand]_[contig_name].fasta is produced for each strand/contig and contains only k-mers with -log KS-test p-value above ks_t (default 3)
- Per-k-mer sample sizes are equal (N=200) for both native and WGA arms, confirming balanced sampling
- Retained k-mers meet the min_conf confidence threshold (default 100)
- k-mer size is odd and >= 11 (default 15), matching the hash-table enumeration
- Known methylated motifs (e.g. validated on H.pylori J99 against PacBio data) appear among retained candidates

## Limitations

- Requires python 3.7; later versions might be incompatible because of inner biopython dependencies
- Minimal genome coverage depth (default 40) must be met or k-mers are not testable
- Tombo is deprecated by Oxford Nanopore in favor of Remora and is now unsupported

## Evidence

- [methods] KS-test per k-mer: [evidence span withheld — anchor status: not_found]
- [full_text] balanced sampling: [evidence span withheld — anchor status: not_found]
- [readme] ks_t threshold: "-ks_t KS_T            -log ks_test p-value (default 3)."
- [methods] Tombo resquiggling prerequisite: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [readme] balanced statistics rationale: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
