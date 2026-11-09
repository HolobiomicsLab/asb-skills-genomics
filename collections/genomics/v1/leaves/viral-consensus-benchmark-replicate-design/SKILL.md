---
name: viral-consensus-benchmark-replicate-design
description: Use when you must demonstrate that a consensus-calling tool achieves
  accuracy parity with an established pipeline (e.g. iVar) across simulated Illumina/ONT
  conditions with replicate-level error statistics.
when_to_use_negative:
- Input is real (not simulated) sequencing data where no ground-truth reference genome
  exists to compute normalized Hamming error against
- Reference genomes are ultra-long (e.g. human chromosomes), since ViralConsensus
  memory consumption scales linearly with reference genome length
- The goal is performance (speed/memory) benchmarking rather than consensus accuracy
  parity testing
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_0622
tools:
- name: ViralConsensus
  role: Consensus caller under test; run at default settings on each alignment to
    produce FASTA consensus sequences
  repo: https://github.com/niemasd/ViralConsensus
- name: iVar
  role: Baseline consensus caller (v1.3.1, default settings) against which parity
    in normalized Hamming error is assessed
- name: minimap2
  role: Maps simulated reads to source lineage reference genomes using '-a -x sr'
    for short reads
- name: ART
  role: Simulates single-end HiSeq 2000 (Illumina) reads from each lineage reference
    genome
- name: NanoSim-H
  role: Simulates Oxford Nanopore reads from each lineage reference genome
- name: MAFFT
  role: Aligns each reconstructed consensus (v7.505) to its source reference genome
    for normalized Hamming error computation
- name: htslib
  role: Dependency of ViralConsensus for reading BAM/SAM alignment data
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:65079611f14d914f862c98d477610ba95f68411f1205b61cf4627a41b37f5eb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - ART
  - MAFFT
  - NanoSim-H
  - ViralConsensus
  - htslib
  - iVar
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# viral-consensus-benchmark-replicate-design

## Summary

A replicated simulation-based benchmarking design that compares viral consensus callers (e.g. ViralConsensus vs iVar) across SARS-CoV-2 lineages, sequencing technologies, and coverage levels using normalized Hamming error against the source reference genome.

## When to use

Use when you must demonstrate that a consensus-calling tool achieves accuracy parity with an established pipeline (e.g. iVar) across simulated Illumina/ONT conditions with replicate-level error statistics.

## When NOT to use

- Input is real (not simulated) sequencing data where no ground-truth reference genome exists to compute normalized Hamming error against
- Reference genomes are ultra-long (e.g. human chromosomes), since ViralConsensus memory consumption scales linearly with reference genome length
- The goal is performance (speed/memory) benchmarking rather than consensus accuracy parity testing

## Inputs

- SARS-CoV-2 lineage reference genomes in FASTA (e.g. NC_045512.2, LC650844, OQ523614.1, OQ194009.1, OQ346068.1)
- Simulated read sets: ART single-end HiSeq 2000 reads and NanoSim-H Oxford Nanopore reads (FASTQ)
- Alignments of simulated reads to source lineage references (SAM/BAM from minimap2 -a -x sr)

## Outputs

- Consensus genome FASTA files per replicate per condition for each consensus caller
- Normalized Hamming error per reconstructed consensus vs source reference (via MAFFT alignment)
- Aggregated TSV table of normalized Hamming errors across all lineages, coverages, and replicates for both callers

## How to apply

Simulate reads from each of five SARS-CoV-2 lineage reference genomes (B/NC_045512.2, B.1.1.7/LC650844, BA.1/OQ523614.1, BA.2/OQ194009.1, XBB.1/OQ346068.1) at 30x, 40x, and 50x coverage with 10 replicates per condition — single-end HiSeq 2000 reads via ART and Oxford Nanopore reads via NanoSim-H. Map each read set to its source lineage reference with minimap2 using '-a -x sr' for short reads. Call a consensus from each alignment with the tool under test (ViralConsensus v0.0.1, default settings) and with the baseline (iVar v1.3.1), writing FASTA outputs. Align each reconstructed consensus back to its source reference with MAFFT v7.505 and compute the normalized Hamming error. Aggregate errors across all 5 lineages x 3 coverages x 10 replicates x 2 simulators into a single TSV table and judge parity by comparable normalized Hamming error across conditions.

## Related tools

- **ViralConsensus** (Consensus caller under test; run at default settings on each alignment to produce FASTA consensus sequences) — https://github.com/niemasd/ViralConsensus
- **iVar** (Baseline consensus caller (v1.3.1, default settings) against which parity in normalized Hamming error is assessed)
- **minimap2** (Maps simulated reads to source lineage reference genomes using '-a -x sr' for short reads)
- **ART** (Simulates single-end HiSeq 2000 (Illumina) reads from each lineage reference genome)
- **NanoSim-H** (Simulates Oxford Nanopore reads from each lineage reference genome)
- **MAFFT** (Aligns each reconstructed consensus (v7.505) to its source reference genome for normalized Hamming error computation)
- **htslib** (Dependency of ViralConsensus for reading BAM/SAM alignment data) — https://github.com/samtools/htslib

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- Normalized Hamming error of ViralConsensus consensus sequences is comparable to iVar across all 300 simulated conditions (5 lineages x 3 coverages x 10 replicates x 2 simulators)
- The aggregated TSV contains error values for every replicate of every condition with no missing rows
- Each MAFFT alignment of a reconstructed consensus to its source reference completes and yields a computable Hamming distance
- Coverage conditions (30x, 40x, 50x) each contain exactly 10 replicates per lineage per simulator
- All consensus callers produce a full-length FASTA consensus sequence per input alignment

## Limitations

- Memory consumption of ViralConsensus scales linearly with reference genome length, so ultra-long references (e.g. human chromosomes) may require prohibitively large amounts of memory
- The web app (WebAssembly port) works well only for reasonably small datasets; larger datasets require the command-line tool
- Benchmark is grounded in simulated reads (ART/NanoSim-H) and SARS-CoV-2 lineages, so extrapolation to other viruses or real reads should be validated separately

## Evidence

- [other] ViralConsensus produces consensus sequences as accurate as the iVar pipeline, consistently across sequencing technologies, SARS-CoV-2 lineages, and sequencing coverage levels.: [evidence span withheld — anchor status: not_found]
- [methods] minimap2 sr mapping: "minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas"
- [abstract] tool summary: "ViralConsensus is a fast and memory-efficient tool for calling viral consensus genome sequences directly from read alignment data."
- [readme] memory scaling caveat: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
- [methods] consensus call command: "viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas"
