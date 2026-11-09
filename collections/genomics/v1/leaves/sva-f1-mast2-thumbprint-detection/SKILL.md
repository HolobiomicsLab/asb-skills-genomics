---
name: sva-f1-mast2-thumbprint-detection
description: Use when a nanopore TE callset contains insertions annotated as generic
  SVA (SVA F) by the RepeatMasker-on-Dfam step and you need to distinguish the newer
  SVA F1 family, which is absent from Dfam.
when_to_use_negative:
- Insertions belong to other TE families (e.g. L1 or Alu) that are already recovered
  by the RepeatMasker-on-Dfam annotation step — the MAST2 thumbprint check is specific
  to SVA elements
- No reassembled inserted sequence is available (the check operates on the reassembled
  inserted region, not on raw supporting reads alone)
- The goal is detection of SVA insertions generally rather than sub-family resolution
  of already-called SVA insertions
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: RetroInspector
  role: Pipeline whose reassembly and annotation steps supply the SVA insertion sequences
    and polished VCFs; it applies the MAST2 thumbprint check and writes the labelled
    SVA F1 callset
  repo: https://github.com/javiercguard/retroinspector
- name: BLAST
  role: Used to query each candidate SVA insertion sequence against the MAST2 fragment
    that SVA F1 elements contain
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:1a36cea4277f0d231b31ae701907077e21b9f18f5f267e7a02fbf4785ce8ac15
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
  - RetroInspector
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# sva-f1-mast2-thumbprint-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A discrete follow-up step in RetroInspector that resolves SVA insertions not distinguished by Dfam/RepeatMasker: called SVA insertions are screened for the MAST2 thumbprint fragment carried by SVA F1 elements and relabelled SVA F1 when the fragment is found.

## When to use

Use when a nanopore TE callset contains insertions annotated as generic SVA (SVA F) by the RepeatMasker-on-Dfam step and you need to distinguish the newer SVA F1 family, which is absent from Dfam.

## When NOT to use

- Insertions belong to other TE families (e.g. L1 or Alu) that are already recovered by the RepeatMasker-on-Dfam annotation step — the MAST2 thumbprint check is specific to SVA elements
- No reassembled inserted sequence is available (the check operates on the reassembled inserted region, not on raw supporting reads alone)
- The goal is detection of SVA insertions generally rather than sub-family resolution of already-called SVA insertions

## Inputs

- Polished VCF with reassembled insertion sequences from the consensus reassembly step (e.g. S1.cutesv.polished.vcf.gz)
- MAST2 thumbprint fragment sequence (the MAST2 fragment carried by SVA F1 elements)
- SVA insertion records selected from the RetroInspector callset

## Outputs

- bgzipped VCF with SVA F1 labels added to qualifying insertion callset records, consistent with other RetroInspector variant outputs

## How to apply

Load the reassembled inserted sequences for each sample from the polished VCFs produced by the consensus reassembly step (e.g. S1.cutesv.polished.vcf.gz); these sequences come from RetroInspector retrieving reads supporting each insertion and reassembling the affected region. Select insertion records consistent with SVA elements, since SVA F1 is not recovered by the RepeatMasker-on-Dfam step used for other TE families. Query each candidate insertion sequence with BLAST against the MAST2 fragment that SVA F1 elements contain, accounting for the strand on which the motif is reported. Label insertions with a BLAST match to the MAST2 fragment as SVA F1, add this classification to the insertion callset records, and write the labelled callset to a bgzipped VCF consistent with the other RetroInspector variant outputs.

## Related tools

- **RetroInspector** (Pipeline whose reassembly and annotation steps supply the SVA insertion sequences and polished VCFs; it applies the MAST2 thumbprint check and writes the labelled SVA F1 callset) — https://github.com/javiercguard/retroinspector
- **BLAST** (Used to query each candidate SVA insertion sequence against the MAST2 fragment that SVA F1 elements contain)

## Examples

```
blastn -query S1_sva_insertions.fasta -subject MAST2_thumbprint_fragment.fasta -outfmt 6
```

## Evaluation signals

- The labelled callset VCF exists and parses as a valid VCF
- Every record flagged as SVA F1 carries a BLAST hit to the MAST2 fragment sequence, with the strand of the motif accounted for
- SVA insertions without a MAST2 hit retain their generic SVA annotation rather than being mislabelled SVA F1
- Expert review of a sample of hits confirms MAST2-thumbprint agreement
- The output is bgzipped and consistent with the other RetroInspector variant outputs

## Limitations

- SVA F1 is absent from Dfam, so the standard RepeatMasker-on-Dfam step cannot classify it and reports it as a generic SVA (normally SVA F); only this discrete thumbprint step resolves F1
- Reassembly quality bounds detection: supporting reads with insertion length differences greater than 15% of the reported insertion length are excluded, and older-basecalled HGSVC data may yield imperfect reassembled sequences, particularly for homopolymers
- The thumbprint check is a single-fragment BLAST screen, so insertion sequences that are truncated or poorly reassembled may fail to recover the MAST2 motif

## Evidence
