---
name: variant-record-repair-and-coordinate-filtering
description: Use when per-sample VCF callsets from cuteSV and Sniffles2 contain malformed
  records (negative or 0 coordinates on decoy/scaffold regions) or unfiltered calls
  that must pass a supporting-read threshold before strict/lax merging or benchmarking.
when_to_use_negative:
- Callsets have already passed the supporting-read threshold and coordinate repair,
  and only strict vs lax (intersection vs union) callset selection remains
- Input comes from short-read TE callers (e.g. MELT) rather than cuteSV/Sniffles2
  structural-variant VCFs from nanopore data
- The goal is read-level reassembly quality control rather than record-level filtering
  (that stage has its own 15% length-difference rule)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: RetroInspector
  role: Snakemake pipeline whose configuration exposes the supporting-read threshold
    and performs the filtering, coordinate repair, genotyping-based 0/0 removal, and
    reassembly read exclusion
  repo: https://github.com/javiercguard/retroinspector
- name: cuteSV
  role: Source of structural-variant VCF records that are threshold-filtered and coordinate-repaired
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Second variant caller whose records are merged and then filtered under the
    same supporting-read threshold
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: pysam
  role: Library used to manipulate the VCF/alignment records during filtering and
    repair
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:00430b5c2cba0a368e84cbaf53e7c956f86a19e8651e0102a0a2da5f71e14d59
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
  - RetroInspector
  - Sniffles2
  - cuteSV
  - pysam
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# variant-record-repair-and-coordinate-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clean and threshold-filter raw structural-variant records emitted by cuteSV and Sniffles2 inside the RetroInspector pipeline: apply the user-set supporting-read threshold (default and recommended: 3 reads), repair malformed records with negative or 0 coordinates on decoy/scaffold regions, and drop variants genotyped 0/0 before merging into strict/lax callsets.

## When to use

Use when per-sample VCF callsets from cuteSV and Sniffles2 contain malformed records (negative or 0 coordinates on decoy/scaffold regions) or unfiltered calls that must pass a supporting-read threshold before strict/lax merging or benchmarking.

## When NOT to use

- Callsets have already passed the supporting-read threshold and coordinate repair, and only strict vs lax (intersection vs union) callset selection remains
- Input comes from short-read TE callers (e.g. MELT) rather than cuteSV/Sniffles2 structural-variant VCFs from nanopore data
- The goal is read-level reassembly quality control rather than record-level filtering (that stage has its own 15% length-difference rule)

## Inputs

- Per-sample insertion VCF callsets from cuteSV and Sniffles2 (post intra-sample and inter-sample merging)
- User-configured supporting-read threshold (e.g. config value, default 3)
- Genotype calls per variant (coverage + read support based)
- GRCh38-aligned BAM with minimap2 (for read-support and reassembly checks)

## Outputs

- Cleaned, threshold-filtered per-sample VCF with repaired coordinates on decoy/scaffold regions
- Callset with 0/0 (homozygous-reference) variants removed
- Filtered records feeding the strict (intersection) and lax (union) callset construction

## How to apply

After intra-sample and inter-sample merging of cuteSV and Sniffles2 output, apply the user-configured read-evidence threshold — 3 supporting reads is the default and recommended setting of RetroInspector — to every variant record. Scan for malformed records and repair those carrying negative or 0 coordinates, which arise on decoy/scaffold regions of GRCh38, so they do not corrupt downstream SURVIVOR benchmarking. Once genotyping (coverage- and read-support-based) completes, filter out variants called 0/0 (homozygous for the reference allele) across samples. In the reassembly stage, additionally exclude supporting reads whose inserted sequence differs in length by more than 15% of the reported insertion length, since callers can report greatly differing sequences as support. This filtering is deliberately lightweight — mostly by supporting reads — because the pipeline compensates with read reassembly and hallmark checks rather than heavy statistical filters.

## Related tools

- **RetroInspector** (Snakemake pipeline whose configuration exposes the supporting-read threshold and performs the filtering, coordinate repair, genotyping-based 0/0 removal, and reassembly read exclusion) — https://github.com/javiercguard/retroinspector
- **cuteSV** (Source of structural-variant VCF records that are threshold-filtered and coordinate-repaired) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Second variant caller whose records are merged and then filtered under the same supporting-read threshold) — https://github.com/fritzsedlazeck/Sniffles
- **pysam** (Library used to manipulate the VCF/alignment records during filtering and repair) — https://github.com/pysam-developers/pysam

## Examples

```
snakemake --use-conda -np --config referenceGenome="/home/yourname/hg38.fa" outputPath="/home/yourname/experiment_results" fastq_directory="/home/yourname/data/samples"
```

## Evaluation signals

- Every retained variant record carries at least 3 supporting reads (or the user-configured threshold)
- No retained record has negative or 0 coordinates on decoy/scaffold regions of GRCh38
- Records genotyped 0/0 (homozygous reference) are absent from the final callset
- Reassembled insertion lengths differ from the reported insertion length by no more than 15% for retained supporting reads
- Downstream strict callset reaches the reported precision of 80.90% for TE insertions (85.63% all insertions) with 87.10% recall against the HGSVC truth set via SURVIVOR (100 bp margin)

## Limitations

- HGSVC truth-set data was basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers, even after reassembly
- Filtering relies mostly on supporting-read counts, so it cannot on its own discriminate false insertions with adequate read support
- Supporting reads reported by callers can include greatly differing sequences of inconsistent length, which only the 15% reassembly length rule partially mitigates

## Evidence
