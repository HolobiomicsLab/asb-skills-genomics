---
name: ont-read-alignment-and-bam-indexing
description: Use when basecalled ONT reads and their signal-level FAST5 data must
  be linked to a reference genome before methylation calling, i.e. when a sorted,
  indexed BAM is needed as input to nanopolish/f5c/Galaxy-methyl call-methylation.
when_to_use_negative:
- Reads are not Oxford Nanopore long reads — use the matching minimap2 preset instead
  (e.g. -ax sr for short Illumina reads, -ax map-pb for PacBio CLR)
- The analysis does not require signal-level methylation or event alignment and only
  needs approximate mapping — plain minimap2 PAF output without -a/samtools suffices
- Nanopore R10.4 flowcells with Nanopolish, which presently does not support R10.4
  methylation calling
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3573
tools:
- name: minimap2
  role: Aligns basecalled ONT reads to the reference genome in SAM mode (-a) with
    the map-ont preset; output piped to samtools sort
  repo: https://github.com/lh3/minimap2
- name: samtools v1.x
  role: Sorts the minimap2 SAM output into a coordinate-sorted BAM and indexes it
    for the methylation callers
  repo: https://www.htslib.org/
- name: Nanopolish
  role: Provides the index step that links read ids with signal-level data in FAST5
    files prior to alignment
  repo: https://github.com/jts/nanopolish
- name: f5c
  role: Alternative index implementation (f5c index) linking read ids to FAST5 signal-level
    data; downstream consumer of the sorted BAM via call-methylation
  repo: https://github.com/hasindu2008/f5c
- name: Galaxy-methyl
  role: Downstream consumer of the sorted, indexed BAM for GPU-accelerated call-methylation
  repo: https://github.com/fengyilin118/Galaxy-methyl
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:67459c491f1bc2847d1c0473a015415bb5612079fd31f83fa8606f2d11ee35ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Galaxy-methyl
  - Nanopolish
  - f5c
  - minimap2
  - samtools
  - samtools v1.x
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3573
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ONT read alignment and BAM indexing

## Summary

Aligns basecalled Oxford Nanopore reads to a reference genome with minimap2 (-a -x map-ont) and pipes the output into samtools sort to produce a sorted, indexed BAM, which is the prerequisite input for downstream call-methylation with Nanopolish, f5c, or Galaxy-methyl.

## When to use

Use when basecalled ONT reads and their signal-level FAST5 data must be linked to a reference genome before methylation calling, i.e. when a sorted, indexed BAM is needed as input to nanopolish/f5c/Galaxy-methyl call-methylation.

## When NOT to use

- Reads are not Oxford Nanopore long reads — use the matching minimap2 preset instead (e.g. -ax sr for short Illumina reads, -ax map-pb for PacBio CLR)
- The analysis does not require signal-level methylation or event alignment and only needs approximate mapping — plain minimap2 PAF output without -a/samtools suffices
- Nanopore R10.4 flowcells with Nanopolish, which presently does not support R10.4 methylation calling

## Inputs

- Basecalled Oxford Nanopore reads (e.g. FASTQ) for ENA datasets ERR2184700, ERR2184710, ERR2184719, ERR1676719, ERR1676724, ERR167672
- Indexed signal-level data linking read ids to FAST5 files (from nanopolish/f5c index)
- Reference genome FASTA: hg38 for D1-D3 and D5-D6, Escherichia_coli_str_k_12_substr_mg1655.GCA_000005845.2 for D4

## Outputs

- Coordinate-sorted BAM file of reads aligned to the reference genome (via minimap2 -a piped into samtools sort)
- BAM index file (from samtools index), required by call-methylation tools

## How to apply

First create an index file that links read ids with their signal-level data in the FAST5 files (nanopolish index / f5c index). Then align the basecalled reads to the matching reference genome (hg38 for human datasets D1-D3, D5-D6; E. coli K-12 MG1655 GCA_000005845.2 for D4) using minimap2 with the -a (SAM output) and -x map-ont (Oxford Nanopore genomic reads) presets. Pipe the minimap2 SAM output directly into samtools sort to obtain a coordinate-sorted BAM file, then run samtools index to create the BAM index. The resulting sorted, indexed BAM is consumed by each tool's call-methylation command (run with -t 4 CPU threads and -B 4.0M batch size in the Galaxy-methyl benchmarks).

## Related tools

- **minimap2** (Aligns basecalled ONT reads to the reference genome in SAM mode (-a) with the map-ont preset; output piped to samtools sort) — https://github.com/lh3/minimap2
- **samtools v1.x** (Sorts the minimap2 SAM output into a coordinate-sorted BAM and indexes it for the methylation callers) — https://www.htslib.org/
- **Nanopolish** (Provides the index step that links read ids with signal-level data in FAST5 files prior to alignment) — https://github.com/jts/nanopolish
- **f5c** (Alternative index implementation (f5c index) linking read ids to FAST5 signal-level data; downstream consumer of the sorted BAM via call-methylation) — https://github.com/hasindu2008/f5c
- **Galaxy-methyl** (Downstream consumer of the sorted, indexed BAM for GPU-accelerated call-methylation) — https://github.com/fengyilin118/Galaxy-methyl

## Examples

```
minimap2 -a -x map-ont ref.fa reads.fq.gz | samtools sort -o sorted.bam - ; samtools index sorted.bam
```

## Evaluation signals

- samtools sort exits cleanly and the output BAM is coordinate-sorted; samtools index produces a .bai/.csi index without errors
- minimap2 is invoked with -a -x map-ont so the output is SAM format aligned with the Oxford Nanopore preset, not raw PAF
- The indexed BAM's read names match the read ids in the FAST5 index produced by the index step, so call-methylation can resolve signals for every mapped read
- The correct reference genome is used per dataset: hg38 for D1-D3/D5-D6 and the E. coli K-12 MG1655 GCA_000005845.2 assembly for D4
- Downstream call-methylation (nanopolish/f5c/Galaxy-methyl with -t 4 and -B 4.0M) runs to completion and writes methylation_calls.tsv per dataset

## Limitations

- minimap2 indexing parameters (-k, -w, -H, -I) are frozen once the reference index is built with -d; a rebuilt index is needed for different data types
- Nanopolish does not support R10.4 flowcells for methylation calling, limiting this workflow on newer chemistries
- Human reference genome indexing in minimap2 takes a few minutes, so pre-building and saving the index with -d can reduce repeated alignment cost

## Evidence

- [methods] align the basecalled reads to the reference genome.  minimap2 is used to map reads to the human genome: "align the basecalled reads to the reference genome.  minimap2 is used to map reads to the human genome"
- [methods] The output is piped directly into samtools sort to get a sorted bam file: "The output is piped directly into samtools sort to get a sorted bam file"
- [methods] create an index file that links read ids with their signal-level data in the FAST5 files: "create an index file that links read ids with their signal-level data in the FAST5 files"
- [readme] ./minimap2 -a test/MT-human.fa test/MT-orang.fa > test.sam: [evidence span withheld — anchor status: external_doc]
- [readme] it should be noted that once you build the index, indexing parameters such as -k, -w, -H and -I can't be changed during mapping: [evidence span withheld — anchor status: not_found]
