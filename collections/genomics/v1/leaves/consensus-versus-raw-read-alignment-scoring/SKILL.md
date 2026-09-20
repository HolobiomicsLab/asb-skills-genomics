---
name: consensus-versus-raw-read-alignment-scoring
description: Use when you need to validate that consensus sequences called from isoCirc
  raw reads reduce error rate, or to infer RCA copy number (raw read length ÷ consensus
  length) and bin error rates by copy number.
when_to_use_negative:
- You only have single-pass reads with no RCA concatemer structure and no consensus
  sequences — there is no consensus to compare against
- You want to correct or output polished consensus sequences — this skill only scores
  error rates; consensus calling itself is done separately with TRF
- Data are PacBio circular consensus sequencing (CCS) reads rather than isoCirc nanopore
  RCA reads — the copy-number inference from read-length ratios does not apply
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: Aligns each raw isoCirc nanopore read back to its consensus sequence to obtain
    per-base match, mismatch, insertion, and deletion counts; must be version >= 2.11
    per the isoCirc dependency list
  repo: https://github.com/lh3/minimap2
- name: isoCirc
  role: Pipeline that generated the raw reads and TRF-called consensus sequences being
    compared; installed via pip or from source
  repo: https://github.com/Xinglab/isoCirc
- name: Python
  role: Used (with pandas) to compute per-base error rates, infer copy numbers, bin
    reads, and write the summary CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:4e195f6d10d442493538528c4f9404fdea66b76f8b836fe619906110c44217a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Python
  - isoCirc
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# consensus-versus-raw-read-alignment-scoring

## Summary

Quantifies the accuracy gain of isoCirc consensus calling by aligning each raw nanopore RCA read back to its TRF-called consensus with minimap2, computing per-base error rates (1 − identity), and stratifying them by inferred RCA copy number.

## When to use

Use when you need to validate that consensus sequences called from isoCirc raw reads reduce error rate, or to infer RCA copy number (raw read length ÷ consensus length) and bin error rates by copy number.

## When NOT to use

- You only have single-pass reads with no RCA concatemer structure and no consensus sequences — there is no consensus to compare against
- You want to correct or output polished consensus sequences — this skill only scores error rates; consensus calling itself is done separately with TRF
- Data are PacBio circular consensus sequencing (CCS) reads rather than isoCirc nanopore RCA reads — the copy-number inference from read-length ratios does not apply

## Inputs

- isoCirc raw nanopore long reads (FASTA/FASTQ) from HEK293 data
- TRF-called consensus sequences from those raw reads (default TRF parameters: match 2, mismatch 7, indel 7, match probability 80, indel probability 10, min score 100, max period 2000)
- Raw read lengths and consensus lengths for copy-number inference

## Outputs

- error_rates_by_copy_number.csv — per-copy-number-bin mean error rates plus summary statistics (average copy number, average raw-read length, average consensus length)
- Per-base alignment match/mismatch/insertion/deletion counts from minimap2 raw-read-vs-consensus alignments
- Per-read RCA copy-number estimates (floor of raw length / consensus length)

## How to apply

Load the HEK293 isoCirc raw nanopore reads (FASTA/FASTQ) and the consensus sequences called by Tandem Repeats Finder (TRF default parameters: match 2, mismatch 7, indel 7, match probability 80, indel probability 10, minimum alignment score 100, maximum period size 2000). Align each raw read back to its consensus using minimap2 (>= 2.11), obtaining per-base matches, mismatches, insertions, and deletions from the CIGAR. Infer each read's RCA copy number as floor(raw read length / consensus length), compute per-read error rate as 1 − identity from the alignment statistics, and bin reads by copy number. Aggregate per-bin mean error rates with Python/pandas, along with summary values for average copy number (~14.5), average raw-read length (~5,679 nt), and average consensus length (~460 nt), and save to error_rates_by_copy_number.csv. The rationale mirrors circular consensus sequencing: multiple concatenated copies of the same circRNA template in one RCA read allow consensus calling that reduces per-base error.

## Related tools

- **minimap2** (Aligns each raw isoCirc nanopore read back to its consensus sequence to obtain per-base match, mismatch, insertion, and deletion counts; must be version >= 2.11 per the isoCirc dependency list) — https://github.com/lh3/minimap2
- **isoCirc** (Pipeline that generated the raw reads and TRF-called consensus sequences being compared; installed via pip or from source) — https://github.com/Xinglab/isoCirc
- **Python** (Used (with pandas) to compute per-base error rates, infer copy numbers, bin reads, and write the summary CSV)

## Examples

```
minimap2 -x map-ont --secondary=no consensus.fa raw_isocirc_reads.fa > read_vs_consensus.paf
```

## Evaluation signals

- Error rate decreases monotonically with increasing RCA copy number, with consensus from reads with >10 copies showing the lowest error rate (Supplementary Fig. 1)
- Summary statistics approximate reported averages: copy number ~14.5, raw reads ~5,679 nt, consensus ~460 nt
- Per-read copy number is consistent with raw read length / consensus length ratio (reads ~12x the consensus length on average)
- minimap2 alignment of each read to its own consensus yields a single dominant alignment (not spliced alignment) covering the consensus

## Limitations

- Copy number is inferred as floor(raw length / consensus length), which is an approximation that may misestimate copy number when the read truncates the concatemer
- Error-rate reduction depends on sufficient sequencing depth and concatemer copies; low-copy reads yield noisy consensus and weaker error reduction
- isoCirc consensus calling uses TRF with specific parameters; deviations in TRF settings will change consensus sequences and the measured error rates

## Evidence

- [methods] Consensus calling with TRF default parameters: "Consensus calling with Tandem Repeats Finder (TRF)):"
- [methods] Filtering and mapping of consensus sequences: "Filtering and mapping of consensus sequences (minimap2):"
- [readme] minimap2 version requirement from isoCirc dependencies: [evidence span withheld — anchor status: not_found]
- [discussion] Consensus calling from isoCirc raw reads and false-positive filtering: "After calling consensus sequences from isoCirc raw reads, we use stringent criteria to remove potential false-positive detection"
- [abstract] RCA + nanopore sequencing is the basis of consensus accuracy: "Here, we report isoCirc, a strategy for sequencing full-length circRNA isoforms, using rolling circle amplification followed by nanopore long-read sequencing"
- [discussion] Circular consensus sequencing analogy for accuracy improvement: "circular consensus sequencing is a well-established approach to improve the accuracy of PacBio long reads"
