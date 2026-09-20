---
name: long-read-metagenomic-quality-control
description: Use when raw Nanopore or PacBio long-read metagenomic FASTQs must be
  cleaned before ARG profiling (e.g., Argo/SARG+), including removal of yeast-mapped
  and unmapped reads from mock communities.
when_to_use_negative:
- Input is already quality-controlled FASTQ, or consists of short Illumina reads —
  the article's QC chain (Porechop/nanoq) targets long reads, and short reads should
  go through a short-read profiler such as Lynx.
- Read coverage is extremely low (< 1x per genome) — the article notes insufficient
  coverage yields unstable normalization and inadequate read overlaps, making profiling
  unreliable regardless of QC.
edam_operation: http://edamontology.org/operation_3238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Porechop
  role: Adapter trimming of raw Nanopore reads with '--discard_middle' to remove adapter
    sequences before quality filtering
- name: nanoq
  role: Long-read quality/length filtering with '--min-qual 10 --min-len 1000'
- name: Argo
  role: Downstream consumer of the QC'd reads; performs overlap-based read clustering
    and species-resolved ARG profiling on the cleaned FASTQ
  repo: github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:c10a1bbc9d3fb0a661f69a39386c98c84e7422eb641185b2994ad6939f53748b
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
  - Kraken2
  - Melon
  - Porechop
  - R
  - ggplot2
  - nanoq
  - vegan
  edam_operation: http://edamontology.org/operation_3238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# long-read-metagenomic-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control raw Nanopore/PacBio long reads with adapter trimming (Porechop --discard_middle) and length/quality filtering (nanoq --min-qual 10 --min-len 1000) before species-resolved ARG profiling with Argo. Argo's read-overlap clustering depends on clean reads, so this QC step is a prerequisite for reliable cpg abundance estimation.

## When to use

Use when raw Nanopore or PacBio long-read metagenomic FASTQs must be cleaned before ARG profiling (e.g., Argo/SARG+), including removal of yeast-mapped and unmapped reads from mock communities.

## When NOT to use

- Input is already quality-controlled FASTQ, or consists of short Illumina reads — the article's QC chain (Porechop/nanoq) targets long reads, and short reads should go through a short-read profiler such as Lynx.
- Read coverage is extremely low (< 1x per genome) — the article notes insufficient coverage yields unstable normalization and inadequate read overlaps, making profiling unreliable regardless of QC.

## Inputs

- raw long-read metagenomic FASTQ files (Nanopore or PacBio)
- mock-community samples requiring removal of yeast-mapping and unmapped reads
- fecal long-read metagenomic samples (e.g., the 329-sample human/non-human primate dataset)

## Outputs

- quality-controlled FASTQ files (adapters trimmed, Phred >= 10, length >= 1000 bp)
- cleaned read sets suitable as input to Argo ARG profiling

## How to apply

Run Porechop with '--discard_middle' on each raw FASTQ to trim adapters and discard middle adapters, then filter with nanoq using '--min-qual 10 --min-len 1000' to retain reads with Phred >= 10 and at least 1000 bp. For sequenced mock communities, additionally remove reads mapping to yeast and unmapped reads so downstream profiling reflects only the target community. QC'd reads feed directly into Argo, whose read-overlap graph clustering (MCL, inflation=2) requires adequate read overlaps — the rationale is that adapters and low-quality tails would fragment overlaps and destabilize cluster-level species assignment. Judgment of correct application: QC'd samples should reproduce Argo's published type-level Pearson correlations (0.990 D6300, 0.976 MSA2006, 0.981 MSA1003, 0.672 D6331) with type/subtype precision above 0.9.

## Related tools

- **Porechop** (Adapter trimming of raw Nanopore reads with '--discard_middle' to remove adapter sequences before quality filtering)
- **nanoq** (Long-read quality/length filtering with '--min-qual 10 --min-len 1000')
- **Argo** (Downstream consumer of the QC'd reads; performs overlap-based read clustering and species-resolved ARG profiling on the cleaned FASTQ) — github.com/xinehc/argo

## Examples

```
porechop -i raw.fastq --discard_middle -o trimmed.fastq && nanoq -i trimmed.fastq --min-qual 10 --min-len 1000 -o qc.fastq
```

## Evaluation signals

- All retained reads meet the thresholds: quality score >= 10 and length >= 1000 bp after nanoq filtering.
- For sequenced mock communities, no yeast-mapping or unmapped reads remain in the cleaned FASTQ.
- Downstream Argo run on QC'd reads reproduces type-level Pearson correlations of 0.990 (D6300), 0.976 (MSA2006), 0.981 (MSA1003), and 0.672 (D6331).
- Type/subtype precision of Argo outputs exceeds 0.9 for every mock community.
- Reported plasmid-borne ARG calls are recovered (e.g., ant(4')-I, qacG, tet(L) for D6300; aph(2")-I at 4.256 cpg in Enterococcus faecalis for MSA2006).

## Limitations

- Insufficient coverage (below ~1x genome copy) hinders accurate taxonomic classification because read overlaps are inadequate and read clustering less effective, even with correct QC.
- Argo considers ARGs of species below one genome copy (1x) as 'unclassified', so QC alone cannot rescue detection of extremely rare species.
- QC chain is designed for long reads; Argo does not accept short reads, so this pipeline does not extend to second-generation data.

## Evidence

- [other] QC toolchain parameters: [evidence span withheld — anchor status: not_found]
