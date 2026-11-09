---
name: retrotransposon-consensus-sequence-analysis
description: Use when annotating SVA insertion calls from nanopore data where the
  target family (SVA F1) is missing from Dfam and RepeatMasker reports only a generic
  SVA, requiring a sequence-based thumbprint search.
when_to_use_negative:
- The TE family of interest is already in Dfam and is correctly recovered by the standard
  RepeatMasker-on-Dfam annotation step (no thumbprint search needed)
- Input is short-read data; RetroInspector's reassembly and detection workflow targets
  nanopore long-read sequencing
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3175
tools:
- name: RetroInspector
  role: Pipeline that calls SVA insertions (via cuteSV/Sniffles2), reassembles inserted
    sequences into polished VCFs, and hosts the discrete SVA F1 identification step
    via MAST2 thumbprint check
  repo: https://github.com/javiercguard/retroinspector
- name: BLAST
  role: Queries each candidate SVA insertion sequence against the MAST2 fragment,
    accounting for motif strand, to assign SVA F1 labels
- name: RepeatMasker
  role: Reports the Dfam-based generic SVA annotation that this skill supplements;
    SVA F1 is absent from Dfam so RepeatMasker alone cannot distinguish it
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:6b17c3ea0a59ce900c17bc391ad08bc6402b7739a4ea1c4620df9ca5824388c5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/javiercguard/retroinspector
  tools:
  - BLAST
  - RepeatMasker
  - RetroInspector
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# retrotransposon-consensus-sequence-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects and classifies retrotransposon insertions whose families are absent from Dfam (e.g., SVA F1) by reassembling the inserted sequence from supporting nanopore reads and searching reassembled consensus sequences for family-specific thumbprints, such as the MAST2 fragment carried by SVA F1 elements.

## When to use

Use when annotating SVA insertion calls from nanopore data where the target family (SVA F1) is missing from Dfam and RepeatMasker reports only a generic SVA, requiring a sequence-based thumbprint search.

## When NOT to use

- The TE family of interest is already in Dfam and is correctly recovered by the standard RepeatMasker-on-Dfam annotation step (no thumbprint search needed)
- Input is short-read data; RetroInspector's reassembly and detection workflow targets nanopore long-read sequencing

## Inputs

- Polished VCFs from RetroInspector's consensus reassembly step (e.g., S1.cutesv.polished.vcf.gz) containing reassembled inserted sequences per sample
- Insertion call records from cuteSV/Sniffles2 with at least 3 supporting reads
- Reference MAST2 thumbprint fragment sequence carried by SVA F1 elements

## Outputs

- Bgzipped VCF with insertion records labelled SVA F1 where the reassembled sequence matches the MAST2 fragment
- SVA F1 classification added to the insertion callset records

## How to apply

Run RetroInspector to detect insertions with cuteSV and Sniffles2 (at least 3 supporting reads, the default recommended settings), then take the reassembled inserted sequences from the polished VCFs produced by the consensus reassembly step (e.g., S1.cutesv.polished.vcf.gz). Because SVA F1 is not recovered by the RepeatMasker-on-Dfam annotation step, select insertion sequences consistent with SVA elements for a discrete follow-up search: BLAST each candidate insertion against the MAST2 fragment that SVA F1 elements carry, accounting for the strand on which the motif is reported. Label insertions with a BLAST hit as SVA F1 rather than a generic SVA, and write the labelled callset to a bgzipped VCF consistent with other RetroInspector outputs. Note that reassembly quality depends on read basecalling; HGSVC-era nanopore data basecalled with older basecallers may yield imperfect homopolymer sequences even after reassembly.

## Related tools

- **RetroInspector** (Pipeline that calls SVA insertions (via cuteSV/Sniffles2), reassembles inserted sequences into polished VCFs, and hosts the discrete SVA F1 identification step via MAST2 thumbprint check) — https://github.com/javiercguard/retroinspector
- **BLAST** (Queries each candidate SVA insertion sequence against the MAST2 fragment, accounting for motif strand, to assign SVA F1 labels)
- **RepeatMasker** (Reports the Dfam-based generic SVA annotation that this skill supplements; SVA F1 is absent from Dfam so RepeatMasker alone cannot distinguish it)

## Examples

```
blastn -query sva_insertions.fa -subject MAST2_thumbprint.fa -strand both; then label matched records SVA F1 in the bgzipped VCF
```

## Evaluation signals

- Output bgzipped VCF exists and parses as a valid VCF
- Every record flagged SVA F1 carries a BLAST hit to the MAST2 fragment sequence
- Expert review of a sample of BLAST hits confirms agreement with the MAST2 thumbprint
- Records lacking the MAST2 thumbprint remain labelled as generic SVA rather than SVA F1
- Strand of the reported MAST2 motif is consistent with the BLAST alignment orientation

## Limitations

- SVA F1 is not included in Dfam, so this family is otherwise reported by RepeatMasker only as a generic SVA (normally SVA F) and requires this dedicated thumbprint search
- Nanopore data basecalled with older basecallers may produce imperfect reassembled sequences, particularly for homopolymers, potentially affecting BLAST sensitivity
- The MAST2 thumbprint must be searched on the correct strand; strand must be accounted for when matching

## Evidence
