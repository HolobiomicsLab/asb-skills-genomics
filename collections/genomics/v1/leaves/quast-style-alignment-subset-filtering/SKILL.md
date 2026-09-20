---
name: quast-style-alignment-subset-filtering
description: Use when orphan contigs from local assembly must be validated against
  a reference genome before insertion calling, e.g. to purge spurious contig alignments
  from Novel-X NRS candidates prior to VCF output.
when_to_use_negative:
- Input is raw reads or read-level alignment QC rather than assembled contigs — the
  read-level filters (average base quality >20, mapping quality <10, >20% soft-clipped)
  are a different upstream stage
- Analysis targets insertions <=300 bp, since Novel-X explicitly restricts analysis
  to insertions longer than 300 bp where barcode coverage is sufficient
- Detecting reference-revealed deletions or other SV classes that do not produce the
  anchor-pair signature (adjacent on reference, far apart on contig)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0102
tools:
- name: Novel-X
  role: Pipeline hosting the QUAST-like best-alignment-subset filtering of contig
    alignments during insertion placement
  repo: https://github.com/1dayac/Novel-X
- name: BLAST
  role: BlastN of orphan contigs against nt/nr to discard contigs aligning to non-human
    references (contamination removal, as in Pamir)
- name: Minimap2
  role: Aligns the remaining contigs to the reference genome (e.g. GRCh38) to produce
    the alignment set filtered by the QUAST-like procedure
- name: QUAST
  role: 'Source of the best-alignment-subset selection idea: maximizing continuously
    covered base pairs'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:000b78448102f1fc4f587c2735e921d3343b0947301a8f11ba884a1c2ca0a322
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Novel-X
  tools:
  - BLAST
  - Minimap2
  - Novel-X
  - QUAST
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# quast-style-alignment-subset-filtering

## Summary

A post-assembly contig filtering step from Novel-X that resembles the QUAST procedure: given a set of contig-to-reference alignments, it greedily selects best alignment subsets that maximize continuously covered base pairs, so that spurious local-assembly contigs are discarded before NRS insertion calls are finalized.

## When to use

Use when orphan contigs from local assembly must be validated against a reference genome before insertion calling, e.g. to purge spurious contig alignments from Novel-X NRS candidates prior to VCF output.

## When NOT to use

- Input is raw reads or read-level alignment QC rather than assembled contigs — the read-level filters (average base quality >20, mapping quality <10, >20% soft-clipped) are a different upstream stage
- Analysis targets insertions <=300 bp, since Novel-X explicitly restricts analysis to insertions longer than 300 bp where barcode coverage is sufficient
- Detecting reference-revealed deletions or other SV classes that do not produce the anchor-pair signature (adjacent on reference, far apart on contig)

## Inputs

- Orphan contigs from Novel-X NRS insertion assembly and barcode-based insertions reassembly stages
- Reference genome FASTA (e.g. GRCh38)
- BlastN alignment of contigs against nt/nr for contamination removal (optional)

## Outputs

- filtered_insertion_candidates.vcf with insertions >300 bp and summed anchor length >300 bp
- Separate file storing non-qualifying insertion placements
- Discarded spurious/contaminant contig alignments (filtered out)

## How to apply

Load orphan contigs emitted by Novel-X's NRS insertion assembly and barcode-based reassembly stages; optionally run BlastN against nt/nr to remove contigs aligning to non-human references (contamination removal as in Pamir). Align surviving contigs to the reference genome (e.g. GRCh38) with Minimap2, first discarding alignments with low-quality scores or >=20% soft/hard-clipped bases. Then apply the QUAST-resembling procedure: choose best alignment subsets from the alignment set that maximize the number of continuously covered base pairs, resolving multi-mapping and overlapping hits. Detect insertion signatures by retaining contigs where adjacent anchor alignments are close on the reference but far apart on the contig, discarding all others. Finally output only insertions longer than 300 bp whose summed anchor lengths exceed 300 bp, writing qualifying calls to filtered_insertion_candidates.vcf and storing non-qualifying placements separately.

## Related tools

- **Novel-X** (Pipeline hosting the QUAST-like best-alignment-subset filtering of contig alignments during insertion placement) — https://github.com/1dayac/Novel-X
- **BLAST** (BlastN of orphan contigs against nt/nr to discard contigs aligning to non-human references (contamination removal, as in Pamir))
- **Minimap2** (Aligns the remaining contigs to the reference genome (e.g. GRCh38) to produce the alignment set filtered by the QUAST-like procedure)
- **QUAST** (Source of the best-alignment-subset selection idea: maximizing continuously covered base pairs)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git
```

## Evaluation signals

- Output VCF contains only insertions longer than 300 bp whose summed anchor lengths exceed 300 bp
- Non-qualifying placements are written to a separate file, not merged into filtered_insertion_candidates.vcf
- Contigs with alignments carrying >=20% soft/hard-clipped or low-quality scores are absent from downstream insertion calls
- Retained contigs show adjacent anchor alignments close on the reference but far apart on the contig (insertion signature)
- On the paper's simulated benchmark this filtering supported Novel-X precision of 0.999

## Limitations

- The optional BlastN contamination-removal step may also drop known viral sequences of interest since it discards contigs aligning to non-human references
- Novel-X needs a substantial number of distinct barcodes; small insertions usually lack sufficient coverage, and contigs recruiting fewer than t=5 barcodes are discarded upstream of this filter
- The repository notes a known issue: 'There is a problem on filter_target_contig stage at the moment'

## Evidence

- [methods] a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs: "a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs"
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [methods] filter read alignments with low-quality scores or with a large fraction (>= 20%) of soft-clipped or hard-clipped sequences: [evidence span withheld — anchor status: not_found]
- [methods] we can perform a contamination removal procedure similar to what was previously done in Pamir (31): "we can perform a contamination removal procedure similar to what was previously done in Pamir (31)"
- [readme] Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information.: "Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information."
