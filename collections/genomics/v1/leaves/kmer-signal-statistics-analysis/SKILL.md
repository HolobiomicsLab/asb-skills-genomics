---
name: kmer-signal-statistics-analysis
description: Use when you have native and control (e.g., WGA) nanopore fast5 reads
  plus a reference genome and need de novo detection of methylated k-mers/motifs from
  raw signal statistics.
when_to_use_negative:
- Input reads are already basecalled motif calls from other methyl callers and no
  raw fast5 signal is available — resquiggling and k-mer signal statistics require
  raw signal
- You need per-read or per-site methylation quantification rather than de novo motif
  discovery — Snapper is a motif caller, not a per-site annotator
- Python environment is >3.7 and cannot be downgraded — the README warns later versions
  might be incompatible because of inner biopython dependencies
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0080
tools:
- name: Tombo
  role: Resquiggles raw nanopore signal to reference basecalls, producing the signal-to-sequence
    alignments the k-mer statistics depend on
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled fast5 files to the multi-fast5 format required by snapper
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: snapper-ont
  role: Computes per-k-mer signal statistics (ks_test) and runs the greedy position-specific
    motif enrichment to emit methylation motifs
  repo: DNKonanov/Snapper
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:82286a69fa89f1ceffbc25d7bbc81477043c12d84b02be896c5f725f8e9acdad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - Python
  - Snapper
  - Tombo
  - ont_fast5_api
  - snapper-ont
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# kmer-signal-statistics-analysis

## Summary

Compute per-k-mer signal statistics from resquiggled ONT raw signal to identify k-mers likely to carry modified bases, then enrich methylation motifs using Snapper's high-sensitive greedy position-specific motif selection algorithm.

## When to use

Use when you have native and control (e.g., WGA) nanopore fast5 reads plus a reference genome and need de novo detection of methylated k-mers/motifs from raw signal statistics.

## When NOT to use

- Input reads are already basecalled motif calls from other methyl callers and no raw fast5 signal is available — resquiggling and k-mer signal statistics require raw signal
- You need per-read or per-site methylation quantification rather than de novo motif discovery — Snapper is a motif caller, not a per-site annotator
- Python environment is >3.7 and cannot be downgraded — the README warns later versions might be incompatible because of inner biopython dependencies

## Inputs

- Resquiggled multi-read fast5 files for the native sample (e.g., J99_multi)
- Resquiggled multi-read fast5 files for the WGA control sample (e.g., J99_wga_multi)
- Reference genome in FASTA format (e.g., J99.fasta)
- Python 3.7 conda environment with ont-tombo, ont-fast5-api, and snapper-ont installed

## Outputs

- passed_motifs_[strand]_[contig_name].fasta files listing all k-mers most likely carrying a modified base with per-k-mer confidence values
- final_motifs FASTA files with the enriched methylation motif set
- A deterministic motif table comparable across samples/conditions

## How to apply

Resquiggle raw fast5 files against the reference genome with Tombo, then convert the resquiggled files to multi-fast5 format using ont_fast5_api. Run snapper with -sample_fast5dir (native reads), -control_fast5dir (WGA control reads), and -reference (fasta genome); Snapper computes balanced statistics for each k-mer likely to be modified by comparing native versus control signal distributions. Key parameters: ks_t=-log of the ks_test p-value (default 3), coverage threshold (default 40, raised to 100x for H.pylori J99), k_size=15 (odd, >=11), long_k_size=29, and min_conf=100. The greedy algorithm performs position-specific motif enrichment without manual control, emitting passed_motifs_[strand]_[contig].fasta files of k-mers most likely carrying a modified base with per-k-mer confidence values, and final_motifs FASTA files after filtering (e.g., removing composite motifs like TCNNGA that are not individual methylation motifs).

## Related tools

- **Tombo** (Resquiggles raw nanopore signal to reference basecalls, producing the signal-to-sequence alignments the k-mer statistics depend on) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled fast5 files to the multi-fast5 format required by snapper) — https://github.com/nanoporetech/ont_fast5_api
- **snapper-ont** (Computes per-k-mer signal statistics (ks_test) and runs the greedy position-specific motif enrichment to emit methylation motifs) — DNKonanov/Snapper

## Examples

```
snapper -sample_fast5dir J99_multi -control_fast5dir J99_wga_multi -reference J99.fasta -coverage 100
```

## Evaluation signals

- passed_motifs and final_motifs FASTA files are present for every analyzed contig/strand
- Recovered motif set reproduces independently characterized motifs (e.g., H.pylori J99 motifs previously characterized by PacBio sequencing)
- Confidence values meet the min_conf threshold (default 100) for retained k-mers
- Genome coverage depth satisfies the -coverage threshold (default 40; 100x used for J99) before motif calls are trusted
- Results are deterministic across reruns — Snapper is designed to return deterministic results unlike MEME-based enrichment

## Limitations

- Python 3.7 is required; later versions might be incompatible because of inner biopython dependencies
- Some detected k-mer patterns are composite and not individual methylation motifs (e.g., TCNNGA in J99 was judged not to be an individual motif) and require manual interpretation
- Tombo is deprecated/unsupported by Oxford Nanopore in favor of Remora
- Performance claims are benchmarked against MEME coupled with Tombo or Nanodisco on H.pylori J99 and four external datasets; other organisms may not be validated

## Evidence

- [readme] Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified.: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
- [intro] Greedy algorithm is the core feature: "The core feature of Snapper in comparison with other tools is a new high-sensitive greedy algorithm that is used 
for position-specific motif enrichment."
- [readme] Resquiggle then convert to multi-fast5: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software. 
After resquiggling, fast5 files should be converted to the multi-fast5 format using"
- [methods] Snapper command line usage with sample/control/reference: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [readme] ks_test parameter documented in usage: "-ks_t KS_T            -log ks_test p-value (default 3)."
- [abstract] Determinism and sensitivity advantage: "Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments"
