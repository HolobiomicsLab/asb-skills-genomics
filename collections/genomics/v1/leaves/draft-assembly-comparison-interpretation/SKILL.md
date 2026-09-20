---
name: draft-assembly-comparison-interpretation
description: Use when a new long-read assembly (e.g. TULIP nanopore scaffolds) must
  be compared against a published short-read draft (e.g. NCBI WGS AZBK01) to test
  reciprocal sequence presence, contiguity gains, and unmatched scaffold fractions.
when_to_use_negative:
- Do not use to adjudicate base-level accuracy at apparent error sites — in the absence
  of a high-quality reference it is not straightforward to establish which assembly
  is correct
- Do not use for polishing or consensus correction; use Racon/Pilon workflows (e.g.
  ont-assembly-polish) instead
- 'Do not use as a genome-size estimate: size discrepancies between assemblies may
  reflect an artificially inflated short-read assembly size rather than missing genomic
  fraction'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: MUMmer
  role: Provides nucmer (--maxmatch --minmatch 100) for whole-genome alignment, delta-filter
    -1 for 1-to-1 reciprocal filtering, and mummerplot --layout for the ordered synteny
    plot
- name: TULIP
  role: Produces the uncorrected nanopore assembly scaffolds (bulb/eel_scaffold_*.fasta)
    that serve as the query set in the comparison
  repo: github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:241a27339092530b2a538382e0b7042cda276a5d41f6a8ce8089964f71086c6a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MUMmer
  - TULIP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# draft-assembly-comparison-interpretation

## Summary

Whole-genome structural comparison of a new long-read assembly against a published short-read draft using MUMmer (nucmer, delta-filter, mummerplot) to quantify reciprocal coverage and identify scaffold sets present in one assembly but not the other. Used here to show that all TULIP nanopore sequence is present in the AZBK01 Illumina draft while ~96 Mbp of tiny draft scaffolds lack matches.

## When to use

Use when a new long-read assembly (e.g. TULIP nanopore scaffolds) must be compared against a published short-read draft (e.g. NCBI WGS AZBK01) to test reciprocal sequence presence, contiguity gains, and unmatched scaffold fractions.

## When NOT to use

- Do not use to adjudicate base-level accuracy at apparent error sites — in the absence of a high-quality reference it is not straightforward to establish which assembly is correct
- Do not use for polishing or consensus correction; use Racon/Pilon workflows (e.g. ont-assembly-polish) instead
- Do not use as a genome-size estimate: size discrepancies between assemblies may reflect an artificially inflated short-read assembly size rather than missing genomic fraction

## Inputs

- New assembly scaffold FASTA: TULIP nanopore scaffolds (bulb/eel_scaffold_*.fasta, e.g. bulb/eel_scaffold_999.fasta)
- Reference draft assembly FASTA: NCBI WGS AZBK01 (A. anguilla Illumina CLC bio + SSPACE draft, 969/923 Mbp)

## Outputs

- tulip_vs_azbk01.delta — raw nucmer whole-genome alignment file
- tulip_vs_azbk01.filter — delta-filter -1 output retaining only 1-to-1 (best reciprocal) alignments
- Ordered synteny plot (mummerplot --layout) reproducing the Fig. 5a scaffold-vs-contig comparison
- Inventory of unmatched draft scaffolds (~96 Mbp total, mean/N50 664/987 bp) lacking reciprocal best matches in the TULIP assembly

## How to apply

Run nucmer with --maxmatch and --minmatch 100 to align all TULIP scaffold sequences (bulb/eel_scaffold_*.fasta) against the AZBK01 draft, writing alignments to tulip_vs_azbk01.delta; --maxmatch is essential so repetitive TULIP sequence is not silently dropped. Filter the delta with delta-filter -1 to retain only 1-to-1 (best reciprocal) alignments, producing tulip_vs_azbk01.filter. Visualize with mummerplot --layout to obtain an ordered scaffold-vs-contig synteny plot reproducing the Fig. 5a layout. Interpret coverage reciprocally: here essentially all TULIP sequence has a best reciprocal match in AZBK01, while ~96 Mbp of very short draft scaffolds (mean/N50 664/987 bp) lack TULIP matches, consistent with the nanopore assembly's reported structural improvement (N50 1.2 Mbp) over the short-read-only draft.

## Related tools

- **MUMmer** (Provides nucmer (--maxmatch --minmatch 100) for whole-genome alignment, delta-filter -1 for 1-to-1 reciprocal filtering, and mummerplot --layout for the ordered synteny plot)
- **TULIP** (Produces the uncorrected nanopore assembly scaffolds (bulb/eel_scaffold_*.fasta) that serve as the query set in the comparison) — github.com/Generade-nl/TULIP

## Examples

```
nucmer --maxmatch --minmatch 100 -p tulip_vs_azbk01 AZBK01.fasta bulb/eel_scaffold_999.fasta; delta-filter -1 tulip_vs_azbk01.delta > tulip_vs_azbk01.filter
```

## Evaluation signals

- Essentially all TULIP scaffold sequence in tulip_vs_azbk01.filter has a best reciprocal match in AZBK01 (near-100% reciprocal coverage of the TULIP assembly)
- Unmatched draft scaffolds sum to approximately 96 Mbp with mean/N50 664/987 bp, matching the reported small-scaffold fraction
- delta-filter -1 output contains only 1-to-1 alignments — each TULIP scaffold and AZBK01 contig participates in at most one best reciprocal alignment
- mummerplot --layout yields an ordered scaffold-vs-contig comparison plot consistent with the Fig. 5a layout
- Contiguity contrast is consistent with the reported N50 1.2 Mbp nanopore assembly being two orders of magnitude less fragmented than the Illumina draft

## Limitations

- No high-quality reference genome was available, so which assembly is correct at apparent error sites cannot be established
- Genome-size discrepancy between assemblies is unresolved: it may reflect absence of a substantial genome fraction from the nanopore data/assembly or an artificially inflated short-read assembly size
- The TULIP assembly is likely suboptimal: scaffolds that could be merged based on long reads remain separate if the reads do not share a fortuitous seed alignment in the correct position
- Large repetitive regions and (sub)telomeric repeats may be absent from the TULIP assembly, biasing the comparison

## Evidence

- [abstract] The resulting genome assembly significantly improves on a previous draft based on short reads only, both in terms of contiguity (N50 1.2 Mbp) and structural quality.: "The resulting genome assembly significantly improves on a previous draft based on short reads only, both in terms of contiguity (N50 1.2 Mbp) and structural quality."
- [discussion] no high-quality reference for adjudication: "In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct"
- [discussion] assembly size discrepancy is ambiguous: "This can be explained either by the absence of a substantial fraction of the genome from the nanopore data or assembly, or by an artificially inflated genome size for the short-read assemblies"
- [other] nucmer parameterization from the source workflow card: [evidence span withheld — anchor status: not_found]
