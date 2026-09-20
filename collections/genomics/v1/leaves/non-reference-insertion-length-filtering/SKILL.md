---
name: non-reference-insertion-length-filtering
description: Use when orphan contigs from synthetic long-read local assembly (10X/TELL-Seq/stLFR
  barcoded reads) must be filtered into final non-reference insertion calls, discarding
  spurious or short candidates before VCF output.
when_to_use_negative:
- The analysis targets insertions of 300 bp or shorter — the paper's analysis explicitly
  focuses on insertions longer than 300 bp, and small insertions usually lack sufficient
  barcode coverage for anchor assembly
- Input is raw reads rather than assembled orphan contigs — this skill operates after
  the NRS insertion assembly and insertions reassembly stages
- A whole-genome de novo assembly approach (e.g. SuperNova-based) is already being
  used — Novel-X is designed to avoid that computationally expensive route
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Novel-X
  role: Pipeline whose post-assembly filtering stage (including the QUAST-like procedure
    and 300 bp cutoffs) implements this skill; outputs filtered_insertion_candidates.vcf
  repo: https://github.com/1dayac/Novel-X
- name: BLAST
  role: BlastN of orphan contigs against nt/nr to remove non-human contamination before
    reference alignment
- name: Minimap2
  role: Aligns contamination-filtered contigs to the reference genome so anchor placements
    and insertion signatures can be evaluated
- name: QUAST
  role: Reference for the best-alignment-subset filtering idea (maximizing continuously
    covered base pairs) that Novel-X reimplements
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:d07232d3b6782c2f5fb081d3791d56e0805f861bfc1d02ae1baabbccbafd81b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Novel-X
  tools:
  - BLAST
  - Minimap2
  - Novel-X
  - QUAST
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# non-reference-insertion-length-filtering

## Summary

Post-assembly filtering of orphan contigs produced by Novel-X local assembly: BLAST-based contamination removal, Minimap2 alignment to the reference, a QUAST-resembling best-alignment-subset filter, and a >300 bp insertion/anchor length cutoff that decides which NRS insertion calls are written to the final VCF.

## When to use

Use when orphan contigs from synthetic long-read local assembly (10X/TELL-Seq/stLFR barcoded reads) must be filtered into final non-reference insertion calls, discarding spurious or short candidates before VCF output.

## When NOT to use

- The analysis targets insertions of 300 bp or shorter — the paper's analysis explicitly focuses on insertions longer than 300 bp, and small insertions usually lack sufficient barcode coverage for anchor assembly
- Input is raw reads rather than assembled orphan contigs — this skill operates after the NRS insertion assembly and insertions reassembly stages
- A whole-genome de novo assembly approach (e.g. SuperNova-based) is already being used — Novel-X is designed to avoid that computationally expensive route

## Inputs

- Orphan contigs from Novel-X local assembly (SPAdes/Velvet/SuperNova output with k=77, cov-cutoff=3)
- Reference genome FASTA (e.g. GRCh38)
- nt/nr BLAST database
- Barcode-recruitment counts per contig (contigs with fewer than t=5 barcodes already discarded upstream)

## Outputs

- filtered_insertion_candidates.vcf containing insertions >300 bp with summed anchor lengths >300 bp
- Separate file of non-qualifying contig placements

## How to apply

Load the orphan contigs emitted by the upstream Novel-X stages (NRS insertion assembly and barcode-based insertions reassembly), run BlastN against nt/nr and drop contigs aligning to non-human references (an optional Pamir-like contamination removal step), then align the remaining contigs to the reference genome (e.g. GRCh38) with Minimap2. Apply the QUAST-resembling filtering procedure: choose best alignment subsets from the alignment set that maximize the number of continuously covered base pairs. Retain contigs whose adjacent anchor alignments are close on the reference but far apart on the contig (the insertion signature) and discard all others. Finally keep only insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp, writing qualifying calls to filtered_insertion_candidates.vcf and storing non-qualifying placements separately.

## Related tools

- **Novel-X** (Pipeline whose post-assembly filtering stage (including the QUAST-like procedure and 300 bp cutoffs) implements this skill; outputs filtered_insertion_candidates.vcf) — https://github.com/1dayac/Novel-X
- **BLAST** (BlastN of orphan contigs against nt/nr to remove non-human contamination before reference alignment)
- **Minimap2** (Aligns contamination-filtered contigs to the reference genome so anchor placements and insertion signatures can be evaluated)
- **QUAST** (Reference for the best-alignment-subset filtering idea (maximizing continuously covered base pairs) that Novel-X reimplements)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git
```

## Evaluation signals

- Every record in filtered_insertion_candidates.vcf satisfies both the >300 bp insertion and the >300 bp summed-anchor criteria; shorter events must appear only in the separate non-qualifying placements file
- On the paper's simulated LRsim benchmark, applying this filtering supported Novel-X precision of 0.999
- No contig aligning to a non-human nt/nr reference survives the BLAST step (known viral sequences of interest may be intentionally sacrificed)
- Within each retained contig, the chosen alignment subset maximizes continuously covered base pairs, mirroring QUAST's procedure
- Retained contigs show the insertion signature: adjacent anchor alignments close on the reference but far apart on the contig

## Limitations

- The optional nt/nr contamination-removal step may also drop known viral sequences of interest
- Only insertions longer than 300 bp are considered; small insertions typically have too few associated barcodes and insufficient coverage for anchor assembly
- Known issue on the filter_target_contig stage is noted in the article's methods/repository notes

## Evidence

- [methods] a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs: "a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs"
- [methods] 300 bp output cutoff: [evidence span withheld — anchor status: not_found]
- [methods] contamination removal step: "we can perform a contamination removal procedure similar to what was previously done in Pamir (31)"
- [methods] barcode discard threshold: [evidence span withheld — anchor status: not_found]
- [readme] Novel-X scope from README: "Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information."
