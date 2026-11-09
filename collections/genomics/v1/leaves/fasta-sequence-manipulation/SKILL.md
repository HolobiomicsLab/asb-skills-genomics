---
name: fasta-sequence-manipulation
description: Use when reads have undergone methylation-sensitive base conversion (e.g.
  EMseq/APOBEC C-to-T) and must be aligned against a reference that has been identically
  converted, requiring FASTA-wide base substitution.
when_to_use_negative:
- The reference must remain unconverted, e.g. when preparing to call methylation with
  sambamba mpileup against the original 'ref' or when aligning native (non-base-converted)
  nanopore reads with nanopolish
- The task is read-level conversion rather than reference conversion — convert_reads.py
  handles the *_CT.fq.gz and *_GA.fq.gz outputs instead
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: pysam
  role: Python library used in convert_ref.py to load the reference FASTA, perform
    C-to-T/G-to-A substitution per sequence, and write the converted sequences
  repo: https://github.com/yos-sk/nanoEM
- name: minimap2
  role: Subsequent consumer of output.fa, aligning *_CT.fq.gz and *_GA.fq.gz converted
    reads to the converted reference with 'map-ont' and --eqx
- name: nanopore sequencing
  role: Underlying technology whose C-to-T base-converted long reads (PromethION,
    3.4–7.6 kb N50) motivate the reference conversion
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:b1ea1344350feb5d6f32395efb86f7b9c417de86b02da5e66fa1d2fb8acd4f5b
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
  - minimap2
  - nanopore sequencing
  - pysam
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# fasta-sequence-manipulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Programmatically transform nucleotide sequences in a reference-genome FASTA — here, converting every C to T and G to A with pysam so base-converted nanoEM long reads can align to a mirrored reference while preserving sequence names and coordinate space.

## When to use

Use when reads have undergone methylation-sensitive base conversion (e.g. EMseq/APOBEC C-to-T) and must be aligned against a reference that has been identically converted, requiring FASTA-wide base substitution.

## When NOT to use

- The reference must remain unconverted, e.g. when preparing to call methylation with sambamba mpileup against the original 'ref' or when aligning native (non-base-converted) nanopore reads with nanopolish
- The task is read-level conversion rather than reference conversion — convert_reads.py handles the *_CT.fq.gz and *_GA.fq.gz outputs instead

## Inputs

- Reference genome FASTA (e.g. GRCh38.p12), passed as the 'ref' argument to convert_ref.py

## Outputs

- Converted reference FASTA (output.fa) with C->T and G->A substitutions, same sequence names and lengths as the original

## How to apply

Load the reference FASTA (e.g. GRCh38.p12) with pysam in a script such as convert_ref.py, substituting your reference path for the 'ref' placeholder. For every reference sequence, replace each C with T and each G with A so the converted reference mirrors the C-to-T/G-to-A conversions applied to nanoEM reads on both strands. Preserve original sequence names and coordinate space, because the converted reference must remain index-compatible with the unconverted genome for downstream CpG-site coordinates used by call_methylation.py. Write converted sequences to stdout and redirect to output.fa, which then serves as the minimap2 'map-ont' alignment target for *_CT.fq.gz and *_GA.fq.gz reads. Validate that output.fa exists, is FASTA-readable by pysam, and that per-sequence lengths match the original GRCh38.p12 reference.

## Related tools

- **pysam** (Python library used in convert_ref.py to load the reference FASTA, perform C-to-T/G-to-A substitution per sequence, and write the converted sequences) — https://github.com/yos-sk/nanoEM
- **minimap2** (Subsequent consumer of output.fa, aligning *_CT.fq.gz and *_GA.fq.gz converted reads to the converted reference with 'map-ont' and --eqx)
- **nanopore sequencing** (Underlying technology whose C-to-T base-converted long reads (PromethION, 3.4–7.6 kb N50) motivate the reference conversion)

## Examples

```
python src/convert_ref.py ref > output.fa
```

## Evaluation signals

- output.fa exists and is a valid FASTA readable by pysam
- Per-sequence lengths in output.fa match the original GRCh38.p12 reference exactly (no indels introduced)
- Sequence names and ordering are identical to the original reference, keeping downstream CpG-site coordinate lookups (cpg_sites.bed) valid
- No C or G residues remain in the converted sequences (all converted to T or A)
- Downstream alignment of *_CT.fq.gz reads to output.fa with minimap2 produces plausible sorted BAMs (1.sorted.bam, 2.sorted.bam)

## Limitations

- Conversion must preserve the original coordinate space; any rename or reordering breaks index compatibility with the unconverted genome used for CpG-site coordinates
- No software version numbers are stated in the article or README for pysam, minimap2, or the pipeline scripts
- The '--split-prefix' minimap2 option note indicates behavior depends on minimap2 vintage (downloads after April 7, 2023 allow omitting it), though this affects the alignment step rather than conversion itself

## Evidence

- [methods] As the first step of the nanoEM methylation-calling pipeline, every C and G in the human reference genome is converted to T and A, respectively, producing the converted reference used to align base-converted reads.: [evidence span withheld — anchor status: not_found]
- [methods] Preserve the original sequence names and coordinate space, since the converted reference must remain index-compatible with the unconverted genome for downstream CpG-site coordinates.: [evidence span withheld — anchor status: not_found]
