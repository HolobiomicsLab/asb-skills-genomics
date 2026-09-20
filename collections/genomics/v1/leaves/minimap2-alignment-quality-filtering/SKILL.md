---
name: minimap2-alignment-quality-filtering
description: Use when ONT nanopore reads must be aligned to a reference genome and
  low-quality mappings removed before pileup-based variant calling, e.g. producing
  sorted, samtools-indexed BAMs as miniSNV input.
when_to_use_negative:
- Input is already a variant callset (e.g. a population VCF for the known-common-variant
  index) rather than raw ONT reads needing alignment
- The analysis targets short reads optimized pipelines; the article notes most existing
  SNV methods are short-read optimized while this filtering scheme is designed for
  ONT long reads
- A BAM is supplied without samtools indexing, since miniSNV explicitly requires samtools-indexed
  FASTA and BAM inputs
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3169
tools:
- name: miniSNV
  role: Applies the MAPQ < 20 alignment-quality filter (plus removal of supplementary,
    secondary, and unmapped alignments) before read pileup and candidate loci recognition
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Oxford Nanopore Technologies (ONT)
  role: Source of the long-read data that requires long-read-aware alignment and quality
    filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:c74b779b058fea46e91f1645d481df40555990af69d5f02f4f4110c0174de972
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
  - Oxford Nanopore Technologies (ONT)
  - miniSNV
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# minimap2-alignment-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align ONT long reads (e.g. HG002 k12/k14 PromethION data) to GRCh38 with Minimap2 and apply miniSNV's read-alignment quality filtering (MAPQ >= 20, removing supplementary, secondary, and unmapped alignments) so that only high-confidence sorted BAM alignments feed downstream SNV calling.

## When to use

Use when ONT nanopore reads must be aligned to a reference genome and low-quality mappings removed before pileup-based variant calling, e.g. producing sorted, samtools-indexed BAMs as miniSNV input.

## When NOT to use

- Input is already a variant callset (e.g. a population VCF for the known-common-variant index) rather than raw ONT reads needing alignment
- The analysis targets short reads optimized pipelines; the article notes most existing SNV methods are short-read optimized while this filtering scheme is designed for ONT long reads
- A BAM is supplied without samtools indexing, since miniSNV explicitly requires samtools-indexed FASTA and BAM inputs

## Inputs

- Oxford Nanopore Technologies long reads (e.g. HG002 k12 or k14 PromethION data, FASTQ/POD5 origin)
- GRCh38 reference genome FASTA (samtools-indexed)
- Sorted BAM alignment of reads against GRCh38 (samtools-indexed) for miniSNV input

## Outputs

- Sorted, indexed BAM containing only high-confidence primary alignments (MAPQ >= 20)
- Candidate-loci pileup derived from the filtered alignments for variant calling

## How to apply

Align the ONT reads (e.g. HG002 kit v12 FLO-PRO112 simplex reads or kit v14 Dorado-basecalled reads) to GRCh38 with Minimap2, then sort and index the BAM since miniSNV requires samtools-indexed inputs. miniSNV then filters out all read alignments with mapping quality below the default MAPQ < 20 threshold, along with supplementary alignments, secondary alignments, and unmapped reads, before building the read pileup used to recognize candidate loci. Additional candidate-level thresholds applied on the filtered alignments include base quality >= 13 for candidate sites (default) and an alignment consistency threshold of 0.98 (read_Ratio). The rationale is that the pileup stage and the downstream classification (known variants with R_alt > 0.3, unknown loci with R_alt > 0.7) are only reliable when built from high-confidence primary alignments.

## Related tools

- **miniSNV** (Applies the MAPQ < 20 alignment-quality filter (plus removal of supplementary, secondary, and unmapped alignments) before read pileup and candidate loci recognition) — https://github.com/CuiMiao-HIT/miniSNV
- **Oxford Nanopore Technologies (ONT)** (Source of the long-read data that requires long-read-aware alignment and quality filtering)

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG002_k12.sorted.bam --workDir work --useindex -i index_folder --human
```

## Evaluation signals

- Input FASTA and BAM are samtools-indexed, as required by miniSNV's --fin_ref and --fin_bam parameters
- No alignment with MAPQ below 20, and no supplementary, secondary, or unmapped alignment, contributes to the candidate-loci pileup
- Candidate sites pass the minimum base quality of 13 and the alignment consistency threshold of 0.98 before entering classification
- Downstream callsets evaluated against the GIAB HG002 truth set with GIAB v3.3 stratifications reproduce the reported precision/recall/F1 (miniSNV > 99% F1 on k12-data)

## Limitations

- The MAPQ < 20 threshold is a default that can be changed by the user; filtering stringency affects which candidate loci are recovered
- Any candidate locus located in a low-complexity region (per GIAB v3.3 stratification annotations) is classified low-quality regardless of alignment quality
- Filtering occurs after alignment, so multi-mapping reads in repetitive regions may already be lost at the Minimap2 step and yield undetected SNVs if under-mapped

## Evidence
