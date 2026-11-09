---
name: te-insertion-sequence-classification
description: Use when a nanopore TE-insertion callset contains SVA calls that must
  be resolved to the newer SVA F1 family, which Dfam lacks and RepeatMasker annotates
  only as generic SVA, via sequence evidence (MAST2 thumbprint).
when_to_use_negative:
- When classifying TE families that ARE represented in Dfam — those are recovered
  by the pipeline's RepeatMasker-on-Dfam step, not by this MAST2-thumbprint search
- 'When input is short-read data: RetroInspector is built for nanopore sequencing,
  and the reassembled insertion sequences this filter consumes come from long-read-supported
  reassembly'
- For non-SVA insertions (L1, Alu, etc.) that lack the MAST2 fragment — the search
  is specific to SVA-called records
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0084
tools:
- name: BLAST
  role: Sequence-search engine used to query each candidate SVA insertion sequence
    against the MAST2 fragment, strand-aware, to assign the SVA F1 label
- name: RetroInspector
  role: Pipeline hosting this discrete SVA F1 identification step; consumes the polished
    VCFs from its consensus reassembly stage and writes the labelled bgzipped VCF
  repo: https://github.com/javiercguard/retroinspector
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:5f1bdf436f956da77ac90c0e0220b0db9ac7a1a3d1f893abcbb479c8dcfb9745
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# te-insertion-sequence-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A discrete RetroInspector step that reclassifies called SVA insertions as SVA F1 by BLAST-searching reassembled insertion sequences for the MAST2 thumbprint fragment, since SVA F1 is absent from Dfam and RepeatMasker otherwise reports these elements only as generic SVA (SVA F).

## When to use

Use when a nanopore TE-insertion callset contains SVA calls that must be resolved to the newer SVA F1 family, which Dfam lacks and RepeatMasker annotates only as generic SVA, via sequence evidence (MAST2 thumbprint).

## When NOT to use

- When classifying TE families that ARE represented in Dfam — those are recovered by the pipeline's RepeatMasker-on-Dfam step, not by this MAST2-thumbprint search
- When input is short-read data: RetroInspector is built for nanopore sequencing, and the reassembled insertion sequences this filter consumes come from long-read-supported reassembly
- For non-SVA insertions (L1, Alu, etc.) that lack the MAST2 fragment — the search is specific to SVA-called records

## Inputs

- Polished, bgzipped per-sample VCF from the consensus reassembly step (e.g. S1.cutesv.polished.vcf.gz) with reassembled insertion sequences
- Candidate SVA insertion sequences selected from the reassembled callset
- MAST2 fragment reference sequence (the thumbprint carried by SVA F1 elements)

## Outputs

- Bgzipped VCF whose SVA records are labelled as SVA F1 where the MAST2 thumbprint BLAST hit was found, schema-consistent with other RetroInspector variant outputs

## How to apply

After the consensus reassembly step produces polished per-sample VCFs (e.g. S1.cutesv.polished.vcf.gz), load the reassembled inserted sequences and select those consistent with SVA elements for this follow-up search, because the RepeatMasker-on-Dfam step used for the other TE families does not recover SVA F1. Query each candidate insertion sequence with BLAST against the MAST2 fragment carried by SVA F1 elements, accounting for the strand on which the motif is reported. Insertions with a BLAST match to the MAST2 fragment are labelled SVA F1 instead of generic SVA, and this classification is written back into the insertion callset records as a bgzipped VCF consistent with other RetroInspector variant outputs. Rationale: without this discrete sequence-based step, SVA F1 elements are silently collapsed into the generic SVA class, losing family-level resolution for the newest SVA branch.

## Related tools

- **BLAST** (Sequence-search engine used to query each candidate SVA insertion sequence against the MAST2 fragment, strand-aware, to assign the SVA F1 label)
- **RetroInspector** (Pipeline hosting this discrete SVA F1 identification step; consumes the polished VCFs from its consensus reassembly stage and writes the labelled bgzipped VCF) — https://github.com/javiercguard/retroinspector

## Examples

```
blastn -query S1.insertions.fasta -subject MAST2_fragment.fasta -strand both -outfmt 6
```

## Evaluation signals

- Output VCF exists, parses as a valid VCF, and is bgzipped like other RetroInspector variant outputs
- Every record flagged SVA F1 carries a BLAST hit to the MAST2 fragment sequence (expert review of a sample of hits to confirm MAST2-thumbprint agreement)
- Strand of each reported BLAST hit is consistent with the strand on which the MAST2 motif is carried by the insertion
- SVA-F1-labelled records form a subset of the SVA insertions called in the polished VCF — no new insertions are introduced by this step
- Insertions without a MAST2 BLAST match remain labelled as generic SVA rather than being discarded

## Limitations

- SVA F1 is not included in Dfam, so this family is invisible to the pipeline's RepeatMasker-on-Dfam annotation step and only recoverable via the discrete MAST2-thumbprint search
- SVA F1 calls depend entirely on presence/detectability of the MAST2 thumbprint; reassembled sequences may be imperfect — HGSVC truth-set data was basecalled with an older basecaller and 'even after reassembly, sequences may be imperfect, particularly for homopolymers', which can weaken BLAST evidence
- Because the filter keys on a single sequence thumbprint, SVA insertions lacking or divergent at the MAST2 fragment cannot be distinguished from other generic SVA (SVA F) elements

## Evidence
