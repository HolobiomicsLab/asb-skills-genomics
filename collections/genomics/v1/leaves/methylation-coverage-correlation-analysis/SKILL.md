---
name: methylation-coverage-correlation-analysis
description: Use when you need to determine the minimum sequencing depth (e.g. 5x-50x)
  at which nanopore-based CpG methylation calls (nanoHiMe/nanopolish) still correlate
  with a bisulfite-based gold standard such as WGBS, or when planning MinION flowcell
  requirements.
when_to_use_negative:
- Input is not nanopore signal/sequence data (e.g. already-processed bisulfite calls
  alone) — the subsampling and event-level methylation calling require raw nanopore
  reads
- The comparison standard is not a methylation-level measurement per CpG (e.g. histone
  peak enrichment) — correlations of methylation levels vs coverage are not meaningful
  there
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: seqkit
  role: Subsamples basecalled FASTQ reads to 5x, 10x, 30x and 50x coverage
- name: minimap2
  role: Aligns subsampled nanopore reads to hg19 with the map-ont preset (-ax map-ont
    -t 8)
- name: samtools
  role: Sorts/indexes BAMs and filters to mapped MAPQ>=20 reads via samtools view
    -b -q 20 -F 4; also faidx-indexes the uppercased reference
- name: nanopolish
  role: Indexes reads against FAST5 signals and aligns nanopore electrical events
    to reference k-mers (nanopolish eventalign -n -t 16 --scale-events)
- name: nanoHiMe
  role: Calls CpG methylation from eventalign files in mCG mode using log-likelihood-ratio
    thresholds (+1.5 / -1.5)
  repo: github.com/YinLabTJ/nanoHiMe
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:a9982b8dc5f8b733a6a0c1813fff5fe74ad20cb72a50d468fdf6060921731869
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - Minimap2
  - minimap2
  - nanoHiMe
  - nanopolish
  - samtools
  - seqkit
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# methylation-coverage-correlation-analysis

## Summary

Assesses the minimum nanopore sequencing depth at which nanoHiMe-seq CpG methylation calls remain accurate by subsampling FASTQ reads to 5x-50x coverage, calling CpG methylation with nanoHiMe's HMM, and computing Pearson correlations against ENCODE WGBS methylation levels.

## When to use

Use when you need to determine the minimum sequencing depth (e.g. 5x-50x) at which nanopore-based CpG methylation calls (nanoHiMe/nanopolish) still correlate with a bisulfite-based gold standard such as WGBS, or when planning MinION flowcell requirements.

## When NOT to use

- Input is not nanopore signal/sequence data (e.g. already-processed bisulfite calls alone) — the subsampling and event-level methylation calling require raw nanopore reads
- The comparison standard is not a methylation-level measurement per CpG (e.g. histone peak enrichment) — correlations of methylation levels vs coverage are not meaningful there

## Inputs

- Basecalled nanopore FASTQ reads from nanoHiMe-seq (e.g. HepG2 H3K27me3)
- FAST5 signal files for nanopolish indexing
- hg19 reference genome FASTA (UCSC)
- ENCODE WGBS methylation calls (ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ, ENCFF369YQW)

## Outputs

- coverage_correlation.csv — coverage depth vs Pearson correlation table (5x, 10x, 30x, 50x vs WGBS; 10x vs 50x)
- Per-CpG scatterplot coordinates (nanoHiMe vs WGBS at 10x and 30x; 10x vs 50x)
- Per-depth nanoHiMe mCG methylation output files (output.mCG.eventalign.txt)

## How to apply

Subsample the basecalled FASTQ reads to target depths (5x, 10x, 30x, 50x) with a read sampler such as seqkit sample, then for each depth align to hg19 with minimap2 (-ax map-ont -t 8), sort/index with samtools, and filter to mapped reads with MAPQ >= 20 via 'samtools view -b -q 20 -F 4'. Run 'nanopolish eventalign -n -t 16 --scale-events' per subsample, convert the reference to uppercase with nanoHiMe's upper.pl and index with samtools faidx, then call CpG methylation in nanoHiMe mCG mode using log-likelihood-ratio thresholds +1.5 (methylated) and -1.5 (unmethylated). Restrict the correlation to CpG sites with coverage depth >= 20 by both WGBS and nanoHiMe-seq, compute Pearson correlations of each depth vs ENCODE WGBS, and write the coverage-vs-correlation table to coverage_correlation.csv. The rationale is that nanoHiMe-seq aims to profile methylation at low coverage with only one or two MinION flowcells, so the depth-correlation curve defines the sensitivity floor.

## Related tools

- **seqkit** (Subsamples basecalled FASTQ reads to 5x, 10x, 30x and 50x coverage)
- **minimap2** (Aligns subsampled nanopore reads to hg19 with the map-ont preset (-ax map-ont -t 8))
- **samtools** (Sorts/indexes BAMs and filters to mapped MAPQ>=20 reads via samtools view -b -q 20 -F 4; also faidx-indexes the uppercased reference)
- **nanopolish** (Indexes reads against FAST5 signals and aligns nanopore electrical events to reference k-mers (nanopolish eventalign -n -t 16 --scale-events))
- **nanoHiMe** (Calls CpG methylation from eventalign files in mCG mode using log-likelihood-ratio thresholds (+1.5 / -1.5)) — github.com/YinLabTJ/nanoHiMe

## Examples

```
./nanoHiMe mCG ref_upper.fa eventalign_10x.txt.gz output_10x.mCG
```

## Evaluation signals

- The 10x-vs-WGBS Pearson correlation is approximately 0.82 in coverage_correlation.csv
- The 30x-vs-WGBS Pearson correlation is approximately 0.92 (near-plateau)
- The 10x-vs-50x correlation is 0.914
- All correlation values are restricted to CpG sites with coverage depth >= 20 by both WGBS and nanoHiMe-seq
- Eventalign files were generated for every depth with mapped reads retained at MAPQ >= 20 (-q 20 -F 4)

## Limitations

- Only CpG sites with coverage depth >= 20 by both platforms contribute to the correlation, so very low-coverage CpGs are excluded from the accuracy assessment
- The correlation plateaus around 30x, meaning depths below ~10x may not provide accurate genome-wide CpG methylation measurements
- The 6mA calling module is genome-wide and may be slow unless peak regions (input.peak.bed) are supplied; this skill's mCG mode is not affected but eventalign generation can be a bottleneck

## Evidence

- [other] Subsample the basecalled HepG2 nanoHiMe-seq FASTQ reads to 5x, 10x, 30x and 50x coverage using a read sampler (e.g. seqkit sample).: [evidence span withheld — anchor status: not_found]
- [readme] Read filter with MAPQ threshold: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [discussion] mCG calling thresholds: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs"
- [discussion] CpG coverage filter for correlation: "calculated for the CpG sites with coverage depth ≥20 by both WGBS and nanoHiMe-seq data"
- [results] Low-coverage sensitivity finding: [evidence span withheld — anchor status: not_found]
