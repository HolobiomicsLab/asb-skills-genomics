---
name: primer3-amplicon-parameter-configuration
description: Use when a fusion gene has been called (e.g., a fusion breakpoint VCF
  from NanoFG/FUDGE) and PCR primers spanning the fusion junction are needed for validation.
when_to_use_negative:
- Input DNA is fragmented/FFPE material, where the FUDGE/NanoFG assay most likely
  fails to comprehensively identify fusion genes and thus provides no reliable breakpoints
  to design against
- The structural variant has not been flagged as a candidate fusion (both SV ends
  must be in different genes and on the same strand with a continuous transcript)
- Only one fusion-spanning read supports the call and the plan is no-validation reporting
  (NanoFG's cutoff requires at least two fusion-spanning reads for validation-free
  detection)
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Primer3
  role: Primer3_core designs the forward/reverse primer pairs on the breakpoint-flanking
    GRCh37 sequence; installed via git clone and built from primer3/src
  repo: https://github.com/primer3-org/primer3
- name: NanoFG
  role: Upstream pipeline whose fusion breakpoint VCF output supplies the breakpoint
    coordinates and sequences used as Primer3 input
  repo: https://github.com/SdeBlank/NanoFG
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:ef7cfe6469c2040786900e1ecdd4aefb3153f3505c504e31220896765c613d30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/primer3-org/primer3
  tools:
  - FUDGE
  - NanoFG
  - Primer3
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# primer3-amplicon-parameter-configuration

## Summary

Designs breakpoint-spanning PCR primer pairs with Primer3 for validation of fusion genes called by the NanoFG pipeline, extracting 200-400 bp of GRCh37 reference sequence flanking each fusion breakpoint and writing designed primers to primer_pairs.tsv.

## When to use

Use when a fusion gene has been called (e.g., a fusion breakpoint VCF from NanoFG/FUDGE) and PCR primers spanning the fusion junction are needed for validation.

## When NOT to use

- Input DNA is fragmented/FFPE material, where the FUDGE/NanoFG assay most likely fails to comprehensively identify fusion genes and thus provides no reliable breakpoints to design against
- The structural variant has not been flagged as a candidate fusion (both SV ends must be in different genes and on the same strand with a continuous transcript)
- Only one fusion-spanning read supports the call and the plan is no-validation reporting (NanoFG's cutoff requires at least two fusion-spanning reads for validation-free detection)

## Inputs

- Fusion breakpoint VCF produced by the NanoFG pipeline (NanoSV or Sniffles SV calls)
- GRCh37 human reference genome sequence (for breakpoint-flanking sequence extraction)

## Outputs

- primer_pairs.tsv containing primer pair IDs, forward/reverse sequences, genomic coordinates, and amplicon size

## How to apply

Parse the NanoFG-produced fusion breakpoint VCF to obtain the breakpoint coordinates of each called fusion. Extract the reference genomic sequence flanking the breakpoint from GRCh37, targeting a 200-400 bp amplicon window across the fusion junction. Run Primer3 (primer3_core) on the breakpoint-flanking sequence to design forward and reverse primers whose amplicon spans the fusion junction, then write the primer pairs (IDs, sequences, coordinates, amplicon size) to primer_pairs.tsv. As a validation step, compare designed primer sequences against published Table 2 primer sequences (e.g., SS18 forward CTGCTTGCCTCAACTAGAAAC) to confirm the design reproduces the reported validation primers.

## Related tools

- **Primer3** (Primer3_core designs the forward/reverse primer pairs on the breakpoint-flanking GRCh37 sequence; installed via git clone and built from primer3/src) — https://github.com/primer3-org/primer3
- **NanoFG** (Upstream pipeline whose fusion breakpoint VCF output supplies the breakpoint coordinates and sequences used as Primer3 input) — https://github.com/SdeBlank/NanoFG

## Examples

```
git clone https://github.com/primer3-org/primer3.git primer3 && cd primer3/src && make && ./primer3_core ../example
```

## Evaluation signals

- primer_pairs.tsv exists, is well-formed, and contains primer IDs, sequences, coordinates, and amplicon sizes
- Each amplicon spans the fusion junction within the intended 200-400 bp target window
- Designed primer sequences match the published Table 2 entries for the corresponding fusion (e.g., SS18 forward CTGCTTGCCTCAACTAGAAAC)
- Every called fusion gene in the NanoFG output has at least one designed primer pair
- Primer coordinates correctly flank the breakpoint coordinates parsed from the NanoFG VCF

## Limitations

- Validation by breakpoint PCR was only required for fusions detected with a single fusion-spanning read, since NanoFG's cutoff of at least two fusion-spanning reads was considered reliable without further validation
- Breakpoint sequence extraction assumes high-quality, non-fragmented DNA; FFPE material is a known limitation of the upstream assay
- Low-input WGA material is only useful with prior knowledge of both fusion gene partners, restricting primer design scope

## Evidence

- [methods] Primer3 - https://github.com/primer3-org/primer3: "Primer3 - https://github.com/primer3-org/primer3"
- [readme] Design PCR primers from DNA sequence: [evidence span withheld — anchor status: external_doc]
- [discussion] two fusions were only detected with one fusion-spanning read each: "two fusions were only detected with one fusion-spanning read each, requiring the manual validation of the fusion gene by breakpoint PCR."
