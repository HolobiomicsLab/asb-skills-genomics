---
name: nanopore-read-subsampling-coverage-scaling
description: Use when assessing the minimum nanopore sequencing depth at which nanoHiMe-seq
  CpG methylation calls remain accurate versus WGBS, e.g. before committing MinION
  flowcells for a low-coverage epigenomics experiment.
when_to_use_negative:
- When the question is histone-modification enrichment (6mA peak calling) rather than
  CpG methylation accuracy vs WGBS — use nanoHiMe 6mA mode with input.peak.bed instead
- When comparing methylation callers rather than depths (e.g. benchmarking nanoHiMe
  vs Megalodon/Guppy accuracy) — subsampling depth scaling does not address that comparison
- When CpG sites lack >= 20x coverage by both WGBS and nanoHiMe-seq — the correlation
  analysis is undefined at those sites
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3295
tools:
- name: minimap2
  role: Aligns subsampled ONT reads to hg19 with the map-ont preset (v2.17, default
    settings)
  repo: github.com/YinLabTJ/nanoHiMe
- name: samtools
  role: Sorts/indexes BAM and filters mapped reads at MAPQ >= 20 (-q 20 -F 4)
- name: nanopolish
  role: Aligns electric current events to reference k-mers (v0.13.2, eventalign -n
    -t 16 --scale-events) per depth subsample
- name: nanoHiMe
  role: Calls CpG methylation in mCG mode with log-likelihood-ratio thresholds +1.5/-1.5
    from each depth's eventalign file
  repo: github.com/YinLabTJ/nanoHiMe
- name: seqkit
  role: Example read sampler for subsampling FASTQ to 5x, 10x, 30x, 50x coverage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:cefbe55eaad933b2d28bfca5efdb228113374ecc05f2c66276c10418f28dea95
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - minimap2
  - nanoHiMe
  - nanopolish
  - samtools
  - seqkit
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# nanopore-read-subsampling-coverage-scaling

## Summary

Subsample a basecalled nanoHiMe-seq FASTQ (HepG2) to 5x, 10x, 30x and 50x coverage, then run the full minimap2/nanopolish/nanoHiMe mCG-calling pipeline per depth and compute Pearson correlations vs ENCODE WGBS to establish the minimum depth for accurate CpG methylation measurement.

## When to use

Use when assessing the minimum nanopore sequencing depth at which nanoHiMe-seq CpG methylation calls remain accurate versus WGBS, e.g. before committing MinION flowcells for a low-coverage epigenomics experiment.

## When NOT to use

- When the question is histone-modification enrichment (6mA peak calling) rather than CpG methylation accuracy vs WGBS — use nanoHiMe 6mA mode with input.peak.bed instead
- When comparing methylation callers rather than depths (e.g. benchmarking nanoHiMe vs Megalodon/Guppy accuracy) — subsampling depth scaling does not address that comparison
- When CpG sites lack >= 20x coverage by both WGBS and nanoHiMe-seq — the correlation analysis is undefined at those sites

## Inputs

- Basecalled nanoHiMe-seq FASTQ reads (e.g. HepG2, H3K27me3/H3K4me3 experiment)
- FAST5 signal files for nanopolish indexing
- hg19 (UCSC) reference genome FASTA
- ENCODE WGBS methylation data (ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ, ENCFF369YQW)

## Outputs

- coverage_correlation.csv: sequencing depth vs Pearson correlation with WGBS (and 10x-vs-50x)
- Per-CpG scatterplot coordinates (nanoHiMe vs WGBS at 10x and 30x; 10x vs 50x)
- Per-depth eventalign files and output.mCG.eventalign.txt methylation calls

## How to apply

Subsample the basecalled FASTQ to 5x, 10x, 30x and 50x coverage with a read sampler (e.g. seqkit sample). For each subsample, align to hg19 with minimap2 -ax map-ont -t 8, sort/index with samtools, and retain mapped reads with MAPQ >= 20 via samtools view -b -q 20 -F 4. Run nanopolish eventalign with -n -t 16 --scale-events against hg19.fa, convert the reference to uppercase with nanoHiMe's upper.pl (samtools faidx), and call CpG methylation in mCG-calling mode (./nanoHiMe mCG ref_upper.fa eventalign output.mCG) using log-likelihood-ratio thresholds +1.5 (methylated) and -1.5 (unmethylated). Restrict correlation analysis to CpG sites with coverage depth >= 20 by both WGBS and nanoHiMe-seq and compute Pearson correlations of methylation levels against ENCODE WGBS (ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ, ENCFF369YQW) and between depths. Write the coverage-vs-correlation table to coverage_correlation.csv and dump per-CpG scatterplot coordinates (10x and 30x vs WGBS; 10x vs 50x) for plotting.

## Related tools

- **minimap2** (Aligns subsampled ONT reads to hg19 with the map-ont preset (v2.17, default settings)) — github.com/YinLabTJ/nanoHiMe
- **samtools** (Sorts/indexes BAM and filters mapped reads at MAPQ >= 20 (-q 20 -F 4))
- **nanopolish** (Aligns electric current events to reference k-mers (v0.13.2, eventalign -n -t 16 --scale-events) per depth subsample)
- **nanoHiMe** (Calls CpG methylation in mCG mode with log-likelihood-ratio thresholds +1.5/-1.5 from each depth's eventalign file) — github.com/YinLabTJ/nanoHiMe
- **seqkit** (Example read sampler for subsampling FASTQ to 5x, 10x, 30x, 50x coverage)

## Examples

```
minimap2 -ax map-ont -t 8 hg19.fa sub10x.fastq | samtools sort -o test.sorted.bam - && samtools view -b -q 20 -F 4 test.sorted.bam > test.q20.bam && nanopolish eventalign -n -t 16 --reads test.fastq --bam test.q20.bam --genome hg19.fa --scale-events > test.eventalign.txt && ./nanoHiMe mCG ref_upper.fa test.eventalign.txt output.mCG
```

## Evaluation signals

- 10x-vs-WGBS Pearson correlation is approximately 0.82
- 30x-vs-WGBS Pearson correlation is approximately 0.92 (near plateau)
- 10x-vs-50x correlation is 0.914
- All expected correlations recorded in coverage_correlation.csv
- Correlation analysis restricted to CpG sites with coverage depth >= 20 by both WGBS and nanoHiMe-seq

## Limitations

- Correlations plateau at ~30x; depths below 5x are not characterized and may degrade further
- Requires one or two MinION flowcells even at low coverage; per-locus power depends on read depth
- nanoHiMe 6mA/mCG calling over the whole genome can be quite slow unless limited by input.peak.bed

## Evidence

- [results] 0.82 at 10x coverage: [evidence span withheld — anchor status: not_found]
- [readme] mCG-calling mode: "mCG-calling mode: ./nanoHiMe mCG ref_upper.fa test.eventalign.txt(.gz) output.mCG"
- [readme] MAPQ >= 20 filter: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [discussion] coverage >= 20 filter: "calculated for the CpG sites with coverage depth ≥20 by both WGBS and nanoHiMe-seq data"
- [discussion] thresholds +1.5/-1.5: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs"
