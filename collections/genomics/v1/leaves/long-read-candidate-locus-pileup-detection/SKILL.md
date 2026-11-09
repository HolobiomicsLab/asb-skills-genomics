---
name: long-read-candidate-locus-pileup-detection
description: Use when calling SNVs from ONT long-read BAM alignments and you need
  to detect candidate variant loci via read pileup with known-variant background stratification
  before genotyping.
when_to_use_negative:
- Input reads are short-read Illumina alignments — miniSNV's pileup and phasing strategy
  is optimized for ONT long reads
- The BAM is not sorted and samtools-indexed, since the chunked pileup stage requires
  indexed random access
- The goal is indel or structural variant calling rather than single nucleotide variants
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: miniSNV
  role: Implements the read pileup, known-variant index lookup, and high/low-quality
    locus classification for candidate SNV detection
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Oxford Nanopore Technologies (ONT)
  role: Source sequencing platform whose long reads with error profiles the pileup
    thresholds were tuned for
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:18cc88e80c6f921c8e452958b2c635aaa700a4e257e5a995896c9a6af2862303
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
  - KSW2
  - Oxford Nanopore Technologies (ONT)
  - WhatsHap
  - abPOA
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# long-read-candidate-locus-pileup-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces miniSNV's candidate-finder stage: after MAPQ/alignment filtering, read pileup over chunked reference windows identifies positions where the alternative allele ratio R_alt exceeds 0.2, then stratifies loci into high- and low-quality classes against a prebuilt 1000 Genomes common-variant index and low-complexity annotations.

## When to use

Use when calling SNVs from ONT long-read BAM alignments and you need to detect candidate variant loci via read pileup with known-variant background stratification before genotyping.

## When NOT to use

- Input reads are short-read Illumina alignments — miniSNV's pileup and phasing strategy is optimized for ONT long reads
- The BAM is not sorted and samtools-indexed, since the chunked pileup stage requires indexed random access
- The goal is indel or structural variant calling rather than single nucleotide variants

## Inputs

- sorted, samtools-indexed ONT BAM alignment file (e.g. HG002 k12-data kit v12 alignments)
- samtools-indexed reference FASTA (e.g. GRCh38)
- miniSNV index folder (five files built from a 1000 Genomes population VCF), used with --useindex
- optional HOMO/DUP BED files (--homo_bed, --dup_bed)

## Outputs

- per-chunk candidate loci table (e.g. candidate_sites.tsv) with chrom, start, ref, alt, and supporting-read count columns
- locus classification (high- vs low-quality) carried forward to binomial genotyping and consensus generation

## How to apply

First filter the sorted, samtools-indexed BAM by removing alignments with MAPQ < 20 plus supplementary, secondary, and unmapped records. Partition the reference (e.g. GRCh38) into chunks of chunkWidth 10,000,000 bp and tally cumulative A/C/G/T allele counts at each position per chunk. Mark a position with a specific alternative allele (identified by its UID) as a candidate when R_alt exceeds the default threshold 0.2, applying the minimum base quality 13 and read_Ratio 0.98 alignment-consistency filters. Optionally load the miniSNV index (five files, built from a 1000 Genomes population VCF, UIDs of form wid_pos_ref_alt) and classify each candidate: known variants with R_alt > 0.3, or unknown variants with R_alt > 0.7 and no other candidate within 10 bp, become high-quality; remaining and low-complexity-region loci become low-quality for later consensus-based genotyping.

## Related tools

- **miniSNV** (Implements the read pileup, known-variant index lookup, and high/low-quality locus classification for candidate SNV detection) — https://github.com/CuiMiao-HIT/miniSNV
- **Oxford Nanopore Technologies (ONT)** (Source sequencing platform whose long reads with error profiles the pileup thresholds were tuned for)

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG002_k12.bam --workDir ./work --useindex -i ./index --human --chunkWidth 10000000 --read_Ratio 0.98 --baseQ 13
```

## Evaluation signals

- Every retained candidate site has an alternative allele ratio above the 0.2 threshold consistent with the pileup criteria
- candidate_sites.tsv exists and is a tab-separated table containing chrom/start/ref/alt/count fields
- Alignments with MAPQ < 20, supplementary, secondary, and unmapped records are absent from the pileup input
- High-quality loci satisfy either known-variant R_alt > 0.3 or unknown-variant R_alt > 0.7 with no other candidate within 10 bp; low-complexity loci are classified low-quality

## Limitations

- The R_alt thresholds of 0.3–0.7 were selected by in-house experiments (Supplementary Table S1) and may need re-tuning for other error profiles
- Any candidate locus in a low-complexity region is forced into the low-quality class, deferring it to the more expensive consensus-generation path
- Default parameters (chunkWidth 10000000, read_Ratio 0.98, baseQ 13) assume human-scale genomes and ONT error rates

## Evidence

- [methods] R_alt threshold: [evidence span withheld — anchor status: not_found]
