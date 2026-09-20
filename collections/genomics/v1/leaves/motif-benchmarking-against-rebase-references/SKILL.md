---
name: motif-benchmarking-against-rebase-references
description: Use when running Snapper de novo motif discovery on native vs WGA nanopore
  FAST5 pairs and you need to validate recovered motifs against known REBASE reference
  methylation motifs for the strain.
when_to_use_negative:
- Input data are not Oxford Nanopore reads with a matched native/WGA pair — Snapper's
  greedy motif enrichment requires sample and control FAST5 directories plus a reference
  genome.
- The goal is single-base modification calling at specific positions rather than de
  novo motif discovery — Tombo-style per-site detection or alternative model approaches
  are the wrong scope.
- The strain has no REBASE reference (or expected motif set) available — benchmarking
  against REBASE references cannot be the acceptance criterion.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Snapper
  role: Runs the greedy position-specific motif enrichment on native vs WGA multi-fast5
    data with default parameters (ks_t=3, coverage=40, threads=8, k_size=15, long_k_size=29,
    min_conf=100) to produce final motif FASTA files.
  repo: DNKonanov/Snapper
- name: Tombo
  role: Resquiggles raw FAST5 files against each strain's reference genome as the
    first preprocessing step before motif calling.
  repo: github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read FAST5 files to multi-fast5 format required
    by Snapper.
  repo: github.com/nanoporetech/ont_fast5_api
- name: snapper-ont
  role: The pip-distributed package providing the 'snapper' command used for motif
    detection.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:943bc742f55c584ab8245befce53ce640f9b68d25c859c0c26e2f73fd2f71f8c
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# motif-benchmarking-against-rebase-references

## Summary

Benchmark de novo methylation motifs called by Snapper on ONT native/WGA read pairs against REBASE reference methyltransferase specificities, as demonstrated on the four external SRP219538 strains (T. phaeum, N. gonorrhoeae, M. hungatei, C. perfringens).

## When to use

Use when running Snapper de novo motif discovery on native vs WGA nanopore FAST5 pairs and you need to validate recovered motifs against known REBASE reference methylation motifs for the strain.

## When NOT to use

- Input data are not Oxford Nanopore reads with a matched native/WGA pair — Snapper's greedy motif enrichment requires sample and control FAST5 directories plus a reference genome.
- The goal is single-base modification calling at specific positions rather than de novo motif discovery — Tombo-style per-site detection or alternative model approaches are the wrong scope.
- The strain has no REBASE reference (or expected motif set) available — benchmarking against REBASE references cannot be the acceptance criterion.

## Inputs

- Native (unamplified) ONT FAST5 read files for the strain
- WGA (whole-genome amplified) control ONT FAST5 read files
- Reference genome FASTA for the strain
- REBASE reference set of known methyltransferase specificities

## Outputs

- final_motifs_[strand]_[contig_name].fasta files per strain from Snapper
- Per-strain motif tables of detected motifs
- Comparison of detected motifs against REBASE reference motifs, including degenerate-base resolution and novel-site annotation

## How to apply

Download the native and WGA whole-genome amplicon read sets (e.g. SRA SRP219538), resquiggle raw FAST5 files against the strain reference genome with Tombo, and convert to multi-fast5 with ont_fast5_api. Run 'snapper' with default parameters (ks_t = 3, coverage = 40, threads = 8, k_size = 15, long_k_size = 29, min_conf = 100), supplying -sample_fast5dir (native), -control_fast5dir (WGA) and -reference (FASTA). Collect the final_motifs_[strand]_[contig_name].fasta files per strain into per-strain motif tables and compare each detected motif against REBASE reference methyltransferase specificity. Accept the run when the tables recover all expected REBASE motifs, allowing for degenerate-base resolution (e.g. CACNNNNNRTAAA inferred as CACNNNNNATAAA) and noting novel sites such as CCAG/TGGCCA in T. phaeum and WGGCCW in C. perfringens.

## Related tools

- **Snapper** (Runs the greedy position-specific motif enrichment on native vs WGA multi-fast5 data with default parameters (ks_t=3, coverage=40, threads=8, k_size=15, long_k_size=29, min_conf=100) to produce final motif FASTA files.) — DNKonanov/Snapper
- **Tombo** (Resquiggles raw FAST5 files against each strain's reference genome as the first preprocessing step before motif calling.) — github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read FAST5 files to multi-fast5 format required by Snapper.) — github.com/nanoporetech/ont_fast5_api
- **snapper-ont** (The pip-distributed package providing the 'snapper' command used for motif detection.)

## Examples

```
snapper -sample_fast5dir ../SRP219538/Tphaeum/native_multi/ -control_fast5dir ../SRP219538/Tphaeum/wga_multi/ -reference ../SRP219538/Tphaeum/genome.fasta
```

## Evaluation signals

- Per-strain motif tables recover all expected REBASE reference motifs for T. phaeum, N. gonorrhoeae, M. hungatei, and C. perfringens.
- The only accepted deviation is resolution of the degenerate REBASE motif CACNNNNNRTAAA to the concrete instance CACNNNNNATAAA.
- Snapper additionally detects the novel CCAG/TGGCCA motifs in T. phaeum DSM 12270 and the WGGCCW motif in C. perfringens ATCC 13124.
- final_motifs_[strand]_[contig_name].fasta files are produced for each strain after the snapper run completes with default parameters.
- FAST5 preprocessing chain completed: all reads resquiggle with Tombo and convert to multi-fast5 via ont_fast5_api before motif calling.

## Limitations

- Tombo is deprecated and unsupported; Oxford Nanopore recommends Remora as a replacement, so resquiggling may require a legacy environment.
- Snapper's pip distribution requires Python 3.7 — later versions might be incompatible because of inner biopython dependencies.
- Detected motifs may resolve degenerate IUPAC references to a single concrete instance (e.g. CACNNNNNRTAAA inferred as CACNNNNNATAAA), so exact string matching against REBASE motifs is insufficient.

## Evidence

- [methods] Reproduce Snapper motif discovery on the four external SRP219538 datasets: [evidence span withheld — anchor status: not_found]
- [results] Default-parameter recovery of REBASE motifs: [evidence span withheld — anchor status: not_found]
- [readme] Preprocessing requirement: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [intro] Greedy algorithm core feature: "The core feature of Snapper in comparison with other tools is a new high-sensitive greedy algorithm that is used for position-specific motif enrichment."
- [readme] Python version constraint: "python 3.7 (later versions might be incompatible because of inner biopython dependencies)"
