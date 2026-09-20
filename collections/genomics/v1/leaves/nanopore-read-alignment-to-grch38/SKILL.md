---
name: nanopore-read-alignment-to-grch38
description: Use when Dorado sup-basecalled ONT reads (kit v14, R10.4.1) from samples
  such as HG002/HG003/HG004 must be placed on GRCh38 as sorted, indexed BAMs before
  variant calling.
when_to_use_negative:
- Input reads are short reads (Illumina/PacBio) — the alignment strategy and downstream
  callers (miniSNV, Longshot) are optimized for ONT long-read error profiles
- The sample is the kit v12 HG002 Guppy-basecalled dataset (k12-data) — that benchmark
  uses a different chemistry and basecaller and separate evaluation
- The goal is variant calling itself rather than producing the aligned BAM input —
  use the miniSNV/Clair3/Longshot calling skill instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: miniSNV
  role: Downstream SNV caller whose input requirement (samtools-indexed BAM, --fin_bam)
    constrains the alignment output format
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Oxford Nanopore Technologies (ONT)
  role: Sequencing platform generating the PromethION kit v14 R10.4.1 long reads that
    are aligned to GRCh38
- name: Dorado
  role: Basecaller (sup model) that produced the reads prior to alignment
- name: Clair3
  role: Alternative downstream caller run on the same aligned BAMs using the nanoporetech/rerio
    ONT-trained model
  repo: https://github.com/nanoporetech/rerio
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:9d06275927b36bae4b59a2533102bad2386757aecbfaf1abe7e3429b13a9a33a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Clair3
  - Delly
  - Dorado
  - LAST
  - Longshot
  - Lumpy
  - Manta
  - MinION nanopore sequencer
  - NanoSV
  - Oxford Nanopore Technologies (ONT)
  - Sniffles
  - miniSNV
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- nanopore-read-alignment-with-last
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# nanopore-read-alignment-to-grch38

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps ONT (kit v14, R10.4.1, Dorado sup-basecalled) nanopore reads onto the GRCh38 reference (GCA_000001405.15) to produce sorted, samtools-indexed BAM files that serve as the input for SNV callers such as miniSNV, Clair3, and Longshot in the GIAB Ashkenazi trio benchmark.

## When to use

Use when Dorado sup-basecalled ONT reads (kit v14, R10.4.1) from samples such as HG002/HG003/HG004 must be placed on GRCh38 as sorted, indexed BAMs before variant calling.

## When NOT to use

- Input reads are short reads (Illumina/PacBio) — the alignment strategy and downstream callers (miniSNV, Longshot) are optimized for ONT long-read error profiles
- The sample is the kit v12 HG002 Guppy-basecalled dataset (k12-data) — that benchmark uses a different chemistry and basecaller and separate evaluation
- The goal is variant calling itself rather than producing the aligned BAM input — use the miniSNV/Clair3/Longshot calling skill instead

## Inputs

- Dorado sup-basecalled ONT reads (FASTQ/POD5) from GIAB Ashkenazi trio HG002, HG003, HG004 (kit v14, R10.4.1, ~200 Gbases per sample)
- GRCh38 reference FASTA (GCA_000001405.15), samtools-indexed

## Outputs

- Per-sample sorted BAM files aligned to GRCh38 (HG002, HG003, HG004)
- samtools BAM index (.bai) for each sorted BAM

## How to apply

Obtain the GIAB AshkenazimTrio ONT deposits (PromethION, ligation sequencing kit v14, R10.4.1 flowcells, ~200 Gbases per sample, Dorado sup-basecalled) and align them against GRCh38 (GCA_000001405.15), producing sorted BAM files per sample. The resulting BAM must be samtools-indexed, since miniSNV explicitly requires an indexed BAM input via --fin_bam. Downstream callers then treat low-quality alignments — MAPQ < 20, supplementary, secondary, and unmapped reads — as filtered, so the alignment step should preserve the MAPQ tags and supplementaries needed for that filtering to act correctly. Evaluate the output indirectly: miniSNV on these aligned sup-dataset BAMs achieved F1-score and recall > 99.7% on HG002, HG003, and HG004, which is the reproducibility signal for the alignment pipeline.

## Related tools

- **miniSNV** (Downstream SNV caller whose input requirement (samtools-indexed BAM, --fin_bam) constrains the alignment output format) — https://github.com/CuiMiao-HIT/miniSNV
- **Oxford Nanopore Technologies (ONT)** (Sequencing platform generating the PromethION kit v14 R10.4.1 long reads that are aligned to GRCh38)
- **Dorado** (Basecaller (sup model) that produced the reads prior to alignment)
- **Clair3** (Alternative downstream caller run on the same aligned BAMs using the nanoporetech/rerio ONT-trained model) — https://github.com/nanoporetech/rerio

## Examples

```
samtools index HG003.GRCh38.sorted.bam
```

## Evaluation signals

- Each output BAM is samtools-indexed (miniSNV: 'The input file must be samtools indexed')
- Alignment preserves MAPQ tags so the downstream MAPQ < 20 filter and supplementary/secondary/unmapped-read filtering can operate
- ~200 Gbases of aligned sequence per GIAB trio sample as reported for the v14 chemistry dataset
- Downstream miniSNV on these BAMs reproduces F1-score and recall > 99.7% on HG002, HG003, HG004 sup-datasets
- Reference contig naming matches GRCh38 (GCA_000001405.15) so the 1000 Genomes known-variant index and GIAB stratifications apply directly

## Limitations

- Reads with low mapping quality (MAPQ < 20), supplementary alignments, secondary alignments, and unmapped reads are discarded by miniSNV, so alignment ambiguities in repetitive regions propagate as missed SNVs
- Only the kit v14 Dorado sup-basecalled GIAB trio dataset is validated end-to-end here; other chemistries (e.g. kit v12 Guppy basecalled data) were benchmarked separately
- The Clair3 models bundled in nanoporetech/rerio are research models not validated by ONT and are compatible with Clair3 v1.x but not v2.x

## Evidence

- [results] Source dataset chemistry and depth for the sup-dataset benchmark: [evidence span withheld — anchor status: not_found]
