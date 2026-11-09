---
name: alignment-coverage-identity-filtering
description: 'Use when benchmarking genome polishing or assembly accuracy against
  a trusted reference: fragment the reference into pseudo-long reads, align them to
  each assembly, and filter mappings by query coverage and identity thresholds to
  compute comparable QV-style accuracy metrics.'
when_to_use_negative:
- The ground truth is a variant benchmark set (e.g. GIAB high-confidence variants)
  rather than a trusted reference genome, in which case variant-counting evaluation
  (as done for HG005 against GRCh37) is more appropriate
- The assembly lacks a high-quality trusted reference, so k-mer-based QV estimation
  with Merqury/yak is the only feasible accuracy assessment
- Alignments come from noisy real long reads rather than reference-derived pseudo-reads,
  where coverage/identity filtering thresholds calibrated for pseudo-reads would be
  invalid
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: minimap2
  role: Aligns 500 kb pseudo-long reads against each assembly with the map-hifi preset
    (-cx map-hifi, -t 5)
  repo: https://github.com/lh3/minimap2/
- name: samtools
  role: Sorts and indexes the minimap2 mapping output for each assembly
- name: seqkit
  role: Splits the reference FASTA into 500 kb pseudo-long reads (or a Python/seqio
    script)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:f5d7293977b4a94ab9272035fee002119aac5c75d31a4a2424ccdc3531841552
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - NextPolish2
  - minimap2
  - samtools
  - seqkit
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# alignment-coverage-identity-filtering

## Summary

An alignment-based accuracy evaluation scheme in which ground-truth reference sequences (pseudo-long reads) are aligned to genome assemblies and only high-quality alignments are retained — filtering out mappings with query coverage ≤90% or alignment identity ≤98% — before computing alignment-identity QV and 100%-identity read fractions to compare polishing tools such as NextPolish2 and RM.

## When to use

Use when benchmarking genome polishing or assembly accuracy against a trusted reference: fragment the reference into pseudo-long reads, align them to each assembly, and filter mappings by query coverage and identity thresholds to compute comparable QV-style accuracy metrics.

## When NOT to use

- The ground truth is a variant benchmark set (e.g. GIAB high-confidence variants) rather than a trusted reference genome, in which case variant-counting evaluation (as done for HG005 against GRCh37) is more appropriate
- The assembly lacks a high-quality trusted reference, so k-mer-based QV estimation with Merqury/yak is the only feasible accuracy assessment
- Alignments come from noisy real long reads rather than reference-derived pseudo-reads, where coverage/identity filtering thresholds calibrated for pseudo-reads would be invalid

## Inputs

- Ground-truth reference genome FASTA (e.g. corrected CHM13 T2T genome)
- Candidate assembly FASTA files (original and polished, e.g. RM-polished and NextPolish2-polished)
- minimap2 BAM/PAF alignments of pseudo-reads against each assembly

## Outputs

- Filtered alignment set of retained pseudo-read mappings (~5868 reads, ~2.93 Gb after filtering)
- qv_report.tsv with per-assembly alignment-identity QV and fraction of 100%-identity pseudo-reads

## How to apply

First fragment the trusted reference genome (e.g. the CHM13 T2T genome) into pseudo-long reads — 6250 reads of 500 kb each, using seqkit split or a Python/seqio script. Align the 500 kb pseudo-reads against each assembly under comparison (original, RM-polished, NextPolish2-polished) with minimap2 using the map-hifi preset (-cx map-hifi, -t 5), then sort and index each mapping with samtools. Filter the alignments, discarding those with query coverage ≤90% or alignment identity ≤98%; in the CHM13 benchmark this retained approximately 5868 reads (~2.93 Gb). For each assembly, compute the alignment-identity QV from the identity distribution of the retained alignments and tabulate per-assembly QV plus the fraction of 100%-identity pseudo-reads in a qv_report.tsv. Correct application is validated when polishing effects match expectations — e.g. NextPolish2 raising the 100%-identity read fraction from 9.48% to 9.58% while the RM-polished assembly's QV decreases relative to the original assembly.

## Related tools

- **minimap2** (Aligns 500 kb pseudo-long reads against each assembly with the map-hifi preset (-cx map-hifi, -t 5)) — https://github.com/lh3/minimap2/
- **samtools** (Sorts and indexes the minimap2 mapping output for each assembly)
- **seqkit** (Splits the reference FASTA into 500 kb pseudo-long reads (or a Python/seqio script))

## Examples

```
minimap2 -cx map-hifi -t 5 asm.fa chm13_pseudo_reads.fa | samtools sort -o aln.sort.bam; samtools index aln.sort.bam
```

## Evaluation signals

- Filtering retains approximately 5868 pseudo-reads (~2.93 Gb) from the 6250 × 500 kb input set after discarding alignments with query coverage ≤90% or identity ≤98%
- The 100%-identity read fraction increases from 9.48% (original) to 9.58% (NextPolish2-polished)
- The RM-polished assembly shows a decreased alignment-identity QV relative to the original assembly
- qv_report.tsv contains per-assembly QV values and 100%-identity read fractions for all three assemblies
- Every BAM/PAF mapping is sorted and indexed before filtering

## Limitations

- Strict coverage/identity cutoffs (≤90% coverage, ≤98% identity) discard genuine alignments, so the metric reflects only high-confidence regions and may hide errors elsewhere
- The evaluation assumes the reference used for pseudo-read generation is itself accurate; errors in the reference propagate into the benchmark
- NextPolish2's performance depends on short-read data to check whether a k-mer contains errors, so evaluation outcomes depend on short-read quality and coverage

## Evidence

- [other] Align the 500 kb pseudo-reads against each of the three assemblies (original, RM-polished, NextPolish2-polished) with Minimap2 (-cx map-hifi, -t 5) and sort/index each mapping with samtools: [evidence span withheld — anchor status: not_found]
- [other] Filter the alignments, discarding those with query coverage <=90% or alignment identity <=98%, retaining approximately 5868 reads (~2.93 Gb).: [evidence span withheld — anchor status: not_found]
- [other] NextPolish2 slightly increased the proportion of reads with 100% mapping identity from 9.48% to 9.58%: [evidence span withheld — anchor status: not_found]
- [methods] winnowmap or minimap2 for HiFi mapping: "Prepare HiFi mapping file ([winnowmap](https://github.com/marbl/Winnowmap) or [minimap2](https://github.com/lh3/minimap2/))."
- [readme] minimap2 map-hifi preset usage: [evidence span withheld — anchor status: external_doc]
- [methods] Merqury-based QV evaluation context: "we employed three approaches to evaluate the genome's accuracy before and after polishing:

1. **Merqury:** To assess the quality value (QV)."
