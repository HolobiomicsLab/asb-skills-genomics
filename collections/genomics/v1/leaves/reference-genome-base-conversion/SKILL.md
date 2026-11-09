---
name: reference-genome-base-conversion
description: 'Use when building the nanoEM pipeline''s first step: base-converted
  long reads (from EMseq/APOBEC treatment) must be mapped against a reference that
  mirrors their C-to-T and G-to-A substitutions.'
when_to_use_negative:
- The downstream alignment target for unconverted reads (e.g. methylation calling
  against 'ref' in sambamba mpileup, or nanopolish-style signal calling) — those steps
  need the original, unconverted reference
- Aligning ordinary, non-base-converted reads where C positions must be scored against
  cytosines directly
- Preparing a bisulfite-style reference for short-read aligners such as bismark —
  this conversion is tailored to the nanoEM minimap2 map-ont workflow
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: pysam
  role: Reads the reference genome FASTA and writes the C-to-T/G-to-A converted sequences
  repo: https://github.com/yos-sk/nanoEM
- name: Python3
  role: Runtime for src/convert_ref.py, the conversion script in the nanoEM repository
  repo: https://github.com/yos-sk/nanoEM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:189211f4c6b2a91ad0d3068b3744ffc9dd29cc93fbaf611ceb72d6474536c264
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yos-sk/nanoEM
  repo_url: https://github.com/yos-sk/nanoEM
  tools:
  - Python3
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# reference-genome-base-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a methylation-conversion-aware reference for the nanoEM pipeline by replacing every C with T and every G with A in a whole-genome FASTA (e.g. GRCh38.p12) using Python3 and pysam, so that C-to-T/G-to-A converted nanopore reads can align to it.

## When to use

Use when building the nanoEM pipeline's first step: base-converted long reads (from EMseq/APOBEC treatment) must be mapped against a reference that mirrors their C-to-T and G-to-A substitutions.

## When NOT to use

- The downstream alignment target for unconverted reads (e.g. methylation calling against 'ref' in sambamba mpileup, or nanopolish-style signal calling) — those steps need the original, unconverted reference
- Aligning ordinary, non-base-converted reads where C positions must be scored against cytosines directly
- Preparing a bisulfite-style reference for short-read aligners such as bismark — this conversion is tailored to the nanoEM minimap2 map-ont workflow

## Inputs

- Reference genome FASTA (e.g. GRCh38.p12), passed as the 'ref' argument
- Python3 environment with the pysam package installed

## Outputs

- output.fa: whole-genome FASTA with all C→T and G→A substitutions, same sequence names and lengths as the original reference

## How to apply

Load the reference genome FASTA with pysam in src/convert_ref.py, substituting the actual reference path (e.g. the GRCh38.p12 FASTA) for the 'ref' placeholder. For every reference sequence, substitute each C with T and each G with A so the reference mirrors the methylation-sensitive base conversions applied to nanoEM reads on both strands. Preserve original sequence names and coordinate space, because CpG-site coordinates (cpg_sites.bed used later by sambamba mpileup) must remain index-compatible with the unconverted genome. Write converted sequences to stdout and redirect to output.fa, which is then consumed by minimap2 (-ax map-ont --eqx) for both the *_CT.fq.gz and *_GA.fq.gz read sets.

## Related tools

- **pysam** (Reads the reference genome FASTA and writes the C-to-T/G-to-A converted sequences) — https://github.com/yos-sk/nanoEM
- **Python3** (Runtime for src/convert_ref.py, the conversion script in the nanoEM repository) — https://github.com/yos-sk/nanoEM

## Examples

```
python src/convert_ref.py ref > output.fa
```

## Evaluation signals

- output.fa exists and is a valid FASTA readable by pysam
- Per-sequence lengths in output.fa exactly match the original GRCh38.p12 reference
- Sequence names are unchanged, so cpg_sites.bed coordinates remain valid for the downstream sambamba mpileup step
- Converted reads (*_CT.fq.gz and *_GA.fq.gz) map to output.fa with minimap2 -ax map-ont --eqx without reference-name mismatches
- Spot-check a known region: every reference C is T and every G is A in output.fa

## Limitations

- No version numbers are stated for Python3 or pysam in the nanoEM README, so exact reproducibility of the conversion environment is unspecified
- The converted reference is only valid for base-converted reads; methylation coordinates are recovered by mapping pileups back against the unconverted 'ref' in the methylation-calling step
- The conversion must be paired with convert_reads.py producing both *_CT.fq.gz and *_GA.fq.gz; converting only one strand breaks the two-BAM (1.sorted.bam / 2.sorted.bam) best-alignment strategy

## Evidence
