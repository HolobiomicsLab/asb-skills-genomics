---
name: greedy-motif-enrichment-algorithm-implementation
description: Use when de novo methylation motif detection from resquiggled ONT multi-fast5
  data is needed and existing tools (MEME with Tombo or Nanodisco) lack sensitivity
  or determinism.
when_to_use_negative:
- Input is raw, unresquiggled single-read fast5 files — Snapper requires Tombo-resquiggled,
  multi-fast5 converted data first.
- The task is generic sequence motif discovery without nanopore signal data (e.g.,
  motif search in a plain FASTA) — this skill is specific to ONT-based methylation
  motif enrichment.
- Manual/interactive motif curation is required — Snapper is designed to run the enrichment
  process without manual control and is deterministic.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: Snapper
  role: Implements the greedy motif selection algorithm for position-specific motif
    enrichment from ONT data
  repo: https://github.com/DNKonanov/Snapper
- name: Tombo
  role: Resquiggles fast5 files to align raw nanopore signal to the reference prior
    to motif enrichment
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files into the multi-fast5 format Snapper
    consumes
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: snapper-ont
  role: Pip distribution of Snapper used to install and invoke the tool
  repo: https://github.com/DNKonanov/Snapper
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:f7a54fb03f1dde939500f4259d39209663f1d09df149a271ac74c6f0dc9d4b28
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
n_claims: 7
---

# greedy-motif-enrichment-algorithm-implementation

## Summary

Implements Snapper's high-sensitive greedy algorithm for position-specific methylation motif enrichment from Oxford Nanopore reads: it iteratively selects the most over-represented k-mer against a reference genome background, extracts it, filters the seq_set, and repeats until the min_conf/max_motifs termination criteria are met.

## When to use

Use when de novo methylation motif detection from resquiggled ONT multi-fast5 data is needed and existing tools (MEME with Tombo or Nanodisco) lack sensitivity or determinism.

## When NOT to use

- Input is raw, unresquiggled single-read fast5 files — Snapper requires Tombo-resquiggled, multi-fast5 converted data first.
- The task is generic sequence motif discovery without nanopore signal data (e.g., motif search in a plain FASTA) — this skill is specific to ONT-based methylation motif enrichment.
- Manual/interactive motif curation is required — Snapper is designed to run the enrichment process without manual control and is deterministic.

## Inputs

- Sample multi-fast5 directory (resquiggled ONT reads)
- Control multi-fast5 directory (e.g., WGA control reads)
- Reference genome in FASTA format

## Outputs

- final_motifs.fasta: sorted list of candidate methylation motifs with confidence levels
- Output directory with per-motif enrichment statistics

## How to apply

Resquiggle fast5 files with Tombo and convert them to multi-fast5 format with ont_fast5_api before running Snapper. Snapper computes statistics for each k-mer likely to be modified against the reference genome (FASTA), then greedily selects the most over-represented motif, performs position-specific motif adjustment, and filters the seq_set by removing k-mers explained by the extracted motif; the loop repeats so each iteration enriches the next motif from the residual set. Key parameters are -k_size (odd, >=11, default 15), -long_k_size (default 29), -max_motifs (default 20), and -min_conf (default 100), which together govern termination; -ks_t (-log KS-test p-value, default 3) and -coverage (minimal genome coverage depth, default 40) control read-level filtering. Surviving motifs are sorted and written with confidence levels to final_motifs.fasta; validate that every emitted motif meets min_conf and the motif count does not exceed max_motifs.

## Related tools

- **Snapper** (Implements the greedy motif selection algorithm for position-specific motif enrichment from ONT data) — https://github.com/DNKonanov/Snapper
- **Tombo** (Resquiggles fast5 files to align raw nanopore signal to the reference prior to motif enrichment) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files into the multi-fast5 format Snapper consumes) — https://github.com/nanoporetech/ont_fast5_api
- **snapper-ont** (Pip distribution of Snapper used to install and invoke the tool) — https://github.com/DNKonanov/Snapper

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- final_motifs.fasta exists and contains a sorted motif list
- Every emitted motif has a confidence level meeting the min_conf threshold (default 100)
- Number of extracted motifs does not exceed max_motifs (default 20)
- k_size and long_k_size are odd and not smaller than 11/21
- Genome coverage depth meets the minimum set by -coverage (default 40)

## Limitations

- Snapper requires Python 3.7; later versions might be incompatible because of inner biopython dependencies.
- The greedy enrichment is iterative and parametric — overly low min_conf or high max_motifs can surface weak motifs, while overly strict settings truncate motif discovery.

## Evidence

- [abstract] greedy motif selection algorithm: "We present Snapper, a new highly sensitive approach, to extract methylation motif sequences based on a greedy motif selection algorithm."
- [intro] high-sensitive greedy algorithm for position-specific motif enrichment: "The core feature of Snapper in comparison with other tools is a new high-sensitive greedy algorithm that is used 
for position-specific motif enrichment."
- [methods] snapper CLI invocation: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [readme] resquiggling and multi-fast5 conversion prerequisites: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software. 
After resquiggling, fast5 files should be converted to the multi-fast5 format using"
- [readme] k_size parameter constraint: "-k_size K_SIZE        k-mer size, must be odd, 
                        should not be less than 11 (default 15)"
