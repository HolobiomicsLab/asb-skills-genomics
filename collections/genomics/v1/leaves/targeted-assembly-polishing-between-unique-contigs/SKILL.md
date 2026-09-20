---
name: targeted-assembly-polishing-between-unique-contigs
description: Use when a HASLR draft hybrid assembly shows elevated mismatch/indel
  error rates versus other assemblers and you want parity via polishing, without polishing
  the entire assembly.
when_to_use_negative:
- When the assembler already includes a polishing step and its mismatch/indel rates
  are at parity — the deficiency this skill targets is specific to HASLR's lack of
  a polishing module
- When the errors of interest are misassemblies, genome fraction, or NGA50 rather
  than small local mismatch/indel errors — polishing fixes small local assembly mistakes,
  not structural issues
- When no corresponding high-quality short reads (e.g. Illumina) are available and
  long-read-only Racon polishing cannot achieve the required correction depth
edam_operation: http://edamontology.org/operation_3834
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: HASLR
  role: Produces the draft hybrid assembly whose SPOA-derived inter-contig regions
    are the polishing targets
  repo: https://github.com/vpc-ccg/haslr
- name: Racon
  role: Long-read-guided consensus polishing of the targeted regions
- name: Pilon
  role: Short-read-guided correction (Illumina) of the targeted regions
- name: minimap2
  role: Maps original long reads back onto the assembly to locate the SPOA-derived
    regions between unique SR contigs
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:0fb1d4087160ad768eb949ef5dd99899a3b09fd0b7e41bb46b2e2886104757dc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/haslr
  repo_url: https://github.com/vpc-ccg/haslr
  tools:
  - HASLR
  - Pilon
  - Racon
  - minimap2
  edam_operation: http://edamontology.org/operation_3834
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Targeted assembly polishing between unique short-read contigs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A single round of targeted polishing (Racon and/or Pilon) applied only to the SPOA-derived regions of a HASLR hybrid assembly that lie between unique short-read contigs, to bring HASLR's elevated mismatch and indel rates (per 100 kbp) to parity with other assemblers.

## When to use

Use when a HASLR draft hybrid assembly shows elevated mismatch/indel error rates versus other assemblers and you want parity via polishing, without polishing the entire assembly.

## When NOT to use

- When the assembler already includes a polishing step and its mismatch/indel rates are at parity — the deficiency this skill targets is specific to HASLR's lack of a polishing module
- When the errors of interest are misassemblies, genome fraction, or NGA50 rather than small local mismatch/indel errors — polishing fixes small local assembly mistakes, not structural issues
- When no corresponding high-quality short reads (e.g. Illumina) are available and long-read-only Racon polishing cannot achieve the required correction depth

## Inputs

- HASLR draft assembly (asm.final.fa) produced with default parameters (25x long-read coverage, --aln-block 500, --aln-sim 0.85, --edge-sup 3, Minia k=49, abundance 3)
- Original long reads (e.g. E. coli Nanopore ultra-long reads from Loman lab; yeast PacBio ERX1725434/ERX1725435/ERX1725441)
- Corresponding Illumina short reads (E. coli MG1655 MiSeq; yeast ERX1943903) for Pilon correction
- Pre-filtered short-read contigs >=250 bp without overlaps (sr_k49_a3.contigs.nooverlap.250.fa)
- Reference genome for error-rate evaluation

## Outputs

- Polished assembly containing corrected sequences for the SPOA-derived inter-contig regions only
- Mismatch and indel rates per 100 kbp for polished vs. unpolished assemblies (polishing_error_rates.csv)

## How to apply

Assemble with haslr.py using default parameters (long-read coverage 25x, --aln-block 500, --aln-sim 0.85, --edge-sup 3, Minia k=49, abundance 3) to produce asm.final.fa. Map the original long reads back onto the assembly with minimap2 to identify the SPOA-derived regions lying between unique short-read contigs (short-read contigs pre-filtered to >=250 bp without overlaps, sr_k49_a3.contigs.nooverlap.250.fa, with a minimum alignment block of 500). Run one round of targeted polishing on only those inter-contig regions using Racon (long-read-guided consensus) and/or Pilon (short-read-guided correction), per the paper's note that a polishing module could be designed to only polish the regions between unique SR contigs. Re-evaluate against the reference by computing mismatch and indel rates per 100 kbp for polished versus unpolished assemblies and record them in polishing_error_rates.csv. Acceptance: after a single round, per-100-kbp mismatch and indel rates match those of other assemblers (Table S4 pattern); the residual deficiency to watch is HASLR's higher indel rate.

## Related tools

- **HASLR** (Produces the draft hybrid assembly whose SPOA-derived inter-contig regions are the polishing targets) — https://github.com/vpc-ccg/haslr
- **Racon** (Long-read-guided consensus polishing of the targeted regions)
- **Pilon** (Short-read-guided correction (Illumina) of the targeted regions)
- **minimap2** (Maps original long reads back onto the assembly to locate the SPOA-derived regions between unique SR contigs) — https://github.com/lh3/minimap2

## Examples

```
racon -m 8 -x -6 -g -8 -u 0 -t 16 lr25x.fasta lr_overlaps.paf asm.final.fa > polished_regions.fa
```

## Evaluation signals

- After one polishing round, per-100-kbp mismatch rate of the HASLR assembly matches other assemblers in the dataset (as reported in Table S4)
- After one polishing round, per-100-kbp indel rate of the HASLR assembly matches other assemblers (indel rate is HASLR's residual deficiency to monitor)
- Polished regions correspond exactly to SPOA-derived intervals between unique short-read contigs, not the whole assembly (compare polished sequence names/coordinates)
- Values recorded in polishing_error_rates.csv for both polished and unpolished assemblies allow a direct before/after comparison

## Limitations

- HASLR ships no polishing module in its current version — this skill supplies the missing step externally via Racon/Pilon
- HASLR usually retains a higher indel rate compared with other tools even after the analysis is framed around polishing
- The backbone graph's connectivity depends on unique SR contigs; regions lacking unique SR contigs cannot be delimited for targeted polishing

## Evidence
