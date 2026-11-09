---
name: primer-sequence-validation-against-published-primers
description: Use when fusion breakpoints (e.g., from a NanoFG VCF) need PCR validation
  primers whose amplicon spans the fusion junction and must reproduce published validation
  primer sequences.
when_to_use_negative:
- Input is raw nanopore FASTQ reads that have not yet been processed by NanoFG — no
  breakpoint coordinates exist to flank with primers
- Starting material is fragmented FFPE DNA, where the article warns the FUDGE/crRNA
  approach and its validation strategy will most likely fail
- The goal is de novo primer design for arbitrary loci with no published primers to
  validate against — this skill specifically checks reproducibility of reported primers
edam_operation: http://edamontology.org/operation_0308
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Primer3
  role: Designs forward/reverse primers spanning the fusion junction from breakpoint-flanking
    GRCh37 sequence
  repo: https://github.com/primer3-org/primer3
- name: NanoFG
  role: Produces the fusion breakpoint VCF that supplies breakpoint coordinates and
    sequences for primer design
  repo: https://github.com/SdeBlank/NanoFG
- name: FUDGE
  role: Upstream enrichment and detection method whose called fusions (with breakpoint-location
    and fusion-spanning primer sequences) this skill reproduces
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:90ffc6a511ff0a7a647a0e0d73ec5ce6114a1e910f616a32be377f7136e15820
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/primer3-org/primer3
  tools:
  - FUDGE
  - NanoFG
  - Primer3
  edam_operation: http://edamontology.org/operation_0308
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# primer-sequence-validation-against-published-primers

## Summary

Designs fusion-junction-spanning PCR primer pairs with Primer3 from breakpoint-flanking reference sequence of fusion genes called by NanoFG, then validates the design by comparing it against published validation primers (e.g., Table 2 SS18 primers).

## When to use

Use when fusion breakpoints (e.g., from a NanoFG VCF) need PCR validation primers whose amplicon spans the fusion junction and must reproduce published validation primer sequences.

## When NOT to use

- Input is raw nanopore FASTQ reads that have not yet been processed by NanoFG — no breakpoint coordinates exist to flank with primers
- Starting material is fragmented FFPE DNA, where the article warns the FUDGE/crRNA approach and its validation strategy will most likely fail
- The goal is de novo primer design for arbitrary loci with no published primers to validate against — this skill specifically checks reproducibility of reported primers

## Inputs

- NanoFG fusion breakpoint VCF (breakpoint coordinates of called fusions)
- GRCh37 human reference genome (FASTA) for extracting breakpoint-flanking sequence
- Published Table 2 validation primer sequences for comparison (e.g., SS18 forward CTGCTTGCCTCAACTAGAAAC)

## Outputs

- primer_pairs.tsv with primer pair IDs, forward/reverse sequences, coordinates, and amplicon size
- Per-fusion junction-spanning amplicon design (200-400 bp target window)

## How to apply

Parse the NanoFG-produced fusion breakpoint VCF to obtain breakpoint coordinates of each called fusion, then extract 200-400 bp of GRCh37 reference sequence flanking each breakpoint as the amplicon target window. Run Primer3 (primer3_core) on the breakpoint-flanking sequence to design forward and reverse primers whose amplicon spans the fusion junction, writing primer IDs, sequences, coordinates, and amplicon sizes to primer_pairs.tsv. Finally, compare the designed primer sequences against the published Table 2 validation primers (e.g., SS18 forward CTGCTTGCCTCAACTAGAAAC) to confirm the in-silico design reproduces the reported primers. This mirrors how the FUDGE/NanoFG paper's methods section lists Primer3 as the primer-design tool for fusion validation.

## Related tools

- **Primer3** (Designs forward/reverse primers spanning the fusion junction from breakpoint-flanking GRCh37 sequence) — https://github.com/primer3-org/primer3
- **NanoFG** (Produces the fusion breakpoint VCF that supplies breakpoint coordinates and sequences for primer design) — https://github.com/SdeBlank/NanoFG
- **FUDGE** (Upstream enrichment and detection method whose called fusions (with breakpoint-location and fusion-spanning primer sequences) this skill reproduces)

## Examples

```
git clone https://github.com/primer3-org/primer3.git primer3 && cd primer3/src && make && ./primer3_core ../example
```

## Evaluation signals

- primer_pairs.tsv exists, is well-formed, and contains columns for primer IDs, sequences, coordinates, and amplicon size
- Designed primer sequences exactly match the published Table 2 entries for the corresponding fusion (e.g., SS18 forward CTGCTTGCCTCAACTAGAAAC)
- Each amplicon spans the reported fusion junction and falls within the 200-400 bp target window
- Each primer pair corresponds one-to-one to a fusion breakpoint present in the NanoFG VCF

## Limitations

- Validation is anchored to a specific reference build (GRCh37); using a different assembly shifts breakpoint coordinates and primer binding sites
- Reproducing published primers only works for the recurrent fusion genes covered by the paper (about ten targets); arbitrary novel fusions yield unvalidated designs
- Fusions detected with only one fusion-spanning read required manual breakpoint PCR validation, so low-support calls may need extra scrutiny before primer design
- The FUDGE assay requires non-fragmented DNA; FFPE-derived material will most likely fail comprehensive fusion identification and downstream validation

## Evidence

- [methods] NanoFG's reporting step provides fusion-spanning primer sequences for validation: "Primer3 - https://github.com/primer3-org/primer3"
- [readme] Primer3 is run via primer3_core on an input sequence file after building from source: [evidence span withheld — anchor status: external_doc]
- [readme] Primer3 installation method grounding the example invocation: [evidence span withheld — anchor status: external_doc]
- [discussion] Two fusions detected with a single supporting read required manual breakpoint PCR validation, motivating primer design: "two fusions were only detected with one fusion-spanning read each, requiring the manual validation of the fusion gene by breakpoint PCR."
