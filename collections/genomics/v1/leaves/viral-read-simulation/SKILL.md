---
name: viral-read-simulation
description: Use when benchmarking a viral genomics tool (e.g. a consensus caller)
  against a known ground-truth genome and you need simulated Illumina and/or Nanopore
  read sets at defined coverage levels (e.g. 30x, 40x, 50x) with replicates.
when_to_use_negative:
- You need real sequencing data rather than simulated reads with a known ground-truth
  genome (e.g. validating on clinical isolates with uncertain truth)
- The reference genome is ultra-long (e.g. human chromosomes), where downstream memory
  footprints scale linearly with reference length and become prohibitive
- You are simulating paired-end reads or non-SARS-CoV-2 error profiles without matching
  simulator models (the benchmarked design uses single-end HiSeq 2000 and Nanopore
  models)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3194
tools:
- name: ART
  role: Simulates single-end HiSeq 2000 (Illumina) reads from each SARS-CoV-2 lineage
    reference genome at 30x, 40x, and 50x coverage with 10 replicates per condition
- name: NanoSim-H
  role: Simulates Oxford Nanopore reads from the same five SARS-CoV-2 lineage genomes
    at the same coverage levels and replicate counts
- name: minimap2
  role: Maps each simulated read set to its source lineage reference genome (-a -x
    sr for short reads) to produce alignments for consensus calling
- name: htslib
  role: Underlying library for reading/writing SAM/BAM alignment data consumed by
    the benchmark's alignment and consensus-calling steps
  repo: https://github.com/samtools/htslib
- name: ViralConsensus
  role: Downstream consumer of the simulated-read alignments; calls consensus genomes
    that are scored against the known source reference
  repo: https://github.com/niemasd/ViralConsensus
- name: MAFFT
  role: Aligns each reconstructed consensus to its source reference genome for computing
    normalized Hamming error
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:828c039cfdd9fecb14d8029b291f727231f5d47621a4c0dd5f7239d116638d83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/samtools/htslib
  tools:
  - ART
  - MAFFT
  - NanoSim-H
  - ViralConsensus
  - htslib
  - minimap2
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# viral-read-simulation

## Summary

Simulate viral sequencing reads in silico from reference genomes (single-end HiSeq 2000 Illumina reads with ART and Oxford Nanopore reads with NanoSim-H) at controlled coverage levels to generate reproducible benchmark datasets for evaluating downstream tools such as consensus callers.

## When to use

Use when benchmarking a viral genomics tool (e.g. a consensus caller) against a known ground-truth genome and you need simulated Illumina and/or Nanopore read sets at defined coverage levels (e.g. 30x, 40x, 50x) with replicates.

## When NOT to use

- You need real sequencing data rather than simulated reads with a known ground-truth genome (e.g. validating on clinical isolates with uncertain truth)
- The reference genome is ultra-long (e.g. human chromosomes), where downstream memory footprints scale linearly with reference length and become prohibitive
- You are simulating paired-end reads or non-SARS-CoV-2 error profiles without matching simulator models (the benchmarked design uses single-end HiSeq 2000 and Nanopore models)

## Inputs

- SARS-CoV-2 lineage reference genome FASTA files (e.g. NC_045512.2, LC650844, OQ523614.1, OQ194009.1, OQ346068.1)
- ART simulator configuration for single-end HiSeq 2000 reads
- NanoSim-H configuration for Oxford Nanopore reads
- Coverage/replicate design parameters (30x, 40x, 50x; 10 replicates per condition)

## Outputs

- Simulated FASTQ read files per lineage x coverage x replicate for each sequencing technology (Illumina via ART, ONT via NanoSim-H)
- Read alignments (SAM/BAM) produced by mapping simulated reads to their source reference with minimap2 -a -x sr
- Aggregatable benchmark table inputs, e.g. normalized Hamming errors per condition after consensus calling and MAFFT alignment

## How to apply

Start from reference genomes — for SARS-CoV-2 benchmarks the paper's task uses five lineage references (B/NC_045512.2, B.1.1.7/LC650844, BA.1/OQ523614.1, BA.2/OQ194009.1, XBB.1/OQ346068.1). Simulate single-end HiSeq 2000 reads with ART and Oxford Nanopore reads with NanoSim-H, at each target coverage level (30x, 40x, 50x) with 10 replicates per lineage x coverage x simulator condition, so accuracy comparisons can be aggregated over 5 lineages x 3 coverages x 10 replicates x 2 simulators. Map each simulated read set back to its source lineage reference with minimap2 using short-read presets (-a -x sr for Illumina). The resulting alignments (BAM/SAM via htslib/samtools tooling) feed downstream tools; because ViralConsensus can read mapper output piped via stdin (e.g. 'minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas'), simulation-to-consensus can run without intermediate disk I/O. Judge the simulation setup correct when every lineage x coverage x replicate x simulator cell is present and each read set maps cleanly to its own source reference.

## Related tools

- **ART** (Simulates single-end HiSeq 2000 (Illumina) reads from each SARS-CoV-2 lineage reference genome at 30x, 40x, and 50x coverage with 10 replicates per condition)
- **NanoSim-H** (Simulates Oxford Nanopore reads from the same five SARS-CoV-2 lineage genomes at the same coverage levels and replicate counts)
- **minimap2** (Maps each simulated read set to its source lineage reference genome (-a -x sr for short reads) to produce alignments for consensus calling)
- **htslib** (Underlying library for reading/writing SAM/BAM alignment data consumed by the benchmark's alignment and consensus-calling steps) — https://github.com/samtools/htslib
- **ViralConsensus** (Downstream consumer of the simulated-read alignments; calls consensus genomes that are scored against the known source reference) — https://github.com/niemasd/ViralConsensus
- **MAFFT** (Aligns each reconstructed consensus to its source reference genome for computing normalized Hamming error)

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- Completeness check: the simulation grid contains 5 lineages x 3 coverages (30x/40x/50x) x 10 replicates x 2 simulators (ART HiSeq 2000 and NanoSim-H ONT) = 300 read sets
- Each simulated read set maps back to its own source lineage reference genome (not a different lineage's reference) with minimap2 -a -x sr for Illumina reads
- Coverage per position approximates the target depth (30x/40x/50x) so consensus accuracy reflects coverage effects rather than gaps
- Downstream sanity: consensus sequences reconstructed from simulated reads of a reference should show low normalized Hamming error against that reference, enabling a parity check between competing callers (e.g. ViralConsensus vs iVar)
- Aggregate outputs form a single TSV of normalized Hamming errors across all lineages, coverages, and replicates per tool

## Limitations

- Simulated reads only approximate real error profiles; benchmark conclusions are conditional on ART's HiSeq 2000 model and NanoSim-H's Nanopore model
- Coverage levels tested (30x, 40x, 50x) do not span low-coverage or very-high-coverage regimes
- Memory of downstream consensus tools scales linearly with reference genome length, so the simulation-to-consensus pipeline does not transfer directly to ultra-long genomes (e.g. human chromosomes)
- The design is single-end Illumina; paired-end or other platforms would require different simulator configurations

## Evidence

- [other] Simulation design spans lineages, coverages, and simulators: [evidence span withheld — anchor status: not_found]
- [other] ONT simulation arm: [evidence span withheld — anchor status: not_found]
- [other] Mapping step: [evidence span withheld — anchor status: not_found]
- [abstract] On-the-fly piping from mapper: "unlike existing methods, ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly"
- [methods] Example pipe invocation: "minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas"
- [intro] Memory scaling caveat: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
