---
name: methylation-motif-de-novo-detection
description: Use when you have paired native and WGA-control ONT FAST5 read sets plus
  a reference genome and need to discover methylated sequence motifs de novo, without
  prior knowledge of methyltransferase specificity.
when_to_use_negative:
- You need per-read modified-base calls at individual positions rather than enriched
  motif sequences — Tombo's modified-base detection or Remora is the alternative
- The dataset lacks a matched WGA control read set, since Snapper's statistics rely
  on sample-versus-control signal-level k-mer comparisons
- 'You expect highly similar motifs that may collide: Nanodisco''s fully automated
  mode was shown to merge H.pylori J99 motifs CATG and GCGC into GCRYG, so beware
  of degenerate-collision artifacts when motifs share k-mer context'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: Snapper
  role: 'Core tool: greedy motif selection over sample-vs-WGA k-mer statistics to
    extract methylation motif sequences from ONT data; installed as pip package snapper-ont'
  repo: DNKonanov/Snapper
- name: Tombo
  role: Resquiggles raw FAST5 signal against the reference genome before motif calling
  repo: github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read FAST5 files into multi-read FAST5 format
    required by Snapper
  repo: github.com/nanoporetech/ont_fast5_api
- name: snapper-ont
  role: The pip-distributed package of Snapper used for installation
  repo: DNKonanov/Snapper
- name: Nanodisco
  role: 'Comparison baseline: its automated mode merges J99 motifs CATG and GCGC into
    GCRYG, illustrating the collision problem Snapper avoids'
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:ebb3c3bd60947c0572f54e5aa8f1d06f1298badc84afab66d413250ef5dc2d41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DNKonanov/Snapper
  tools:
  - Nanodisco
  - Python
  - Snapper
  - Tombo
  - ont_fast5_api
  - pip
  - snapper-ont
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# methylation-motif-de-novo-detection

## Summary

De novo detection of bacterial methylation motifs from Oxford Nanopore sequencing by comparing native versus WGA-control read signal at k-mer level using Snapper's greedy position-specific motif enrichment algorithm, replacing manual, iterative workflows like MEME+Nanodisco.

## When to use

Use when you have paired native and WGA-control ONT FAST5 read sets plus a reference genome and need to discover methylated sequence motifs de novo, without prior knowledge of methyltransferase specificity.

## When NOT to use

- You need per-read modified-base calls at individual positions rather than enriched motif sequences — Tombo's modified-base detection or Remora is the alternative
- The dataset lacks a matched WGA control read set, since Snapper's statistics rely on sample-versus-control signal-level k-mer comparisons
- You expect highly similar motifs that may collide: Nanodisco's fully automated mode was shown to merge H.pylori J99 motifs CATG and GCGC into GCRYG, so beware of degenerate-collision artifacts when motifs share k-mer context

## Inputs

- Native (methylated) ONT raw FAST5 read directory
- Whole-genome amplified (WGA) control ONT raw FAST5 directory
- Reference genome in FASTA format
- Resquiggled FAST5 files converted to multi-read FAST5 format via ont_fast5_api

## Outputs

- final_motifs_[strand]_[contig_name].fasta motif files per strand and contig
- Per-strain detected motif tables for comparison against REBASE reference motifs
- Novel motif calls (e.g. CCAG/TGGCCA in T. phaeum, WGGCCW in C. perfringens)

## How to apply

Resquiggle the raw FAST5 files against the reference genome with Tombo, then convert them to multi-read FAST5 format using ont_fast5_api so Snapper can consume them. Run the snapper command supplying the native multi-fast5 directory as -sample_fast5dir, the WGA multi-fast5 directory as -control_fast5dir, and the reference FASTA as -reference; default parameters are ks_t=3 (-log KS-test p-value), coverage=40, threads=8, k_size=15 (must be odd and >=11), long_k_size=29, and min_conf=100. Snapper computes balanced per-k-mer statistics comparing native against WGA control signal and applies its high-sensitive greedy algorithm for position-specific motif enrichment, requiring no manual control during the enrichment process. Collect the emitted final_motifs_[strand]_[contig_name].fasta files per strain and compare each detected motif against a reference set (e.g. REBASE methyltransferase specificities), resolving degenerate IUPAC motifs to concrete instances where needed (e.g. CACNNNNNRTAAA resolved to CACNNNNNATAAA).

## Related tools

- **Snapper** (Core tool: greedy motif selection over sample-vs-WGA k-mer statistics to extract methylation motif sequences from ONT data; installed as pip package snapper-ont) — DNKonanov/Snapper
- **Tombo** (Resquiggles raw FAST5 signal against the reference genome before motif calling) — github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read FAST5 files into multi-read FAST5 format required by Snapper) — github.com/nanoporetech/ont_fast5_api
- **snapper-ont** (The pip-distributed package of Snapper used for installation) — DNKonanov/Snapper
- **Nanodisco** (Comparison baseline: its automated mode merges J99 motifs CATG and GCGC into GCRYG, illustrating the collision problem Snapper avoids)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- Per-strain motif tables recover all expected REBASE reference methylation motifs for the four SRP219538 strains (T. phaeum, N. gonorrhoeae, M. hungatei, C. perfringens)
- The long REBASE motif CACNNNNNRTAAA is resolved to the concrete instance CACNNNNNATAAA with no other unexplained deviations
- Novel motifs CCAG/TGGCCA in T. phaeum DSM 12270 and WGGCCW in C. perfringens ATCC 13124 are detected in addition to reference motifs
- Enrichment sensitivity exceeds MEME coupled with Tombo and Nanodisco, which failed to extract all H.pylori methylation sites de novo
- No manual intervention during enrichment was required and results are deterministic across runs

## Limitations

- Python 3.7 is required; later versions might be incompatible because of inner biopython dependencies
- k_size must be odd and not less than 11 (default 15); long_k_size default is 29
- Degenerate motif collisions can merge distinct motifs — Nanodisco's automated mode merged CATG and GCGC into GCRYG on H.pylori J99
- Tombo is deprecated by Oxford Nanopore in favor of Remora; the pipeline depends on this unsupported resquiggling tool
- External validation on SRP219538 showed one motif deviation (CACNNNNNRTAAA inferred as CACNNNNNATAAA) rather than exact recovery

## Evidence

- [abstract] greedy motif selection: "We present Snapper, a new highly sensitive approach, to extract methylation motif sequences based on a greedy motif selection algorithm."
- [abstract] manual-free higher sensitivity: "Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments"
- [methods] resquiggle workflow: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [methods] multi-fast5 conversion: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [abstract] pip installation: "Snapper is implemented using Python and is freely available as a pip package named "snapper-ont.""
- [intro] core greedy algorithm: "The core feature of Snapper in comparison with other tools is a new high-sensitive greedy algorithm that is used 
for position-specific motif enrichment."
- [readme] python version constraint: "- python 3.7 (later versions might be incompatible because of inner biopython dependencies)"
- [intro] k-mer statistics: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
