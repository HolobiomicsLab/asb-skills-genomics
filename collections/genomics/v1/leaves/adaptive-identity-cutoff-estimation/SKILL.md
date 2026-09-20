---
name: adaptive-identity-cutoff-estimation
description: Use when setting up Argo on new quality-controlled ONT/PacBio metagenomic
  reads and needing a platform-adaptive identity cutoff for the SARG+ DIAMOND blastx
  screening step instead of a fixed threshold.
when_to_use_negative:
- Input reads are short reads — Argo (and this cutoff estimation via read-overlap
  divergence) is designed for long-read (ONT/PacBio) metagenomes; use a short-read
  profiler like Lynx instead
- A cutoff from a previous run or platform is already known and the read set was sequenced
  on the same platform — re-estimation is unnecessary
- The goal is de novo identification of ARG genomic contexts in understudied environments
  — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) approaches
  are better suited
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3693
tools:
- name: Argo
  role: Pipeline whose screening layer consumes the adaptive identity cutoff to flag
    ARG-carrying long reads
  repo: github.com/xinehc/argo
- name: minimap2
  role: Performs approximate read-overlap mapping ('-x ava-ont') yielding 'dv' divergence
    tags and indexes nucleotide files with '-x map-ont -d'
- name: DIAMOND
  role: Frameshift-aware blastx alignment of reads against SARG+ proteins, filtered
    at the adaptive cutoff to identify ARG-containing reads
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:0932e4034c54956ac9959b8883c1e96a75f21cc625ab7a52a563117926df54e8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  tools:
  - Argo
  - DIAMOND
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3693
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# adaptive-identity-cutoff-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates Argo's screening identity cutoff from the median per-base sequence divergence ('dv' tags) of overlaps among the first 10,000 input long reads, computed via minimap2 approximate mapping, so that ARG calling thresholds remain comparable across Nanopore and PacBio sequencing platforms.

## When to use

Use when setting up Argo on new quality-controlled ONT/PacBio metagenomic reads and needing a platform-adaptive identity cutoff for the SARG+ DIAMOND blastx screening step instead of a fixed threshold.

## When NOT to use

- Input reads are short reads — Argo (and this cutoff estimation via read-overlap divergence) is designed for long-read (ONT/PacBio) metagenomes; use a short-read profiler like Lynx instead
- A cutoff from a previous run or platform is already known and the read set was sequenced on the same platform — re-estimation is unnecessary
- The goal is de novo identification of ARG genomic contexts in understudied environments — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) approaches are better suited

## Inputs

- QC'd long-read FASTA (Nanopore or PacBio, e.g. Zenodo record 12571849 example.fa.gz)
- Indexed SARG+ protein database (diamond makedb on database/sarg.fa from Zenodo record 20451539 database.tar.gz)
- minimap2 read-overlap PAF with 'dv' tags from '-x ava-ont' mapping

## Outputs

- Adaptive identity cutoff value (e.g. 77.45 for the example dataset)
- Per-read annotation structure (example.sarg.) for ARG-containing reads with 'remark', 'hit', 'plasmid', and 'lineage' fields

## How to apply

Overlap the input reads with minimap2's approximate mapping mode ('-x ava-ont') to collect 'dv' (approximate per-base sequence divergence) tags, then compute the median divergence from the overlaps of the first 10,000 reads. Set the adaptive identity cutoff as 90 - 2.5 * 100 * median(dv); for Argo's Zenodo example.fa.gz this yields 77.45 from a median divergence of 0.0502. Use this cutoff when filtering DIAMOND's frameshift-aware blastx alignments (evalue 1e-15, --range-culling, --frameshift 15, --range-cover 25, --max-target-seqs 25, --max-hsps 0) of reads against the indexed SARG+ protein database to flag ARG-containing reads. The rationale is that sequencing error profiles differ between platforms, and anchoring the cutoff to the observed divergence of actual read overlaps keeps ARG profiles comparable across sequencing platforms.

## Related tools

- **Argo** (Pipeline whose screening layer consumes the adaptive identity cutoff to flag ARG-carrying long reads) — github.com/xinehc/argo
- **minimap2** (Performs approximate read-overlap mapping ('-x ava-ont') yielding 'dv' divergence tags and indexes nucleotide files with '-x map-ont -d')
- **DIAMOND** (Frameshift-aware blastx alignment of reads against SARG+ proteins, filtered at the adaptive cutoff to identify ARG-containing reads)

## Examples

```
diamond blastx --db database/sarg --query example.fa --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0
```

## Evaluation signals

- For the Zenodo example.fa.gz, the run must report 'candidate HSPs: 10000 | ARG-containing reads: 579' at identity cutoff 77.45
- The recomputed cutoff must equal 90 - 2.5 * 100 * median(dv) from the first 10,000 reads' overlaps (median divergence 0.0502 → cutoff 77.45)
- The emitted per-read annotation JSON must parse as valid JSON and each record must contain 'remark', 'hit', 'plasmid', and 'lineage' fields
- Only reads with DIAMOND HSPs passing the adaptive identity threshold should be flagged as ARG-containing
- Re-running on reads from a different sequencing platform should yield a different median(dv) and hence a different cutoff, confirming platform adaptivity

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Read-based profiling cannot reliably distinguish SNPs from sequencing errors, so point-mutation-derived ARGs are excluded from the SARG+ database
- With insufficient coverage, read overlaps and clustering degrade, and the divergence estimate underlying the cutoff may become unstable

## Evidence

- [methods] adaptive cutoff from overlap divergence: [evidence span withheld — anchor status: not_found]
- [methods] blastx parameters: "blastx --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0"
- [readme] Argo input/output contract: "Argo takes *quality-controlled* long reads (either Nanopore or PacBio) as input and returns a table listing predicted ARGs (types and subtypes), their potential hosts"
- [discussion] reference-based limitation: "Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies."
