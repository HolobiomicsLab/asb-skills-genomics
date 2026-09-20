---
name: insertion-breakpoint-signature-detection
description: Use when orphan contigs from a local-assembly insertion pipeline must
  be screened against a reference genome to discard spurious placements and output
  non-reference insertion calls with breakpoints defined by anchor alignments.
when_to_use_negative:
- Input reads still need local assembly of the insertion sequence first — this skill
  operates on already-assembled orphan contigs
- Short-read-only insertion callers (MindTheGap, Pamir, PopIns2) are the intended
  approach when no synthetic long-read barcode data exist
- Insertions of interest are ≤ 300 bp — the output filter and the paper's analysis
  explicitly restrict to insertions longer than 300 bp
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Novel-X
  role: upstream pipeline producing orphan contigs; implements the signature detection
    and >300 bp output filter
  repo: https://github.com/1dayac/Novel-X
- name: BLAST
  role: optional contamination removal — BlastN orphan contigs against nt/nr and discard
    non-human alignments
- name: Minimap2
  role: aligns filtered contigs to the reference genome to produce the anchor alignments
    used for signature detection
- name: QUAST
  role: source of the best-alignment-subset filtering concept (maximize continuously
    covered base pairs)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:79ca809befd1785496de0dd64bbab6e35c3688dee2b11595b5aba73741e3367c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Novel-X
  tools:
  - BLAST
  - Minimap2
  - Novel-X
  - QUAST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# insertion-breakpoint-signature-detection

## Summary

Filters and places locally assembled insertion contigs onto a reference genome (e.g. GRCh38) using a QUAST-resembling best-alignment-subset procedure, then retains only contigs showing the anchor-pair insertion signature (anchors close on the reference but far apart on the contig) with insertion and summed-anchor lengths exceeding 300 bp.

## When to use

Use when orphan contigs from a local-assembly insertion pipeline must be screened against a reference genome to discard spurious placements and output non-reference insertion calls with breakpoints defined by anchor alignments.

## When NOT to use

- Input reads still need local assembly of the insertion sequence first — this skill operates on already-assembled orphan contigs
- Short-read-only insertion callers (MindTheGap, Pamir, PopIns2) are the intended approach when no synthetic long-read barcode data exist
- Insertions of interest are ≤ 300 bp — the output filter and the paper's analysis explicitly restrict to insertions longer than 300 bp

## Inputs

- orphan contigs from Novel-X NRS insertion assembly and barcode-based reassembly (FASTA)
- reference genome, e.g. GRCh38 FASTA
- nt/nr BLAST database for contamination screening

## Outputs

- filtered_insertion_candidates.vcf containing insertions > 300 bp with summed anchor lengths > 300 bp
- separate file of non-qualifying contig placements

## How to apply

Load orphan contigs emitted by upstream Novel-X local-assembly stages (NRS insertion assembly and barcode-based reassembly). Optionally run BlastN against nt/nr and remove contigs aligning to non-human references as a contamination-removal step. Align surviving contigs to the reference genome with Minimap2, then apply the QUAST-resembling filtering: choose best alignment subsets from the alignment set that maximize the number of continuously covered base pairs. Detect the insertion signature — adjacent anchor alignments close on the reference but far apart on the contig — and discard all other contigs. Finally output only insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp, writing qualifying calls to a VCF (e.g. filtered_insertion_candidates.vcf) and storing non-qualifying placements in a separate file.

## Related tools

- **Novel-X** (upstream pipeline producing orphan contigs; implements the signature detection and >300 bp output filter) — https://github.com/1dayac/Novel-X
- **BLAST** (optional contamination removal — BlastN orphan contigs against nt/nr and discard non-human alignments)
- **Minimap2** (aligns filtered contigs to the reference genome to produce the anchor alignments used for signature detection)
- **QUAST** (source of the best-alignment-subset filtering concept (maximize continuously covered base pairs))

## Evaluation signals

- Every VCF record satisfies both criteria: insertion length > 300 bp and sum of anchor lengths > 300 bp
- Retained contigs show the insertion signature: adjacent anchor alignments close on the reference but far apart on the contig
- Selected alignment subsets maximize continuously covered base pairs, matching QUAST-like best-subset behavior
- On the paper's simulated benchmark, this filtering supports Novel-X precision of 0.999
- Non-qualifying placements are written to a separate file rather than silently dropped

## Limitations

- The optional BLAST contamination-removal step may also drop known viral sequences of interest
- Small insertions usually recruit too few barcodes, so Novel-X needs a substantial number of distinct barcodes and coverage is often insufficient for anchor assembly
- Repository notes a known issue: 'There is a problem on filter_target_contig stage at the moment'

## Evidence

- [methods] QUAST-like best-alignment-subset filtering: "a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs"
- [methods] 300 bp output thresholds: [evidence span withheld — anchor status: not_found]
- [methods] anchor placement on reference: [evidence span withheld — anchor status: not_found]
- [methods] contig alignment quality filter: [evidence span withheld — anchor status: not_found]
- [readme] Novel-X detects and genotypes insertions from linked reads: "Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information."
