---
name: balanced-sampling-signal-distributions
description: Use when comparing resquiggled ONT signal events between a native sample
  and a WGA control to rank k-mers by signal-level distribution shift and emit candidate
  methylation motifs.
when_to_use_negative:
- No matched WGA control multi-fast5 dataset is available, since the KS test requires
  paired native/control signal distributions
- Genome coverage depth is below the minimal coverage threshold (default 40), making
  balanced N=200 per-k-mer sampling unreliable
- The goal is single-base modification calling rather than de novo motif/k-mer enrichment,
  for which Tombo's alternative-model detection may be more appropriate
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: Snapper
  role: Implements the balanced per-k-mer sampling and KS-test comparison of native
    vs WGA signal distributions; installed as the pip package snapper-ont
  repo: DNKonanov/Snapper
- name: Tombo
  role: Resquiggles raw fast5 reads to align raw signal to the reference, producing
    the normalized signal events consumed by the comparison stage
  repo: github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files into the multi-read fast5 format
    expected by Snapper
  repo: github.com/nanoporetech/ont_fast5_api
- name: scipy
  role: Provides scipy.stats.ks_2samp used for the two-sample Kolmogorov–Smirnov test
    per k-mer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:78e97cd7bec4a9ccf77905f17c10dde6642290d05f10beeda37fdc8998ba7292
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# balanced-sampling-signal-distributions

## Summary

Balanced per-k-mer sampling of normalized nanopore signal events from native and WGA control multi-fast5 data, followed by a two-sample Kolmogorov–Smirnov test, to identify k-mers whose signal distributions differ and which likely carry a modified base.

## When to use

Use when comparing resquiggled ONT signal events between a native sample and a WGA control to rank k-mers by signal-level distribution shift and emit candidate methylation motifs.

## When NOT to use

- No matched WGA control multi-fast5 dataset is available, since the KS test requires paired native/control signal distributions
- Genome coverage depth is below the minimal coverage threshold (default 40), making balanced N=200 per-k-mer sampling unreliable
- The goal is single-base modification calling rather than de novo motif/k-mer enrichment, for which Tombo's alternative-model detection may be more appropriate

## Inputs

- Resquiggled multi-fast5 files for the native sample (Tombo event data)
- Resquiggled multi-fast5 files for the WGA control sample
- Reference genome in fasta format
- Parameters: ks_t (default 3), coverage (default 40), k_size (default 15, odd, >=11), min_conf (default 100), threads (default 8)

## Outputs

- passed_motifs_[strand]_[contig_name].fasta containing the seq_set of candidate modified-base k-mers

## How to apply

Read the resquiggled multi-fast5 files from the sample and control directories with h5py/ont-tombo event data, keying each read's normalized signal events by genome position and strand. Enumerate all odd-sized k-mers of the reference genome (k_size default 15, minimum 11) in fasta format for both strands and aggregate per-k-mer signal values into a hash table. Draw equal-size (N=200) balanced signal samples from native and WGA for each k-mer to avoid coverage bias between samples. Run scipy.stats.ks_2samp per k-mer and retain candidates whose -log KS-test p-value exceeds ks_t (default 3) with confidence above min_conf (default 100). Write the retained seq_set of candidate modified-base k-mers to passed_motifs_[strand]_[contig_name].fasta.

## Related tools

- **Snapper** (Implements the balanced per-k-mer sampling and KS-test comparison of native vs WGA signal distributions; installed as the pip package snapper-ont) — DNKonanov/Snapper
- **Tombo** (Resquiggles raw fast5 reads to align raw signal to the reference, producing the normalized signal events consumed by the comparison stage) — github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files into the multi-read fast5 format expected by Snapper) — github.com/nanoporetech/ont_fast5_api
- **scipy** (Provides scipy.stats.ks_2samp used for the two-sample Kolmogorov–Smirnov test per k-mer)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- passed_motifs_[strand]_[contig_name].fasta exists for each strand/contig and contains only k-mers of odd size (default 15) drawn from the reference genome
- Every retained k-mer has -log KS-test p-value above the ks_t threshold (default 3) and confidence above min_conf (default 100)
- Native and WGA signal samples per k-mer are equal size (N=200), i.e. no coverage imbalance remains in the test input
- Per-k-mer coverage in both native and control datasets meets the minimal genome coverage depth (default 40) before testing
- Known methylated motifs (e.g. validated on H.pylori J99 against prior PacBio data) appear in the candidate seq_set

## Limitations

- Python 3.7 is required; later versions might be incompatible because of inner biopython dependencies
- k_size must be odd and should not be less than 11 (default 15)
- Balanced sampling caps per-k-mer signal values at N=200, so very low-coverage positions cannot be evaluated reliably (minimal coverage depth default 40)
- Snapper does not extract all methylation sites in other tools' settings (e.g. MEME fails on H.pylori even with Nanodisco's iterative approach), so results should be cross-validated

## Evidence

- [readme] Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified.: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
- [methods] fast5 resquiggling requirement: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [methods] multi-fast5 conversion requirement: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [readme] default ks_t threshold: "-ks_t KS_T            -log ks_test p-value (default 3)."
- [readme] k-mer size constraints: "-k_size K_SIZE        k-mer size, must be odd, 
                        should not be less than 11 (default 15)"
