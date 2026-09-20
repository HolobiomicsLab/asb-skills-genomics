---
name: long-read-alignment-to-reference-genome
description: Use when HiFi (CCS) long reads need to be aligned to GRCh38 as the evidence
  layer for structural variant discovery, e.g. before running sawfish discover.
when_to_use_negative:
- Input reads are short Illumina reads rather than HiFi/CCS long reads — pbmm2's CCS
  preset and the downstream sawfish SV model assume mapped high-quality long reads
- The goal is de novo assembly or read-to-read overlap rather than reference-based
  SV evidence generation
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: pbmm2
  role: Aligns HiFi CCS reads to the GRCh38 no-alt reference using the CCS preset
    with --sort and --unmapped, producing the input BAM for sawfish discover
  repo: https://github.com/PacificBiosciences/sawfish
- name: sawfish
  role: Consumes the pbmm2-aligned BAM in its discover step to scan for SV/CNV evidence;
    version 0.12.9 used in the analysis
  repo: https://github.com/PacificBiosciences/sawfish
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:efe1c68c2775b0e712cc605c0e1556b02654ed032a262e7fc20bd7d618d20d70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - WFA2-lib
  - pbmm2
  - sawfish
  - spoa
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# long-read-alignment-to-reference-genome

## Summary

Maps HiFi long reads to a human reference genome with pbmm2 so that downstream SV/CNV callers can scan alignments for breakpoint evidence. In the sawfish workflow, pbmm2 1.13.1 alignment of HG002 Revio HiFi reads against the human_GRCh38_no_alt_analysis_set reference is the prerequisite input for SV discovery and genotyping.

## When to use

Use when HiFi (CCS) long reads need to be aligned to GRCh38 as the evidence layer for structural variant discovery, e.g. before running sawfish discover.

## When NOT to use

- Input reads are short Illumina reads rather than HiFi/CCS long reads — pbmm2's CCS preset and the downstream sawfish SV model assume mapped high-quality long reads
- The goal is de novo assembly or read-to-read overlap rather than reference-based SV evidence generation

## Inputs

- HiFi CCS read BAM (e.g. m84011_220902_175841_s1.hifi_reads.bam from HG002 Revio sequencing)
- Reference genome FASTA (human_GRCh38_no_alt_analysis_set/human_GRCh38_no_alt_analysis_set.fasta)
- pbmm2 1.13.1 (conda-installable per the article's recommended environment)

## Outputs

- Sorted, coordinate-ordered alignment BAM with unmapped reads retained (e.g. m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam), suitable as input to sawfish discover

## How to apply

Obtain the human_GRCh38_no_alt_analysis_set reference FASTA and unpack it so the reference path is human_GRCh38_no_alt_analysis_set/human_GRCh38_no_alt_analysis_set.fasta. Align the HiFi reads (e.g. m84011_220902_175841_s1.hifi_reads.bam) with pbmm2 1.13.1 using the CCS preset, supplying --sample to embed the sample name, and enabling --sort and --unmapped with 16 jobs. The sorted, coordinate-ordered BAM with unmapped reads retained becomes the input to sawfish discover, which scans the alignments for breakpoint evidence, clusters them, and assembles SV haplotype contigs (via spoa POA and WFA2-lib wavefront alignment). Alignment quality matters because sawfish filters reads during its scan if they are flagged unmapped, secondary, QC-failed or duplicate, have gap-compressed identity below 0.97, or mapping quality below 5.

## Related tools

- **pbmm2** (Aligns HiFi CCS reads to the GRCh38 no-alt reference using the CCS preset with --sort and --unmapped, producing the input BAM for sawfish discover) — https://github.com/PacificBiosciences/sawfish
- **sawfish** (Consumes the pbmm2-aligned BAM in its discover step to scan for SV/CNV evidence; version 0.12.9 used in the analysis) — https://github.com/PacificBiosciences/sawfish

## Examples

```
pbmm2 align --log-level INFO -j 16 --preset CCS --sample HG002 --sort --unmapped m84011_220902_175841_s1.hifi_reads.bam human_GRCh38_no_alt_analysis_set/human_GRCh38_no_alt_analysis_set.fasta m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam
```

## Evaluation signals

- Output BAM exists, is BGZF-compressed and coordinate-sorted, and the RG/sample tag matches --sample HG002
- Unmapped reads are retained in the BAM (because --unmapped was used)
- Downstream sawfish discover completes and produces the expected sawfish_discover_output directory with depth track and candidate SVs
- Mapped read alignments pass sawfish's read filter (not flagged unmapped/secondary/QC-fail/duplicate, gap-compressed identity >= 0.97, mapping quality >= 5)

## Limitations

- sawfish filters out reads with gap-compressed identity < 0.97 or mapping quality < 5, so very noisy alignments silently drop out of SV evidence
- Soft-clipped read ends only contribute breakend candidates when at least 500 bases are clipped on one end with no clipping on the other end
- Benchmarking used pbmm2 1.13.1 specifically; other mappers or versions were not assessed in this article

## Evidence

- [methods] pbmm2 alignment to GRCh38 reference: "pbmm2 --log-level INFO align -j 16 --preset CCS --sample HG002 --sort --unmapped"
- [methods] sawfish read filtering thresholds: "Reads are filtered from this scan if they are flagged as unmapped, secondary, QC-failed or duplicate. Reads with gap-compressed identity less than 0.97 are filtered out"
- [methods] soft-clip breakend filter: "Soft-clipped read ends only contribute to breakend candidates when at least 500 bases of the read are soft-clipped on one end, and no clipping is found on the other end"
- [methods] sawfish discover step definition: "`discover` - The discover step needs to be run once on each sample prior to entering that sample in the the joint-call step."
- [readme] sawfish scope statement: "Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads."
