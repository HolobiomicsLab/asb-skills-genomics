---
name: blast-sequence-similarity-searching
description: Use when reassembled insertion sequences from a structural-variant callset
  must be screened for a known diagnostic motif (e.g., a transduction thumbprint)
  that standard TE annotation libraries do not model.
when_to_use_negative:
- The goal is general TE family annotation of all repeats, which RetroInspector already
  performs via RepeatMasker against Dfam — the BLAST step is a targeted follow-up
  only for SVA calls.
- The insertion class of interest is already represented in Dfam and recovered by
  the standard annotation step, so no thumbprint search is needed.
- The reference motif is a profile HMM (e.g., Dfam family model) rather than a short
  transduced fragment; use HMMER/RepeatMasker-style search instead.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0157
tools:
- name: BLAST
  role: Sequence similarity search of each candidate SVA insertion sequence against
    the MAST2 thumbprint fragment; a hit labels the insertion as SVA F1
- name: RetroInspector
  role: Upstream Snakemake pipeline producing the reassembled, polished insertion
    VCFs and consuming the SVA F1 labels into its final annotated callset
  repo: https://github.com/javiercguard/retroinspector
- name: RepeatMasker/Dfam
  role: Baseline TE annotation step that assigns generic SVA (normally SVA F) labels;
    SVA F1 is not in Dfam so the BLAST thumbprint check supplements it
  repo: http://www.repeatmasker.org/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:7c67eddb75ad5758610d3ec13422515ed94f0a04d0f330de44407c6a0c02de1d
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
  - RepeatMasker/Dfam
  - RetroInspector
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# blast-sequence-similarity-searching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Use BLAST to identify specific sequence signatures inside reassembled insertion sequences — here, detecting the MAST2 thumbprint fragment carried by SVA F1 retrotransposon elements that Dfam/RepeatMasker annotation misses, labeling generic SVA calls as SVA F1.

## When to use

Use when reassembled insertion sequences from a structural-variant callset must be screened for a known diagnostic motif (e.g., a transduction thumbprint) that standard TE annotation libraries do not model.

## When NOT to use

- The goal is general TE family annotation of all repeats, which RetroInspector already performs via RepeatMasker against Dfam — the BLAST step is a targeted follow-up only for SVA calls.
- The insertion class of interest is already represented in Dfam and recovered by the standard annotation step, so no thumbprint search is needed.
- The reference motif is a profile HMM (e.g., Dfam family model) rather than a short transduced fragment; use HMMER/RepeatMasker-style search instead.

## Inputs

- Reassembled insertion sequences from RetroInspector polished VCFs (e.g., S1.cutesv.polished.vcf.gz)
- SVA insertion callset records (subset selected for follow-up)
- MAST2 thumbprint fragment reference sequence (FASTA)

## Outputs

- SVA F1-labelled insertion callset records
- bgzipped VCF with SVA F1 annotations consistent with other RetroInspector variant outputs
- BLAST hit list per candidate insertion (hit presence/absence drives the F1 label)

## How to apply

Load the reassembled inserted sequences for each sample from the polished VCFs produced by RetroInspector's consensus reassembly step (e.g., S1.cutesv.polished.vcf.gz). Restrict the search to insertions already classified as SVA, because SVA F1 is absent from Dfam and therefore not recovered by the pipeline's RepeatMasker-on-Dfam annotation step. Query each candidate insertion sequence with BLAST against the MAST2 fragment that SVA F1 elements contain, accounting for the strand on which the motif is reported. Label insertions with a BLAST match to the MAST2 fragment as SVA F1 and write the labelled callset to a bgzipped VCF consistent with the other RetroInspector variant outputs.

## Related tools

- **BLAST** (Sequence similarity search of each candidate SVA insertion sequence against the MAST2 thumbprint fragment; a hit labels the insertion as SVA F1)
- **RetroInspector** (Upstream Snakemake pipeline producing the reassembled, polished insertion VCFs and consuming the SVA F1 labels into its final annotated callset) — https://github.com/javiercguard/retroinspector
- **RepeatMasker/Dfam** (Baseline TE annotation step that assigns generic SVA (normally SVA F) labels; SVA F1 is not in Dfam so the BLAST thumbprint check supplements it) — http://www.repeatmasker.org/

## Examples

```
blastn -query MAST2_thumbprint.fa -subject SVA_insertions.fa -strand both -outfmt 6
```

## Evaluation signals

- Output VCF exists and parses as a valid VCF.
- Every record flagged as SVA F1 carries a BLAST hit to the MAST2 fragment sequence.
- Expert review of a sample of BLAST hits confirms agreement with the MAST2-thumbprint call.
- Output is written as a bgzipped VCF consistent with the other RetroInspector variant outputs.
- Insertions without a MAST2-fragment BLAST match retain their generic SVA label rather than being promoted to SVA F1.

## Limitations

- SVA F1 is absent from Dfam, so this signature-based step is required as a supplement; the approach only covers the F1/MAST2-thumbprint case, not other unmodelled TE families.
- HGSVC nanopore data was basecalled with an older basecaller and, even after reassembly, sequences may be imperfect, particularly for homopolymers, potentially weakening BLAST evidence.
- The motif's strand must be accounted for when interpreting BLAST hits.

## Evidence

- [results] RetroInspector examines SVA insertions for the MAST2's thumbprint25 and, if found, reports them as SVA F1: "RetroInspector examines SVA insertions for the MAST2’s thumbprint25 and, if found, reports them as SVA F1"
- [results] The newer family of SVA elements, SVA F1, is not included in Dfam: "The newer family of SVA elements, SVA F1, is not included in Dfam"
- [results] our tool retrieves reads that support the insertion, and reassembles the affected region in order to obtain a new sequence: "our tool retrieves reads that support the insertion, and reassembles the affected region in order to obtain a new sequence."
- [readme] RetroInspector is a pipeline to detect and annotate TE insertions and deletions on human genomes sequenced by nanopore sequencing: "RetroInspector is a pipeline to detect and annotate TE insertions and deletions on human genomes sequenced by nanopore sequencing."
