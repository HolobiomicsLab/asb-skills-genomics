---
name: genome-assembly-whole-genome-alignment
description: Use when two de novo assemblies of the same genome (e.g. a new long-read
  assembly and a published short-read draft such as NCBI WGS AZBK01) must be compared
  to find unmatched scaffolds and validate structural improvement.
when_to_use_negative:
- When only a single assembly is available and the goal is quality assessment (use
  N50/QUAST-style metrics instead of reciprocal alignment).
- When reads, not assemblies, need to be aligned to a reference (use BWA MEM or minimap2
  read mapping, not nucmer assembly-to-assembly comparison).
edam_operation: http://edamontology.org/operation_0277
edam_topics:
- http://edamontology.org/topic_0193
tools:
- name: MUMmer (nucmer)
  role: whole-genome aligner run with --maxmatch --minmatch 100 to align TULIP scaffolds
    against the AZBK01 draft
- name: delta-filter
  role: filters the nucmer delta with -1 to retain only 1-to-1 best reciprocal alignments
- name: mummerplot
  role: generates the ordered scaffold-vs-contig synteny visualization using the --layout
    option
- name: TULIP
  role: produces the nanopore-based query assembly (eel_scaffold_*.fasta) being validated
    against the draft
  repo: https://github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:8d843ada7920130c0a542fd5a8129eaf558a85f0f964353031d4205d16dbe98b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Generade-nl/TULIP
  tools:
  - MUMmer (nucmer)
  - TULIP
  - delta-filter
  - mummerplot
  edam_operation: http://edamontology.org/operation_0277
  edam_topics:
  - http://edamontology.org/topic_0193
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# genome-assembly-whole-genome-alignment

## Summary

Whole-genome alignment workflow that compares two genome assemblies (here the uncorrected TULIP nanopore assembly vs. the 2012 Illumina-based AZBK01 draft of A. anguilla) with MUMmer nucmer, delta-filter, and mummerplot to reveal sequence present in one assembly but absent from the other.

## When to use

Use when two de novo assemblies of the same genome (e.g. a new long-read assembly and a published short-read draft such as NCBI WGS AZBK01) must be compared to find unmatched scaffolds and validate structural improvement.

## When NOT to use

- When only a single assembly is available and the goal is quality assessment (use N50/QUAST-style metrics instead of reciprocal alignment).
- When reads, not assemblies, need to be aligned to a reference (use BWA MEM or minimap2 read mapping, not nucmer assembly-to-assembly comparison).

## Inputs

- FASTA of query assembly scaffolds (TULIP output, e.g. bulb/eel_scaffold_999.fasta)
- FASTA of reference draft assembly (NCBI WGS AZBK01)
- MUMmer suite (nucmer, delta-filter, mummerplot)

## Outputs

- nucmer alignment file (tulip_vs_azbk01.delta)
- filtered 1-to-1 alignment file (tulip_vs_azbk01.filter)
- ordered synteny plot (scaffold-vs-contig comparison, Fig. 5a layout)
- list/total size of unmatched reference scaffolds (~96 Mbp, mean/N50 664/987 bp)

## How to apply

Align every scaffold of the new assembly (e.g. bulb/eel_scaffold_*.fasta) against the reference draft with nucmer using --maxmatch and --minmatch 100 so that even small/degenerate matches are recorded in the .delta file. Filter the delta with delta-filter -1 to keep only 1-to-1 best reciprocal alignments, ensuring each query scaffold maps to a single best reference contig. Visualize the filtered alignments with mummerplot --layout to reproduce an ordered scaffold-vs-contig synteny plot (analogous to Fig. 5a). Validate by checking that essentially all TULIP scaffold sequence has a reciprocal best match, while unmatched draft scaffolds (~96 Mbp total, mean/N50 664/987 bp) account for the size discrepancy between assemblies.

## Related tools

- **MUMmer (nucmer)** (whole-genome aligner run with --maxmatch --minmatch 100 to align TULIP scaffolds against the AZBK01 draft)
- **delta-filter** (filters the nucmer delta with -1 to retain only 1-to-1 best reciprocal alignments)
- **mummerplot** (generates the ordered scaffold-vs-contig synteny visualization using the --layout option)
- **TULIP** (produces the nanopore-based query assembly (eel_scaffold_*.fasta) being validated against the draft) — https://github.com/Generade-nl/TULIP

## Examples

```
nucmer --maxmatch --minmatch 100 -p tulip_vs_azbk01 bulb/eel_scaffold_*.fasta AZBK01.fasta; delta-filter -1 tulip_vs_azbk01.delta > tulip_vs_azbk01.filter; mummerplot --layout -p fig5a tulip_vs_azbk01.filter
```

## Evaluation signals

- After delta-filter -1, essentially all TULIP scaffold sequence has a best reciprocal match in AZBK01.
- Unmatched AZBK01 scaffolds total approximately 96 Mbp with mean/N50 of 664/987 bp — small scaffolds, not large blocks.
- mummerplot --layout yields an ordered synteny plot with TULIP scaffolds collinear against draft contigs (Fig. 5a layout).
- The nanopore assembly shows N50 ~1.2 Mbp, i.e. two orders of magnitude less fragmented than the Illumina draft.

## Limitations

- The comparison cannot establish which assembly is correct at apparent error sites in the absence of a high-quality reference.
- It is uncertain whether inflated short-read assembly sizes or missing genomic fractions explain size discrepancies; large repetitive regions and (sub)telomeric repeats may be absent from the nanopore assembly.
- TULIP scaffolds that could be merged based on long reads may remain separate if reads lack a fortuitous seed alignment, inflating apparent unmatched-query content.

## Evidence

- [methods] Run nucmer with --maxmatch and --minmatch 100 to align all TULIP scaffold sequences: [evidence span withheld — anchor status: not_found]
- [methods] delta-filter -1 retains 1-to-1 alignments: [evidence span withheld — anchor status: not_found]
- [abstract] assembly improves on short-read draft: "The resulting genome assembly significantly improves on a previous draft based on short reads only, both in terms of contiguity (N50 1.2 Mbp) and structural quality."
- [results] finding from full-genome alignment: [evidence span withheld — anchor status: not_found]
