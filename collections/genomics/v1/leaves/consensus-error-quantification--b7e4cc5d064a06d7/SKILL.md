---
name: consensus-error-quantification
description: Use when benchmarking consensus sequence callers on simulated or real
  reads and you must verify whether reconstruction error decreases with coverage,
  differs by sequencing platform, or is elevated for a specific viral lineage.
when_to_use_negative:
- Input is raw read data with no called consensus sequences — consensus calling must
  precede error quantification
- Input lacks replication (single observation per condition), so per-replicate spread
  and effect-size sign consistency cannot be assessed
- The goal is caller speed or memory benchmarking rather than reconstruction accuracy
  — the paper's headline result is that ViralConsensus is orders of magnitude faster
  and more memory-efficient, which does not require error-trend analysis
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0622
tools:
- name: ViralConsensus
  role: Calls viral consensus genome sequences directly from read alignment data (BAM)
    for each replicate/coverage condition being benchmarked
  repo: github.com/niemasd/ViralConsensus
- name: htslib
  role: C library dependency used by ViralConsensus to read SAM/BAM/CRAM alignment
    input
  repo: github.com/samtools/htslib
- name: minimap2
  role: Read mapper whose output can be piped via standard input directly into viral_consensus
    for on-the-fly consensus calling
- name: pandas
  role: Loads and stratifies the per-replicate consensus error table by platform,
    lineage, and coverage depth
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:c99783bac6b0a7ecbff043bffb04df0e1217b7779c84f75354154b2a4ef3331b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ViralConsensus
  - htslib
  - minimap2
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# consensus-error-quantification

## Summary

Quantify viral consensus genome reconstruction error from ViralConsensus benchmark outputs by stratifying per-replicate error tables over sequencing platform (Illumina vs ONT), SARS-CoV-2 lineage (e.g. BA.1), and coverage depth (30x/40x/50x), then testing trend direction and effect sizes.

## When to use

Use when benchmarking consensus sequence callers on simulated or real reads and you must verify whether reconstruction error decreases with coverage, differs by sequencing platform, or is elevated for a specific viral lineage.

## When NOT to use

- Input is raw read data with no called consensus sequences — consensus calling must precede error quantification
- Input lacks replication (single observation per condition), so per-replicate spread and effect-size sign consistency cannot be assessed
- The goal is caller speed or memory benchmarking rather than reconstruction accuracy — the paper's headline result is that ViralConsensus is orders of magnitude faster and more memory-efficient, which does not require error-trend analysis

## Inputs

- Per-replicate consensus error table from the ViralConsensus accuracy benchmark (simulated reads from five SARS-CoV-2 lineages at 30x/40x/50x coverage)
- Consensus sequences (FASTA) called from read alignment data (BAM from Illumina or ONT mapper output) with ViralConsensus
- SARS-CoV-2 reference sequence (e.g. NC_045512.2 FASTA)

## Outputs

- error_trend_verification.tsv: trend verdicts with fold-change and Cohen's d effect sizes for coverage, platform, and lineage contrasts
- Per-condition mean and per-replicate spread of consensus error
- Consensus genome FASTA per replicate from ViralConsensus

## How to apply

Load the per-replicate consensus error table with pandas, stratifying rows by sequencing platform (Illumina vs ONT), viral lineage (including BA.1), and coverage depth (30x, 40x, 50x) from the ViralConsensus accuracy benchmark on simulated SARS-CoV-2 reads. Compute mean and per-replicate spread of consensus error within each coverage level, then test the monotonic decrease across 30x > 40x > 50x within each condition. At matched coverage levels, compare ONT versus Illumina error and at matched platform compare BA.1 against other lineages, reporting fold-change and a standardized effect size (Cohen's d) so direction and magnitude are both recorded. Because the paper reports no numeric acceptance threshold, accept trends via qualitative consistency: error_50x < error_40x < error_30x per condition, ONT error above Illumina at equal coverage, and BA.1-Illumina error noticeably above other Illumina lineages, with consistently signed non-negligible effect sizes across replicates. Compile all trend verdicts and effect sizes into a single TSV (error_trend_verification.tsv) as the auditable output. Consensus sequences themselves should be produced with ViralConsensus, which can pipe mapper output (e.g. minimap2 or samtools) directly via standard input to avoid unnecessary disk I/O.

## Related tools

- **ViralConsensus** (Calls viral consensus genome sequences directly from read alignment data (BAM) for each replicate/coverage condition being benchmarked) — github.com/niemasd/ViralConsensus
- **htslib** (C library dependency used by ViralConsensus to read SAM/BAM/CRAM alignment input) — github.com/samtools/htslib
- **minimap2** (Read mapper whose output can be piped via standard input directly into viral_consensus for on-the-fly consensus calling)
- **pandas** (Loads and stratifies the per-replicate consensus error table by platform, lineage, and coverage depth)

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- error_50x < error_40x < error_30x at each platform/lineage condition (monotonic decrease with coverage)
- ONT consensus error exceeds Illumina error at every matched coverage level, with positive fold-change and Cohen's d
- BA.1 Illumina-replicate errors are noticeably above other Illumina lineages, with consistent effect-size sign across replicates
- Output TSV (error_trend_verification.tsv) contains all three trend verdicts with consistently signed, non-negligible effect sizes
- Consensus FASTA files regenerate via the documented pipeline: minimap2 piped to viral_consensus with the NC_045512.2 reference

## Limitations

- Memory consumption of ViralConsensus scales linearly with reference genome length, so ultra-long references (e.g. human chromosomes) may be prohibitively large
- The paper reports no numeric threshold for error trends, so acceptance is a qualitative consistency check of trend direction and effect-size sign across replicates rather than a formal statistical cutoff
- Effect sizes and trend verdicts rest on simulated reads from five SARS-CoV-2 lineages at 30x/40x/50x; extrapolation to other viruses, coverages, or real reads is not validated here
- The discussion section text was not present in the provided document, so caveats beyond the indexed findings are unavailable

## Evidence

- [results] Consensus error decreases with coverage: [evidence span withheld — anchor status: not_found]
- [methods] ViralConsensus calling from alignment data: "viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas"
- [abstract] On-the-fly consensus piping: "unlike existing methods, ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly"
- [readme] Memory scaling limitation: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
- [methods] Mapper pipe with tee: "data can be piped directly from the read mapper to ViralConsensus to avoid unnecessary disk I/O"
