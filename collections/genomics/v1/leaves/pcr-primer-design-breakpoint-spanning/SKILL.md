---
name: pcr-primer-design-breakpoint-spanning
description: Use when a NanoFG (or similar SV-caller) breakpoint VCF contains a called
  fusion gene whose junction must be validated by breakpoint PCR, and flanking reference
  sequence is available from GRCh37.
when_to_use_negative:
- Input is raw nanopore FASTQ/BAM that has not yet been through NanoFG fusion calling
  — no breakpoint coordinates exist to design around.
- Starting material is fragmented FFPE DNA, which the article notes will most likely
  fail FUDGE/NanoFG fusion identification and thus cannot supply reliable breakpoints
  for primer design.
- The goal is de novo assembly or SV discovery rather than validating an already-called
  fusion breakpoint.
edam_operation: http://edamontology.org/operation_0308
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Primer3
  role: Designs the forward/reverse PCR primers on the breakpoint-flanking reference
    sequence so the amplicon spans the fusion junction
  repo: https://github.com/primer3-org/primer3
- name: NanoFG
  role: Produces the fusion breakpoint VCF and per-fusion breakpoint-location/breakpoint-sequence
    output that supplies the primer-design input coordinates
  repo: https://github.com/SdeBlank/NanoFG
- name: NanoSV
  role: Upstream SV caller (v1.2.4, default min_mapq=12, depth_support=False, mapq_flag=48)
    whose filtered VCF defines the breakpoint coordinates used for primer design
  repo: https://github.com/mroosmalen/nanosv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:ed5b123a2abecde64a3fa4407807c6d00299cf588111af78b8e4c8afc35c6f2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/primer3-org/primer3
  tools:
  - FUDGE
  - NanoFG
  - NanoSV
  - Primer3
  edam_operation: http://edamontology.org/operation_0308
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pcr-primer-design-breakpoint-spanning

## Summary

Designs PCR primer pairs that span a detected fusion gene breakpoint by running Primer3 on reference sequence flanking the NanoFG-called breakpoint (GRCh37), enabling experimental validation of fusion calls.

## When to use

Use when a NanoFG (or similar SV-caller) breakpoint VCF contains a called fusion gene whose junction must be validated by breakpoint PCR, and flanking reference sequence is available from GRCh37.

## When NOT to use

- Input is raw nanopore FASTQ/BAM that has not yet been through NanoFG fusion calling — no breakpoint coordinates exist to design around.
- Starting material is fragmented FFPE DNA, which the article notes will most likely fail FUDGE/NanoFG fusion identification and thus cannot supply reliable breakpoints for primer design.
- The goal is de novo assembly or SV discovery rather than validating an already-called fusion breakpoint.

## Inputs

- Fusion breakpoint VCF from NanoFG (NanoSV/Sniffles SV calls annotated with fusion genes)
- GRCh37 human reference genome (FASTA) for extracting breakpoint-flanking sequence
- Fusion gene configuration and breakpoint-sequence fields from the NanoFG reporting step

## Outputs

- primer_pairs.tsv containing primer pair IDs, forward/reverse sequences, genomic coordinates, and amplicon size
- Fusion-spanning primer sequences intended for PCR validation of the fusion junction

## How to apply

Parse the fusion breakpoint VCF produced by NanoFG (which itself runs NanoSV v1.2.4 or Sniffles v1.0.9 with default parameters) to obtain the exact breakpoint coordinates of the called fusion. Extract 200-400 bp of reference genomic sequence flanking the breakpoint from the GRCh37 human reference so the eventual amplicon spans the fusion junction. Run Primer3 (primer3_core) on this breakpoint-flanking sequence to design forward and reverse primers. Write the resulting primer pairs (IDs, sequences, coordinates, amplicon size) to primer_pairs.tsv. Rationale: the article reports that fusions detected with fewer than two fusion-spanning reads require manual validation by breakpoint PCR, and Primer3 is the designated design tool in the NanoFG workflow.

## Related tools

- **Primer3** (Designs the forward/reverse PCR primers on the breakpoint-flanking reference sequence so the amplicon spans the fusion junction) — https://github.com/primer3-org/primer3
- **NanoFG** (Produces the fusion breakpoint VCF and per-fusion breakpoint-location/breakpoint-sequence output that supplies the primer-design input coordinates) — https://github.com/SdeBlank/NanoFG
- **NanoSV** (Upstream SV caller (v1.2.4, default min_mapq=12, depth_support=False, mapq_flag=48) whose filtered VCF defines the breakpoint coordinates used for primer design) — https://github.com/mroosmalen/nanosv

## Examples

```
git clone https://github.com/primer3-org/primer3.git primer3; cd primer3/src && make; ./primer3_core breakpoint_flank.fa > primer3_out.txt
```

## Evaluation signals

- primer_pairs.tsv exists and is well-formed (IDs, sequences, coordinates, amplicon size present for each pair).
- Designed primer sequences match the published Table 2 entries for the corresponding fusion (e.g., SS18 forward CTGCTTGCCTCAACTAGAAAC).
- Each amplicon spans the fusion junction: forward and reverse primers bind on opposite sides of the reported breakpoint coordinates.
- Primer target window is within the 200-400 bp flanking the breakpoint extracted from GRCh37.
- Every fusion call flagged for validation (e.g., those supported by fewer than two fusion-spanning reads) has at least one designed primer pair.

## Limitations

- Fusions detected with only one supporting read require this manual breakpoint PCR validation; the pipeline's ≥2 fusion-spanning read cutoff is intended to avoid it.
- Primer design depends on accurate breakpoint coordinates; FFPE-derived or fragmented DNA yields unreliable calls that undermine primer placement.
- Whole-genome-amplified low-input DNA is only useful for breakpoint identification with prior knowledge of both fusion partners.

## Evidence

- [methods] Primer3 listed as a NanoFG-integrated tool in the methods, with its repository given as the installation source: "Primer3 - https://github.com/primer3-org/primer3"
- [discussion] The discussion explains why breakpoint PCR validation (and hence primer design) is needed for low-support fusion calls: "two fusions were only detected with one fusion-spanning read each, requiring the manual validation of the fusion gene by breakpoint PCR."
- [readme] The README documents how to obtain and set up NanoFG, whose reporting step supplies the breakpoint output for primer design: "git clone https://github.com/SdeBlank/NanoFG.git NanoFG"
- [readme] Primer3 README documents the actual executable used to run primer design on an input sequence file: [evidence span withheld — anchor status: external_doc]
