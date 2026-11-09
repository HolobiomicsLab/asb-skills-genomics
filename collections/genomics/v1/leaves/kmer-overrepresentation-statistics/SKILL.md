---
name: kmer-overrepresentation-statistics
description: Use when de novo methylation motifs must be extracted from Oxford Nanopore
  reads by testing k-mer over-representation against a reference genome background
  with chi-squared statistics.
when_to_use_negative:
- Input is raw fast5 signal that has not been resquiggled with Tombo and converted
  to multi-fast5 with ont_fast5_api — run those preprocessing steps first
- The goal is calling per-read modified bases at known motifs (use Tombo's alternative
  model or remora instead) rather than de novo motif enrichment
- Reference genome FASTA for the strain is unavailable, since the chi-squared enrichment
  requires genome background frequencies
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: Snapper
  role: Implements the greedy chi-squared motif-enrichment loop (selection, adjustment,
    seq_set filtering, termination) exposed via snapper-ont CLI flags -k_size, -long_k_size,
    -max_motifs, -min_conf
  repo: DNKonanov/Snapper
- name: scipy
  role: Provides the chi-squared statistics used to score k-mer over-representation
    against the reference genome background
- name: Tombo
  role: Resquiggles raw fast5 files upstream so k-mers carry signal-level statistics
    feeding the enrichment step
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files to the multi-fast5 format Snapper
    consumes
  repo: https://github.com/nanoporetech/ont_fast5_api
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:2c3f6317e576fc1de50e66e30defafbf4260d5c0ecb27874cac9b02f822459c6
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
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# kmer-overrepresentation-statistics

## Summary

Snapper's greedy motif-enrichment procedure: iteratively score k-mers in a seq_set for over-representation against a reference genome background using chi-squared statistics, extract the top position-specific methylation motif, filter the seq_set, and repeat until min_conf/max_motifs termination.

## When to use

Use when de novo methylation motifs must be extracted from Oxford Nanopore reads by testing k-mer over-representation against a reference genome background with chi-squared statistics.

## When NOT to use

- Input is raw fast5 signal that has not been resquiggled with Tombo and converted to multi-fast5 with ont_fast5_api — run those preprocessing steps first
- The goal is calling per-read modified bases at known motifs (use Tombo's alternative model or remora instead) rather than de novo motif enrichment
- Reference genome FASTA for the strain is unavailable, since the chi-squared enrichment requires genome background frequencies

## Inputs

- Passed-motif k-mer seq_set from upstream Snapper k-mer statistics (task_006)
- Reference genome in FASTA format (negative background)
- Sample and control (WGA) multi-fast5 directories processed via Tombo resquiggling and ont_fast5_api conversion

## Outputs

- final_motifs.fasta containing a sorted list of extracted methylation motifs with their confidence (chi-squared) levels

## How to apply

Load the passed-motif k-mer seq_set produced upstream by Snapper's per-k-mer statistics stage and the reference genome in FASTA format as the negative background. For each candidate k-mer in the seq_set, compute an enrichment statistic versus the genome using scipy.stats chi-squared tests, with odd k-mer sizes (default k_size 15, must not be smaller than 11; default long_k_size 29, must not be smaller than 21). Greedily select the motif with the highest over-representation score, perform position-specific motif adjustment, and remove from the seq_set all k-mers explained by the selected motif before repeating. Terminate when the minimal chi-squared confidence falls below the min_conf threshold or when max_motifs (default 20) motifs have been extracted. Finally, sort the surviving motifs and write them with their confidence levels to final_motifs.fasta.

## Related tools

- **Snapper** (Implements the greedy chi-squared motif-enrichment loop (selection, adjustment, seq_set filtering, termination) exposed via snapper-ont CLI flags -k_size, -long_k_size, -max_motifs, -min_conf) — DNKonanov/Snapper
- **scipy** (Provides the chi-squared statistics used to score k-mer over-representation against the reference genome background)
- **Tombo** (Resquiggles raw fast5 files upstream so k-mers carry signal-level statistics feeding the enrichment step) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files to the multi-fast5 format Snapper consumes) — https://github.com/nanoporetech/ont_fast5_api

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- final_motifs.fasta exists and contains a sorted motif list
- Every emitted motif meets the min_conf confidence threshold (chi-squared statistic at or above the cutoff)
- Number of extracted motifs does not exceed max_motifs (default 20)
- k_size and long_k_size passed as odd values (>=11 and >=21 respectively; defaults 15 and 29)
- Each greedily extracted motif no longer appears in the filtered seq_set during subsequent iterations

## Limitations

- k_size must be odd and not smaller than 11 (default 15); long_k_size must be odd and not smaller than 21 (default 29)
- Snapper depends on Python 3.7 — later versions might be incompatible because of inner biopython dependencies
- Tombo, a required upstream dependency, is deprecated and unsupported by Oxford Nanopore (replaced by remora)
- Requires both a sample and a WGA control fast5 directory plus a reference genome; termination defaults trade sensitivity against runtime via min_conf and max_motifs

## Evidence

- [abstract] Greedy chi-squared motif selection: "We present Snapper, a new highly sensitive approach, to extract methylation motif sequences based on a greedy motif selection algorithm."
- [intro] Position-specific greedy enrichment: "The core feature of Snapper in comparison with other tools is a new high-sensitive greedy algorithm that is used for position-specific motif enrichment."
- [readme] Balanced per-k-mer statistics: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
- [readme] CLI parameters for enrichment: "[-k_size K_SIZE] [-long_k_size LONG_K_SIZE] [-max_motifs MAX_MOTIFS] [-min_conf MIN_CONF]"
- [methods] Upstream preprocessing requirement: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
