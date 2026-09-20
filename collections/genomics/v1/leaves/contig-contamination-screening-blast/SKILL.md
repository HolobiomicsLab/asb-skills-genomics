---
name: contig-contamination-screening-blast
description: Use when orphan/local-assembly contigs from a linked-read insertion pipeline
  must be screened for non-human contamination against nt/nr before reference placement
  and insertion-call filtering.
when_to_use_negative:
- The goal is to detect known viral/non-human sequences of interest — the nt/nr screen
  will discard exactly those contigs
- Input reads/contigs are already aligned and placed on the reference, and only placement-level
  filtering is needed
- Insertions of interest are ≤ 300 bp — the pipeline explicitly targets and outputs
  only > 300 bp events
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: BLAST
  role: BlastN screen of orphan contigs against nt/nr to remove contigs aligning to
    non-human references
- name: Novel-X
  role: Produces the orphan contigs and downstream QUAST-like filtering/insertion-call
    output that this screening feeds
  repo: github.com/1dayac/Novel-X
- name: Minimap2
  role: Aligns contamination-screened contigs to the reference genome (e.g. GRCh38)
    for anchor placement
- name: QUAST
  role: Source of the best-alignment-subset filtering procedure (maximize continuously
    covered base pairs) applied after screening
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:c29218bf4c1dbd1f5f10514afbac4e6049cfca1e2e265448b7f90fb783f7874e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - Minimap2
  - Novel-X
  - QUAST
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# contig-contamination-screening-blast

## Summary

Screens orphan contigs emitted by Novel-X's local-assembly stages with BLASTN against nt/nr to remove contigs matching non-human references (contamination), before QUAST-like alignment filtering and NRS insertion calling.

## When to use

Use when orphan/local-assembly contigs from a linked-read insertion pipeline must be screened for non-human contamination against nt/nr before reference placement and insertion-call filtering.

## When NOT to use

- The goal is to detect known viral/non-human sequences of interest — the nt/nr screen will discard exactly those contigs
- Input reads/contigs are already aligned and placed on the reference, and only placement-level filtering is needed
- Insertions of interest are ≤ 300 bp — the pipeline explicitly targets and outputs only > 300 bp events

## Inputs

- Orphan contigs (FASTA) from Novel-X local-assembly stages (NRS insertion assembly + barcode-based insertions reassembly)
- nt/nr BLAST database
- Reference genome (e.g. GRCh38 FASTA) for downstream Minimap2 alignment

## Outputs

- Set of contamination-free contigs passing to Minimap2 alignment
- filtered_insertion_candidates.vcf — insertions > 300 bp with summed anchor lengths > 300 bp
- Separate file of non-qualifying insertion placements

## How to apply

Load the orphan contigs produced by the upstream Novel-X stages (NRS insertion assembly and barcode-based insertions reassembly) and run BlastN against the nt/nr database, discarding contigs that align to non-human references — an optional contamination-removal procedure similar to Pamir's. Note this step can also drop known viral sequences of interest, so treat it as an optional, reviewable decision point. Pass the surviving contigs to Minimap2 for alignment to the reference genome (e.g. GRCh38), then apply the QUAST-resembling filtering procedure that chooses best alignment subsets maximizing continuously covered base pairs. Retain contigs where adjacent anchor alignments are close on the reference but far apart on the contig, and finally output only insertions longer than 300 bp with summed anchor lengths exceeding 300 bp (filtered_insertion_candidates.vcf), storing non-qualifying placements separately.

## Related tools

- **BLAST** (BlastN screen of orphan contigs against nt/nr to remove contigs aligning to non-human references)
- **Novel-X** (Produces the orphan contigs and downstream QUAST-like filtering/insertion-call output that this screening feeds) — github.com/1dayac/Novel-X
- **Minimap2** (Aligns contamination-screened contigs to the reference genome (e.g. GRCh38) for anchor placement)
- **QUAST** (Source of the best-alignment-subset filtering procedure (maximize continuously covered base pairs) applied after screening)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git  # then run the Snakemake pipeline whose post-assembly stage BlastN-screens orphan contigs against nt/nr
```

## Evaluation signals

- filtered_insertion_candidates.vcf contains only insertions longer than 300 bp whose summed anchor lengths exceed 300 bp
- No contig in the downstream Minimap2 input aligns to a non-human reference in nt/nr
- Orphan contigs recruiting fewer than t=5 barcodes are already discarded before/alongside this stage
- On the paper's simulated benchmark, the full filtering chain supports Novel-X precision of 0.999

## Limitations

- The optional contamination-removal step may also drop known viral (non-human) sequences of interest
- Novel-X needs a substantial number of distinct barcodes; small insertions often lack sufficient coverage, producing too few barcodes for the downstream stages
- Repository notes a known issue: 'There is a problem on filter_target_contig stage at the moment'

## Evidence

- [methods] contamination removal: "we can perform a contamination removal procedure similar to what was previously done in Pamir (31)"
- [methods] QUAST-like filter: "a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs"
- [methods] output thresholds: [evidence span withheld — anchor status: not_found]
- [methods] barcode discard threshold: [evidence span withheld — anchor status: not_found]
- [readme] Novel-X repo: "git clone --recursive git@github.com:1dayac/Novel-X.git"
