---
name: realtime-streaming-assembly-pipelining
description: Use when base-called nanopore reads are arriving during an active run
  (or emulated stream) and you need a hybrid SPAdes+npScarf scaffolded assembly with
  live metrics to decide when to stop sequencing.
when_to_use_negative:
- The nanopore data are already fully base-called and you want the fastest/best final
  assembly rather than in-run scaffolding and termination decisions — batch mode or
  a dedicated long-read assembler (npScarf is unmaintained; npGraph is its successor)
  is more appropriate.
- There are no short-read data and none can be generated — the pipeline scaffolds
  a short-read draft assembly and cannot work from long reads alone.
- Millisecond-level real-time updates on every added base are required — the pipeline
  is real-time only at minute-level temporal resolution and depends on base-caller
  latency.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: npScarf (jsa.np.npscarf)
  role: 'Core streaming scaffolder: consumes SAM alignments block-wise, scores/merges
    contig bridges, fills gaps, emits <prefix>.fin.fasta/.fin.japsa in real-time mode'
  repo: https://github.com/mdcao/npScarf
- name: Japsa (jsa.np.npreader, jsa.seq.sort, jsa.np.timeEmulate)
  role: Provides the npScarf module, the sort utility for the draft contigs, and real-time
    nanopore read streaming (or time-emulation for replay)
  repo: https://github.com/mdcao/japsa
- name: npReader
  role: Real-time conversion of MinION HDF data into a read stream feeding BWA-MEM
    (requires HDF/JHI5 library installed)
  repo: https://github.com/mdcao/npReader
- name: SPAdes
  role: Assembles the trimmed Illumina paired-end reads (--careful, 16 threads) into
    the draft assembly spades/contigs.fasta that npScarf scaffolds
- name: BWA-MEM
  role: Streaming-capable aligner used with the fixed parameter set -k11 -W20 -r10
    -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 to map arriving long reads to the draft
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:4aa0381fd6ebbd6a716bb5c0f1431c183fe544e60eed59c5cdd1f350175164b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - Japsa (jsa.np.npreader, jsa.seq.sort, jsa.np.timeEmulate)
  - SPAdes
  - jsa.np.timeEmulate
  - jsa.seq.sort
  - npReader
  - npScarf
  - npScarf (jsa.np.npscarf)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# realtime-streaming-assembly-pipelining

## Summary

Run npScarf (Japsa) to scaffold and complete a short-read draft assembly while a nanopore run is still in progress, streaming base-called long reads block-wise through BWA-MEM and merging contigs as bridge scores cross threshold. Reports assembly metrics (contigs, N50) in real-time so sequencing can be terminated once sufficient quality is reached.

## When to use

Use when base-called nanopore reads are arriving during an active run (or emulated stream) and you need a hybrid SPAdes+npScarf scaffolded assembly with live metrics to decide when to stop sequencing.

## When NOT to use

- The nanopore data are already fully base-called and you want the fastest/best final assembly rather than in-run scaffolding and termination decisions — batch mode or a dedicated long-read assembler (npScarf is unmaintained; npGraph is its successor) is more appropriate.
- There are no short-read data and none can be generated — the pipeline scaffolds a short-read draft assembly and cannot work from long reads alone.
- Millisecond-level real-time updates on every added base are required — the pipeline is real-time only at minute-level temporal resolution and depends on base-caller latency.

## Inputs

- Draft assembly FASTA from SPAdes on trimmed Illumina MiSeq paired-end reads (spades/contigs.fasta, sorted by jsa.seq.sort)
- BWA index of the draft assembly
- Base-called nanopore long reads in FASTQ/FASTA, either live from npReader/MinION or batch FASTQ emulated with jsa.np.timeEmulate

## Outputs

- <prefix>.fin.fasta — final scaffolded/completed assembly in FASTA
- <prefix>.fin.japsa — scaffolds in JAPSA format
- <prefix>.anno.japsa — detected features (when real-time annotation analysis is enabled)
- Real-time assembly metrics (contig count, N50) used to decide sequencing termination

## How to apply

