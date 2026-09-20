---
name: sars-cov-2-lineage-benchmark-analysis
description: Use when analyzing per-replicate consensus error tables stratified by
  sequencing platform (Illumina vs ONT), viral lineage (including BA.1), and simulated
  coverage depth (30x, 40x, 50x) to verify replication of reported error trends.
when_to_use_negative:
- Input is raw read data or read alignments without a precomputed per-replicate consensus
  error table — consensus must first be called with ViralConsensus
- The genome reference is ultra-long (e.g. human chromosomes), where ViralConsensus
  memory consumption scales linearly with reference length and may be prohibitive
- The question concerns runtime or memory benchmarks of consensus callers rather than
  reconstruction error trends across coverage, platform, and lineage
edam_topics:
- http://edamontology.org/topic_0781
tools:
- name: ViralConsensus
  role: Tool whose accuracy benchmark supplies the consensus sequences and per-replicate
    reconstruction error values being analyzed for coverage, platform, and lineage
    trends
  repo: https://github.com/niemasd/ViralConsensus
- name: manual expert review
  role: Qualitative acceptance check that the three reported trends replicate with
    consistently signed, non-negligible effect sizes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:b0da08bed0a8125ed52bdbd9f257998e2a12d275305ac7c6e30b7810d8e3339f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - ViralConsensus
  - manual expert review
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# sars-cov-2-lineage-benchmark-analysis

## Summary

Verifies consensus reconstruction error trends from the ViralConsensus accuracy benchmark on simulated SARS-CoV-2 reads, testing that error decreases monotonically across 30x, 40x, and 50x coverage, that ONT error exceeds Illumina error at matched coverage, and that Illumina-reconstructed BA.1 shows elevated error versus other lineages.

## When to use

Use when analyzing per-replicate consensus error tables stratified by sequencing platform (Illumina vs ONT), viral lineage (including BA.1), and simulated coverage depth (30x, 40x, 50x) to verify replication of reported error trends.

## When NOT to use

- Input is raw read data or read alignments without a precomputed per-replicate consensus error table — consensus must first be called with ViralConsensus
- The genome reference is ultra-long (e.g. human chromosomes), where ViralConsensus memory consumption scales linearly with reference length and may be prohibitive
- The question concerns runtime or memory benchmarks of consensus callers rather than reconstruction error trends across coverage, platform, and lineage

## Inputs

- Per-replicate consensus sequence reconstruction error table (TSV/CSV) keyed by replicate, sequencing platform (Illumina, ONT), SARS-CoV-2 lineage (including BA.1), and coverage depth (30x, 40x, 50x)
- Consensus FASTA files produced by ViralConsensus from simulated read alignments (BAM), e.g. example.trimmed.unsorted.bam against the NC_045512.2 reference

## Outputs

- error_trend_verification.tsv summarizing trend verdicts, fold-changes, and Cohen's d effect sizes for the coverage, platform, and lineage contrasts
- Per-coverage-level mean error and per-replicate spread statistics

## How to apply

Load the per-replicate consensus error table with pandas, stratifying rows by sequencing platform (Illumina vs ONT), viral lineage (including BA.1), and coverage depth (30x, 40x, 50x). Compute per-coverage-level means and per-replicate spread, then test the monotonic decrease 30x > 40x > 50x within each platform-lineage condition. Compare ONT versus Illumina error at each matched coverage level, reporting fold-change and Cohen's d to confirm ONT reconstruction has higher error. Contrast BA.1 Illumina-replicate errors against the other lineages' Illumina errors, again quantifying fold-change and Cohen's d for the elevated-error observation. Compile all trend verdicts and effect sizes into a single TSV (error_trend_verification.tsv). Acceptance is qualitative consistency of trend direction and effect-size sign across replicates, since the paper reports no numeric threshold.

## Related tools

- **ViralConsensus** (Tool whose accuracy benchmark supplies the consensus sequences and per-replicate reconstruction error values being analyzed for coverage, platform, and lineage trends) — https://github.com/niemasd/ViralConsensus
- **manual expert review** (Qualitative acceptance check that the three reported trends replicate with consistently signed, non-negligible effect sizes)

## Examples

```
viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas
```

## Evaluation signals

- Monotonic ordering holds at each condition: error_50x < error_40x < error_30x
- ONT consensus error exceeds Illumina error at equal coverage, with fold-change and Cohen's d of consistent, non-negligible sign
- BA.1 Illumina-replicate error is noticeably above other Illumina lineages' error
- error_trend_verification.tsv is a well-formed TSV containing a verdict and effect size for each of the three tested trends
- Since the paper reports no numeric threshold, all effect-size signs agree across replicates (qualitative consistency criterion)

## Limitations

- The paper reports no numeric threshold for the trends; acceptance relies on qualitative consistency of trend direction and effect-size sign rather than a predefined cutoff
- ViralConsensus memory consumption scales linearly with reference genome length, limiting use on ultra-long references
- Findings derive from simulated reads across five SARS-CoV-2 lineages and simulated coverage levels (30x, 40x, 50x), not empirical sequencing runs

## Evidence

- [abstract] Abstract tool statement: "ViralConsensus is a fast and memory-efficient tool for calling viral consensus genome sequences directly from read alignment data."
- [intro] Piping capability: "ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly, making it an ideal tool for viral sequencing pipelines."
- [readme] Consensus calling command: "viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas"
- [readme] Memory scaling caveat: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
