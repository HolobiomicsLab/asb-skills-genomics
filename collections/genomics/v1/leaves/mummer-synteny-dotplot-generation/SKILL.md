---
name: mummer-synteny-dotplot-generation
description: 'Use when comparing two genome assemblies (FASTA sets) to test bidirectional
  completeness: whether one assembly contains sequence missing from the other, and
  to visualize syntenic layout as in the Fig. 5a eel comparison.'
when_to_use_negative:
- When only read-level validation (not whole-assembly structural comparison) is needed,
  e.g. checking consensus accuracy with Racon/Pilon — this skill compares finished
  scaffold sets only
- When a high-quality reference genome exists for structural validation — the article
  notes no such reference was available, and this skill only gives reciprocal assembly-vs-assembly
  agreement, not ground truth
- When quantifying base-level accuracy rather than presence/absence and layout of
  large syntenic blocks
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: MUMmer (nucmer)
  role: Full-genome alignment of TULIP scaffolds against AZBK01 with --maxmatch --minmatch
    100, producing the .delta file
- name: MUMmer (delta-filter)
  role: Filters nucmer delta output with -1 to keep only 1-to-1 (best reciprocal)
    alignments
- name: MUMmer (mummerplot)
  role: Renders the ordered scaffold-vs-contig synteny plot using the --layout option
- name: TULIP
  role: Source of the nanopore-based assembly (bulb/eel_scaffold_*.fasta) being aligned
    against the draft
  repo: github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:af154ef66912a05cba09eeda4ff122c14372bf7fdb9c160a02ddb26d77c96b67
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MUMmer (delta-filter)
  - MUMmer (mummerplot)
  - MUMmer (nucmer)
  - MUMmer (nucmer, delta-filter, mummerplot)
  - TULIP
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mummer-synteny-dotplot-generation

## Summary

A MUMmer-based whole-genome alignment and synteny visualization workflow that aligns an assembly (e.g. the TULIP nanopore eel scaffolds) against a reference draft (AZBK01) with nucmer, filters to reciprocal-best 1-to-1 alignments, and renders an ordered scaffold-vs-contig dot plot to detect sequence present in one assembly but absent from the other.

## When to use

Use when comparing two genome assemblies (FASTA sets) to test bidirectional completeness: whether one assembly contains sequence missing from the other, and to visualize syntenic layout as in the Fig. 5a eel comparison.

## When NOT to use

- When only read-level validation (not whole-assembly structural comparison) is needed, e.g. checking consensus accuracy with Racon/Pilon — this skill compares finished scaffold sets only
- When a high-quality reference genome exists for structural validation — the article notes no such reference was available, and this skill only gives reciprocal assembly-vs-assembly agreement, not ground truth
- When quantifying base-level accuracy rather than presence/absence and layout of large syntenic blocks

## Inputs

- TULIP nanopore assembly scaffolds in FASTA (bulb/eel_scaffold_*.fasta)
- Reference draft assembly FASTA (NCBI WGS AZBK01, Illumina-based A. anguilla draft)

## Outputs

- tulip_vs_azbk01.delta (nucmer alignment file)
- tulip_vs_azbk01.filter (delta-filter -1 reciprocal-best 1-to-1 alignments)
- Ordered scaffold-vs-contig synteny dot plot (mummerplot --layout, Fig. 5a layout)

## How to apply

Run nucmer with --maxmatch and --minmatch 100 to align all TULIP scaffold sequences (bulb/eel_scaffold_*.fasta) against the NCBI WGS AZBK01 draft assembly, recording alignments in a .delta file. Filter the delta with delta-filter -1 to retain only 1-to-1 (best reciprocal) alignments. Generate the synteny visualization with mummerplot --layout to obtain an ordered, scaffold-vs-contig comparison plot reproducing the article's Fig. 5a. Judge success by reciprocal coverage: essentially all TULIP scaffold sequence should have a best reciprocal match in AZBK01, while the draft's unmatched scaffolds are restricted to the tiny fraction (~96 Mbp total, mean/N50 664/987 bp) that the nanopore assembly restructured or absorbed.

## Related tools

- **MUMmer (nucmer)** (Full-genome alignment of TULIP scaffolds against AZBK01 with --maxmatch --minmatch 100, producing the .delta file)
- **MUMmer (delta-filter)** (Filters nucmer delta output with -1 to keep only 1-to-1 (best reciprocal) alignments)
- **MUMmer (mummerplot)** (Renders the ordered scaffold-vs-contig synteny plot using the --layout option)
- **TULIP** (Source of the nanopore-based assembly (bulb/eel_scaffold_*.fasta) being aligned against the draft) — github.com/Generade-nl/TULIP

## Examples

```
nucmer --maxmatch --minmatch 100 tulip_vs_azbk01 bulb/eel_scaffold_999.fasta AZBK01.fasta; delta-filter -1 tulip_vs_azbk01.delta > tulip_vs_azbk01.filter; mummerplot --layout tulip_vs_azbk01.filter
```

## Evaluation signals

- Essentially all TULIP scaffold sequence has a best reciprocal (1-to-1) match in AZBK01 after delta-filter -1
- Only the tiny unmatched draft scaffolds lack TULIP matches — expected ~96 Mbp total with mean/N50 of 664/987 bp
- The mummerplot --layout output reproduces the ordered Fig. 5a scaffold-vs-contig comparison
- The .delta and .filter files are non-empty and the filtered alignments cover both large and small scaffolds consistently with the reported structural improvement

## Limitations

- In the absence of a high-quality reference, this alignment cannot establish which assembly is correct at apparent error sites — it only shows reciprocal presence/absence of sequence
- Draft scaffolds with no reciprocal best match (~96 Mbp) could reflect genome absent from the nanopore data/assembly OR an artificially inflated short-read assembly size — the dot plot cannot distinguish these
- TULIP scaffolds that could be merged based on long reads remain separate if reads lack a fortuitous seed alignment, which can fragment synteny blocks in the plot

## Evidence

- [other] nucmer parameters: [evidence span withheld — anchor status: not_found]
- [other] delta-filter step: [evidence span withheld — anchor status: not_found]
- [other] mummerplot step: [evidence span withheld — anchor status: not_found]
- [other] finding: [evidence span withheld — anchor status: not_found]
- [discussion] no reference limitation: "In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct"
