---
name: homopolymer-run-annotation
description: Use when you have a set of genomic variant positions (e.g. truth variants
  missed by one or more sequencing technologies) plus a matching reference FASTA,
  and you need to quantify how many fall in homopolymer runs or short tandem repeats.
when_to_use_negative:
- Input is raw sequencing reads or a BAM rather than a defined variant position list
  — the skill annotates variant coordinates, not calls
- No build-matched reference FASTA is available (GRCh37 truth set with only GRCh38
  reference, or vice versa) — flank extraction would misrepresent contexts
- The question is variant-calling accuracy itself (F1 per technology) rather than
  explaining undetected variants by sequence context
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Clairvoyante
  role: Deep neural network variant caller whose per-technology undetected-variant
    sets (Illumina, PacBio, ONT) are the subject of the context annotation
  repo: https://github.com/aquaskyline/Clairvoyante
- name: samtools/pysam faidx
  role: Indexed extraction of downstream reference flanking sequence at each variant
    position from the GRCh37/GRCh38 FASTA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:913ade4a6f795bc9ee529a1d6a3bf5a2254ce7369905807d82e333879b2d32cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Illumina
  - Oxford Nanopore
  - PacBio
  - samtools/pysam faidx
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# homopolymer-run-annotation

## Summary

Annotates variant positions with reference-sequence context labels — homopolymer runs (>=4 identical consecutive bases) and short tandem repeats (>=3 units of a 1–6 bp motif) — to explain why known GIAB/1KGp3 variants go undetected by Illumina, PacBio, and/or Oxford Nanopore callers.

## When to use

Use when you have a set of genomic variant positions (e.g. truth variants missed by one or more sequencing technologies) plus a matching reference FASTA, and you need to quantify how many fall in homopolymer runs or short tandem repeats.

## When NOT to use

- Input is raw sequencing reads or a BAM rather than a defined variant position list — the skill annotates variant coordinates, not calls
- No build-matched reference FASTA is available (GRCh37 truth set with only GRCh38 reference, or vice versa) — flank extraction would misrepresent contexts
- The question is variant-calling accuracy itself (F1 per technology) rather than explaining undetected variants by sequence context

## Inputs

- Set of undetected variant positions from GIAB/1KGp3 truth variants (e.g. 12,331 missed by all of Illumina, PacBio, ONT; 178,331 missed by PacBio+ONT)
- Reference genome FASTA in build-matched GRCh37 or GRCh38 with faidx index

## Outputs

- Per-variant context annotations (homopolymer run / short tandem repeat labels)
- Fraction summary (e.g. 26.67% homopolymer / 29.45% STR for the 12,331 set) written to homopolymer_str_classification.csv

## How to apply

For each variant in the undetected set (e.g. the 12,331 variants missed by Illumina, PacBio and ONT callers simultaneously, or the 178,331 missed by both PacBio and ONT), extract the downstream reference flank from GRCh37/GRCh38 using a genome FASTA reader (pysam/samtools faidx), matching the truth-set reference build. Scan the extracted flanks with regex/tandem-repeat logic for homopolymer runs (>=4 identical consecutive bases) and STRs (>=3 repeat units of a 1–6 bp motif). Tabulate per-variant context labels, then compute the homopolymer and STR fractions for each variant set. Rationale: SMS technologies (PacBio, ONT) have known elevated error rates in these contexts, so enrichment of undetected variants in homopolymers/STRs is diagnostic of technology-specific systematic failure.

## Related tools

- **Clairvoyante** (Deep neural network variant caller whose per-technology undetected-variant sets (Illumina, PacBio, ONT) are the subject of the context annotation) — https://github.com/aquaskyline/Clairvoyante
- **samtools/pysam faidx** (Indexed extraction of downstream reference flanking sequence at each variant position from the GRCh37/GRCh38 FASTA)

## Examples

```
python annotate_homopolymer_str.py --variants undetected_all3.vcf --reference GRCh37.fa --out homopolymer_str_classification.csv; # flanks via pysam.FastaFile(pysam.faidx), homopolymer: r'(.)\1{3,}', STR: (1-6bp motif)x>=3
```

## Evaluation signals

- Computed homopolymer fraction for the 12,331 all-technology-undetected set reproduces the reported 26.67%
- Computed STR fraction for the same set reproduces the reported 29.45%
- For the 178,331 PacBio+ONT-undetected set, fractions match the reported 57.67% homopolymer / 18.54% STR
- Per-variant labels satisfy the stated definitions: homopolymer = >=4 identical consecutive bases; STR = >=3 repeat units of a 1–6 bp motif
- Output file homopolymer_str_classification.csv contains one annotated row per input variant plus the fraction summary

## Limitations

- Context labels are derived from the reference flanking sequence, so results depend on using the reference build matching the truth set (GRCh37 vs GRCh38)
- The annotation explains undetected variants only indirectly — enrichment in homopolymers/STRs is consistent with elevated SMS error rates but does not prove per-variant causality
- Thresholds (>=4 bases for homopolymers, >=3 units of 1–6 bp motifs for STRs) are fixed scan parameters; variants just below cutoffs are unlabeled

## Evidence

- [results] Finding: homopolymer/STR enrichment among undetected variants: [evidence span withheld — anchor status: not_found]
- [readme] SMS error rate motivates context analysis: "This task is particularly difficult when dealing with Single Molecule Sequencing, the error rate of which is still tens to hundreds of times higher than Next Generation Sequencing."
- [intro] SMS variant calling challenges: "single nucleotide and small indel variant calling with SMS remain challenging because the traditional variant caller algorithms fail to handle such a high-sequencing error rate"
