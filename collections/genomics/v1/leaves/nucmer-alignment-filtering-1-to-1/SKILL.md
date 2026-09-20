---
name: nucmer-alignment-filtering-1-to-1
description: Use when you must determine, at whole-genome scale, whether a new long-read
  assembly contains sequence absent from a prior short-read draft (and vice versa)
  via reciprocal -match scafford-level alignments.
when_to_use_negative:
- When only contiguity metrics (N50, total size) are needed and no structural/reciprocal-match
  question is being asked — the article reports these directly from assembly statistics
- When the goal is base-level consensus correction or polishing rather than structural
  synteny assessment (the article handles that separately with Racon and Pilon)
- When reads rather than assembled scaffolds are the input — TULIP's seed/bulb steps
  handle read-level processing; this skill operates on final scaffold FASTAs
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: MUMmer (nucmer)
  role: Whole-genome aligner run with --maxmatch --minmatch 100 to align TULIP scaffolds
    against the AZBK01 draft, emitting the .delta alignment file
- name: MUMmer (delta-filter)
  role: Filters the nucmer delta file with -1 to keep only 1-to-1 (best reciprocal)
    alignments before plotting
- name: MUMmer (mummerplot)
  role: Renders the filtered 1-to-1 alignments as an ordered scaffold-vs-contig synteny
    plot using --layout
- name: TULIP
  role: Produced the query nanopore assembly scaffolds (bulb/eel_scaffold_*.fasta)
    that are aligned against the draft
  repo: github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:78e9d7b0cfbc01cccabd5aba4e90cbfa283d92200d50f2bf3a5159aa76e9e423
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MUMmer (delta-filter)
  - MUMmer (mummerplot)
  - MUMmer (nucmer)
  - MUMmer (nucmer, delta-filter, mummerplot)
  - TULIP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nucmer-alignment-filtering-1-to-1

## Summary

A whole-genome structural comparison workflow that aligns the uncorrected TULIP nanopore-based European eel assembly against the 2012 Illumina-based AZBK01 draft using MUMmer nucmer, retains only best reciprocal (1-to-1) alignments with delta-filter -1, and visualizes synteny with mummerplot to assess whether either assembly contains sequence absent from the other.

## When to use

Use when you must determine, at whole-genome scale, whether a new long-read assembly contains sequence absent from a prior short-read draft (and vice versa) via reciprocal best-match scafford-level alignments.

## When NOT to use

- When only contiguity metrics (N50, total size) are needed and no structural/reciprocal-match question is being asked — the article reports these directly from assembly statistics
- When the goal is base-level consensus correction or polishing rather than structural synteny assessment (the article handles that separately with Racon and Pilon)
- When reads rather than assembled scaffolds are the input — TULIP's seed/bulb steps handle read-level processing; this skill operates on final scaffold FASTAs

## Inputs

- TULIP nanopore assembly scaffolds in FASTA format (bulb/eel_scaffold_*.fasta, e.g. bulb/eel_scaffold_999.fasta)
- AZBK01 A. anguilla draft genome assembly (NCBI WGS) as reference FASTA

## Outputs

- tulip_vs_azbk01.delta — raw nucmer alignment delta file
- tulip_vs_azbk01.filter — delta-filter -1 output containing only 1-to-1 (best reciprocal) alignments
- Mummerplot --layout synteny figure reproducing the Fig. 5a ordered scaffold-vs-contig comparison

## How to apply

Run nucmer with --maxmatch and --minmatch 100 to align all TULIP scaffold sequences (bulb/eel_scaffold_*.fasta) against the AZBK01 draft assembly, recording alignments in tulip_vs_azbk01.delta. Filter the delta file with delta-filter -1 to retain only 1-to-1 (best reciprocal) alignments, producing tulip_vs_azbk01.filter. Generate a synteny visualization of the filtered alignments with mummerplot using the --layout option to obtain an ordered scaffold-vs-contig comparison plot. Interpret reciprocal coverage: if all query (TULIP) sequence has a best reciprocal match in the reference while only small unmatched reference scaffolds lack query matches, the new assembly is structurally complete relative to the draft.

## Related tools

- **MUMmer (nucmer)** (Whole-genome aligner run with --maxmatch --minmatch 100 to align TULIP scaffolds against the AZBK01 draft, emitting the .delta alignment file)
- **MUMmer (delta-filter)** (Filters the nucmer delta file with -1 to keep only 1-to-1 (best reciprocal) alignments before plotting)
- **MUMmer (mummerplot)** (Renders the filtered 1-to-1 alignments as an ordered scaffold-vs-contig synteny plot using --layout)
- **TULIP** (Produced the query nanopore assembly scaffolds (bulb/eel_scaffold_*.fasta) that are aligned against the draft) — github.com/Generade-nl/TULIP

## Examples

```
nucmer --maxmatch --minmatch 100 AZBK01.fasta bulb/eel_scaffold_999.fasta; delta-filter -1 tulip_vs_azbk01.delta > tulip_vs_azbk01.filter; mummerplot --layout tulip_vs_azbk01.filter
```

## Evaluation signals

- Essentially all TULIP scaffold sequence has a best reciprocal (delta-filter -1) match in AZBK01 — uncovered query sequence should be near zero
- Unmatched AZBK01 scaffolds total approximately 96 Mbp with mean/N50 of 664/987 bp — small, highly fragmented scaffolds only
- The tulip_vs_azbk01.filter file contains strictly 1-to-1 alignments (each query/reference region appears in at most one alignment)
- mummerplot --layout produces an ordered plot consistent with the reported Fig. 5a layout, showing draft contigs collapsing into far fewer TULIP scaffolds
- Observed structural redundancy is consistent with the TULIP assembly's reported N50 of 1.2 Mbp versus the highly fragmented Illumina-only draft

## Limitations

- The comparison answers structural completeness, not base-level correctness: without a high-quality reference it is 'not straightforward to establish which assembly is correct' at apparent error sites
- The two assemblies derive from a hybrid of two sequencing technologies, three nanopore chemistries, and two different animals, so divergent regions may reflect specimen/chemistry differences rather than true absence
- It cannot resolve whether inflated short-read assembly size reflects missing genomic fraction in the nanopore data or artificially inflated draft size ('This can be explained either by the absence of a substantial fraction of the genome from the nanopore data or assembly, or by an artificially inflated genome size for the short-read assemblies')

## Evidence

- [abstract] The resulting genome assembly significantly improves on a previous draft based on short reads only, both in terms of contiguity (N50 1.2 Mbp) and structural quality.: "The resulting genome assembly significantly improves on a previous draft based on short reads only, both in terms of contiguity (N50 1.2 Mbp) and structural quality."
- [results] Full-genome alignment confirms all TULIP sequence is present in the 2012 draft while only small draft scaffolds lack matches.: [evidence span withheld — anchor status: not_found]
- [methods] nucmer parameters and alignment target as executed in the workflow.: [evidence span withheld — anchor status: not_found]
- [readme] TULIP scaffold file conventions used as alignment query input.: "Long reads on one line and in FASTA format. The read identifiers should be unique in each file, so be careful if you concatenate data!"
