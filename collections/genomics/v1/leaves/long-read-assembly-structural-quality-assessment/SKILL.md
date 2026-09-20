---
name: long-read-assembly-structural-quality-assessment
description: Use when a new de novo assembly (e.g. nanopore/TULIP scaffolds) must
  be validated against an existing draft genome to determine mutually missing sequence,
  unmatched scaffold fractions, and structural improvement.
when_to_use_negative:
- The goal is base-level error assessment or polishing of the new assembly — use Racon/Illumina-Pilon
  correction instead, as this skill assesses structural presence/absence, not consensus
  accuracy
- No prior draft or reference assembly exists for comparison, so reciprocal matching
  cannot be performed
- Assemblies differ so drastically in ploidy or haplotype content that 1-to-1 filtering
  would discard biologically real duplicated alignments
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: MUMmer (nucmer)
  role: Whole-genome aligner run with --maxmatch --minmatch 100 to produce the .delta
    alignment of query scaffolds vs the draft
- name: delta-filter
  role: Filters the nucmer delta to 1-to-1 best reciprocal alignments (tulip_vs_azbk01.filter)
- name: mummerplot
  role: Generates the ordered scaffold-vs-contig synteny plot with --layout
- name: TULIP
  role: Source of the query nanopore-based scaffolds being assessed (bulb/eel_scaffold_*.fasta)
  repo: github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:ff650834634789cdb2f57321b79f198efdd58a631f6febb365975bfb32549138
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MUMmer (nucmer)
  - TULIP
  - delta-filter
  - mummerplot
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# long-read-assembly-structural-quality-assessment

## Summary

Whole-genome alignment workflow (MUMmer nucmer/delta-filter/mummerplot) that compares a new long-read assembly against a prior short-read draft to quantify which sequence each assembly uniquely contains, reproducing the eel TULIP-vs-AZBK01 synteny analysis of Fig. 5a.

## When to use

Use when a new de novo assembly (e.g. nanopore/TULIP scaffolds) must be validated against an existing draft genome to determine mutually missing sequence, unmatched scaffold fractions, and structural improvement.

## When NOT to use

- The goal is base-level error assessment or polishing of the new assembly — use Racon/Illumina-Pilon correction instead, as this skill assesses structural presence/absence, not consensus accuracy
- No prior draft or reference assembly exists for comparison, so reciprocal matching cannot be performed
- Assemblies differ so drastically in ploidy or haplotype content that 1-to-1 filtering would discard biologically real duplicated alignments

## Inputs

- Query assembly scaffolds in FASTA (e.g. bulb/eel_scaffold_999.fasta from the TULIP bulb output)
- Reference draft assembly in FASTA (e.g. NCBI WGS AZBK01, the Illumina-based A. anguilla draft)
- MUMmer suite installed (nucmer, delta-filter, mummerplot)

## Outputs

- nucmer alignment file tulip_vs_azbk01.delta
- 1-to-1 filtered alignment file tulip_vs_azbk01.filter
- Ordered synteny plot (scaffold-vs-contig layout, reproducing Fig. 5a)
- Quantification of unmatched reference scaffolds (total Mbp, mean/N50 lengths)

## How to apply

Run nucmer with --maxmatch and --minmatch 100 to align all query assembly scaffolds (e.g. bulb/eel_scaffold_*.fasta) against the reference draft assembly, writing alignments to a .delta file. Filter the delta with delta-filter -1 to retain only 1-to-1 (best reciprocal) alignments, then visualise the filtered alignments with mummerplot --layout to obtain an ordered scaffold-vs-contig synteny plot. Interpret coverage reciprocally: in the eel case, essentially all TULIP scaffold sequence had a best reciprocal match in AZBK01, while only ~96 Mbp of tiny draft scaffolds (mean/N50 664/987 bp) lacked TULIP matches. Correct application is judged by confirming the long-read assembly loses no query sequence while quantifying the unmatched reference fraction that reflects its structural improvement.

## Related tools

- **MUMmer (nucmer)** (Whole-genome aligner run with --maxmatch --minmatch 100 to produce the .delta alignment of query scaffolds vs the draft)
- **delta-filter** (Filters the nucmer delta to 1-to-1 best reciprocal alignments (tulip_vs_azbk01.filter))
- **mummerplot** (Generates the ordered scaffold-vs-contig synteny plot with --layout)
- **TULIP** (Source of the query nanopore-based scaffolds being assessed (bulb/eel_scaffold_*.fasta)) — github.com/Generade-nl/TULIP

## Examples

```
nucmer --maxmatch --minmatch 100 bulb/eel_scaffold_*.fasta AZBK01.fasta -p tulip_vs_azbk01 && delta-filter -1 tulip_vs_azbk01.delta > tulip_vs_azbk01.filter && mummerplot --layout --filter -p fig5a tulip_vs_azbk01.filter
```

## Evaluation signals

- Essentially all query (TULIP) scaffold sequence is covered by 1-to-1 filtered alignments against the reference draft
- Only the expected tiny unmatched reference scaffolds lack query matches (~96 Mbp total, mean/N50 664/987 bp)
- tulip_vs_azbk01.filter contains only 1-to-1 (best reciprocal) alignments after delta-filter -1
- mummerplot --layout renders an ordered synteny plot consistent with the reported Fig. 5a scaffold-vs-contig comparison
- Unmatched reference fraction is consistent with the reported contiguity gain (nanopore assembly N50 1.2 Mbp vs fragmented draft)

## Limitations

- In the absence of a high-quality reference, structural alignment cannot establish which assembly is correct at apparent error sites
- The short-read draft may be artificially inflated, or the nanopore assembly may lack genomic sequence — the alignment alone does not distinguish these explanations for size discrepancy
- TULIP scaffolds that could be merged via long reads remain separate if reads lack a fortuitous seed alignment, so unmatched-query signal must be interpreted with the assembly's known suboptimality in mind

## Evidence

- [other] Run nucmer with --maxmatch and --minmatch 100 to align all TULIP scaffold sequences: [evidence span withheld — anchor status: not_found]
- [other] delta-filter -1 retains only 1-to-1 alignments: [evidence span withheld — anchor status: not_found]
- [other] 96 Mbp of draft scaffolds lack reciprocal matches: [evidence span withheld — anchor status: not_found]
- [abstract] nanopore assembly improves contiguity over short-read draft: "The resulting genome assembly significantly improves on a previous draft based on short reads only, both in terms of contiguity (N50 1.2 Mbp) and structural quality."
- [discussion] no high-quality reference to judge correctness: "In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct"
