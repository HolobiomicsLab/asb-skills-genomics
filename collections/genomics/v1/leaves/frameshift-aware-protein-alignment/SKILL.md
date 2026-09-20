---
name: frameshift-aware-protein-alignment
description: Use when screening error-prone long reads (ONT/PacBio metagenomic FASTA/FASTQ)
  for ARG-carrying reads via protein reference alignment, where frameshifts from sequencing
  errors would break standard six-frame translation alignments.
when_to_use_negative:
- Input is short-read (Illumina) data — the frameshift-aware long-read screening layer
  targets Nanopore/PacBio error profiles; the SARG+ README points short-read profiling
  to Lynx instead
- Goal is de novo identification of ARG genomic contexts in understudied environments
  — the article notes assembly-based/binning-based methods may be better suited there
- Detection of point-mutation-derived ARGs is required — read-based profiling cannot
  reliably distinguish SNPs from sequencing errors and such ARGs are excluded from
  the SARG+ database
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0080
tools:
- name: DIAMOND
  role: Performs frameshift-aware DNA-to-protein (blastx) alignment of long reads
    against the SARG+ protein database to flag ARG-containing reads
  repo: https://github.com/xinehc/argo
- name: Argo
  role: Wraps the DIAMOND blastx screening step with an adaptive identity cutoff and
    emits per-read ARG annotations
  repo: https://github.com/xinehc/argo
- name: minimap2
  role: Provides approximate read overlaps (-x ava-ont) whose dv tags set the adaptive
    identity cutoff applied to DIAMOND hits
  repo: https://github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:e9b98dae22cb69e670464fb5c9a312a41d04fcada7c9411de69c3970ecbbf027
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - DIAMOND
  - minimap2
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# frameshift-aware-protein-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align noisy long reads against a protein ARG database using DIAMOND's frameshift-aware blastx mode (e.g. --frameshift 15) so that sequencing-induced indels in Nanopore/PacBio reads do not abort gene hits, forming the ARG-flagging layer of Argo's screening step.

## When to use

Use when screening error-prone long reads (ONT/PacBio metagenomic FASTA/FASTQ) for ARG-carrying reads via protein reference alignment, where frameshifts from sequencing errors would break standard six-frame translation alignments.

## When NOT to use

- Input is short-read (Illumina) data — the frameshift-aware long-read screening layer targets Nanopore/PacBio error profiles; the SARG+ README points short-read profiling to Lynx instead
- Goal is de novo identification of ARG genomic contexts in understudied environments — the article notes assembly-based/binning-based methods may be better suited there
- Detection of point-mutation-derived ARGs is required — read-based profiling cannot reliably distinguish SNPs from sequencing errors and such ARGs are excluded from the SARG+ database

## Inputs

- Quality-controlled long-read metagenomic FASTA/FASTQ (e.g. ONT reads, example.fa.gz from Zenodo record 12571849)
- DIAMOND-indexed SARG+ protein database (database/sarg.dmnd built via diamond makedb from Zenodo record 20451539 database.tar.gz)

## Outputs

- Per-read ARG annotation JSON (e.g. example.sarg.) with 'remark', 'hit', 'plasmid', and 'lineage' fields
- Set of ARG-containing reads with type|subtype|reference identifiers and alignment coordinates

## How to apply

First build the protein database with 'diamond makedb --in database/sarg.fa --db database/sarg' from the Argo SARG+ bundle. Then run DIAMOND blastx of the QC'd long reads against the SARG+ protein database with the frameshift-aware settings used by Argo: --evalue 1e-15, --range-culling, --frameshift 15, --range-cover 25, --max-target-seqs 25, --max-hps 0 (recorded as --max-hsps 0 in the workflow). The frameshift penalty allows a single alignment HSP to bridge indel errors in the read rather than splitting the hit, which is why Argo relies on it before the identity cutoff (90 - 2.5 * 100 * median dv, e.g. 77.45 for the example data) is applied. For each flagged read, record its ARG hits as type|subtype|reference-sequence identifiers with coordinates, plus plasmid flag and lineage, into a per-read JSON structure (e.g. example.sarg.).

## Related tools

- **DIAMOND** (Performs frameshift-aware DNA-to-protein (blastx) alignment of long reads against the SARG+ protein database to flag ARG-containing reads) — https://github.com/xinehc/argo
- **Argo** (Wraps the DIAMOND blastx screening step with an adaptive identity cutoff and emits per-read ARG annotations) — https://github.com/xinehc/argo
- **minimap2** (Provides approximate read overlaps (-x ava-ont) whose dv tags set the adaptive identity cutoff applied to DIAMOND hits) — https://github.com/xinehc/argo

## Examples

```
diamond blastx --db database/sarg --query example.fa --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0
```

## Evaluation signals

- The example run reports 'candidate HSPs: 10000 | ARG-containing reads: 579' at identity cutoff 77.45
- The adaptive cutoff equals 90 - 2.5 * 100 * median(dv) and matches 77.45 for a median overlap divergence of 0.0502
- The emitted example.sarg. file parses as JSON and every per-read record contains 'remark', 'hit', 'plasmid', and 'lineage' fields
- diamond makedb completed on database/sarg.fa without error before the blastx run
- Flagged reads carry type|subtype|reference-sequence identifiers with alignment coordinates

## Limitations

- Argo is reference-based and may misclassify unseen species or novel plasmid syntenies
- Point-mutation-derived ARGs are excluded because SNPs cannot be distinguished from long-read sequencing errors
- Highly similar ARGs are clustered (e.g. blaOXA-1 and blaOXA-1042 as blaOXA), limiting ORF-based annotation and sequence typing resolution

## Evidence

- [methods] Run DIAMOND's frameshift-aware DNA-to-protein alignment: [evidence span withheld — anchor status: not_found]
- [methods] Validation of screening output: [evidence span withheld — anchor status: not_found]