Assemble trimmed Illumina MiSeq paired-end reads (300 bp PE, e.g. >250-fold coverage) with SPAdes (--careful, ~16 threads) and sort spades/contigs.fasta by length with 'jsa.seq.sort -r -n' to produce the draft; index it with 'bwa index'. Stream base-called nanopore reads (via jsa.np.npreader in real-time, or jsa.np.timeEmulate -s 100 to emulate) into BWA-MEM with the fixed parameter set '-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000', piping SAM into 'jsa.np.npscarf -realtime -format sam -seq <draft>'. npScarf scores bridges between unique contigs (multiplicity threshold θ=1.5, comparing per-contig short-read coverage to genome depth), merges greedily Kruskal-style once a bridge reaches its scoring threshold, fills gaps with read consensus, and breaks the lowest-scoring contradicting bridge on conflict. Retrieve the final scaffolds from <prefix>.fin.fasta (and <prefix>.fin.japsa); rerun with --help first since parameters can change significantly between versions.

## Related tools

- **npScarf (jsa.np.npscarf)** (Core streaming scaffolder: consumes SAM alignments block-wise, scores/merges contig bridges, fills gaps, emits <prefix>.fin.fasta/.fin.japsa in real-time mode) — https://github.com/mdcao/npScarf
- **Japsa (jsa.np.npreader, jsa.seq.sort, jsa.np.timeEmulate)** (Provides the npScarf module, the sort utility for the draft contigs, and real-time nanopore read streaming (or time-emulation for replay)) — https://github.com/mdcao/japsa
- **npReader** (Real-time conversion of MinION HDF data into a read stream feeding BWA-MEM (requires HDF/JHI5 library installed)) — https://github.com/mdcao/npReader
- **SPAdes** (Assembles the trimmed Illumina paired-end reads (--careful, 16 threads) into the draft assembly spades/contigs.fasta that npScarf scaffolds)
- **BWA-MEM** (Streaming-capable aligner used with the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 to map arriving long reads to the draft)

## Examples

```
jsa.np.timeEmulate -s 100 | bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 draft.fasta - | jsa.np.npscarf -realtime -format sam -seq draft.fasta -prefix run > log.out 2>&1
```

## Evaluation signals

- Both output files exist with the chosen -prefix: prefix.fin.fasta (FASTA scaffolds) and prefix.fin.japsa (JAPSA format)
- For the S. cerevisiae W303 benchmark, the QUAST report reproduces 19 contigs with N50 ≈ 913 Kb and a circularized mitochondrial contig (vs next-best Nanocorr: 108 contigs, N50 = 600 Kb)
- Bridge merging occurs only after bridges reach the pre-defined scoring threshold; contradictions cause the lowest-scoring previous bridge to be broken in favor of the current one
- Contig multiplicity classification is stable when the statistic is computed over up to the 20 largest contigs longer than 20 Kb
- Metrics update at minute-level temporal resolution during the run, allowing termination once assembly quality is sufficient

## Limitations

- npScarf is no longer maintained; npGraph (hsnguyen/assembly) is the replacement
- Real-time is at minute-level resolution, not per-base/millisecond level; latency also depends on base-caller speed (e.g. Nanocall, DeepNano)
- The pipeline still requires short-read data, which cannot be generated in real-time with current technology
- The bridge scoring threshold is 'pre-defined' but its value is not stated in the paper; θ=1.5 multiplicity threshold comes from the authors' experience
- The README warns the option structure/parameter list can change significantly between versions — always check jsa.np.npscarf --help before running
- Running truly live requires npReader and the HDF library installed correctly; a Linux/Java 8+ environment is needed (Japsa requires Java 8 to compile/run from source)

## Evidence

- [discussion] Streaming alignment and threshold-triggered merging: "npScarf employs a mapping method that supports streaming processing such as BWA-MEM40 to align a small number of long reads to the existing assembly as they arrive."
- [abstract] Real-time scaffolding claim: "Here we present npScarf, which can scaffold and complete short read assemblies while the long read sequencing run is in progress."
- [abstract] Termination rationale: "It reports assembly metrics in real-time so the sequencing run can be terminated once an assembly of sufficient quality is obtained."
- [readme] Fixed BWA-MEM parameters: "bwa mem -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y <*draft*> <*nanopore*>"
- [readme] Realtime command-line pattern: "jsa.np.npscarf -realtime -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-realtime > log.out 2>&1"
- [readme] Maintenance warning: "Note: *npScarf* is not on maintenance anymore, instead [*npGraph*](https://github.com/hsnguyen/assembly/blob/master/docs/npgraph.md) is under development and would be the replacement."
- [readme] Version-change warning: "Please always check the help option first before running *npScarf* since the structure and parameters list of the command can be changed significantly from different versions."
