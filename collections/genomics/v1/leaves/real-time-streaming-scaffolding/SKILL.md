---
name: real-time-streaming-scaffolding
description: Use when nanopore reads arrive incrementally during a MinION run and
  you want to merge SPAdes/Illumina contigs into completed scaffolds on-the-fly, using
  streaming bwa mem alignments piped into jsa.np.npscarf -realtime.
when_to_use_negative:
- The long-read data are already fully sequenced and base-called and you do not need
  incremental assembly metrics — use npScarf in batch mode (same bwa mem parameters,
  no -realtime) instead
- The draft assembly consists of long-read-only contigs or the input is already a
  finished/scaffolded assembly — npScarf scaffolds short-read draft assemblies from
  SPAdes-style contigs
- npScarf is unmaintained and you are starting a new pipeline rather than reproducing
  this method — the README points to npGraph (github.com/hsnguyen/assembly) as the
  replacement
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: npScarf
  role: Scaffolds and completes the draft in real time from the streamed SAM alignments;
    invoked as jsa.np.npscarf -realtime
  repo: https://github.com/mdcao/npScarf
- name: npReader
  role: Streams base-called nanopore reads from the Downloads folder in real time
    (--realtime --folder Downloads --fail --stat --number --output -)
  repo: https://github.com/mdcao/npReader
- name: BWA-MEM
  role: Aligns incoming long-read blocks to the draft contigs with the fixed parameter
    set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000
- name: japsa
  role: Package bundling jsa.np.npscarf, jsa.np.npreader, and jsa.np.timeEmulate utilities;
    installed via git clone and make install
  repo: https://github.com/mdcao/japsa
- name: SPAdes
  role: Assembles trimmed Illumina paired-end reads into the draft contigs (>= 3.5
    required)
- name: QUAST
  role: Evaluates final scaffolds (contig count, N50, completeness) against the pre-scaffolding
    draft
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:a85a172f6318a921664ceb65f69992f4da43188f2e3349c8b58a6ea50ffab638
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - QUAST
  - SPAdes
  - japsa
  - npReader
  - npScarf
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# real-time-streaming-scaffolding

## Summary

Scaffold and complete a short-read draft genome assembly using Oxford Nanopore long reads streamed during an ongoing sequencing run, via npScarf (jsa.np.npscarf), so assembly metrics are reported in real time and the run can be terminated once sufficient quality is reached.

## When to use

Use when nanopore reads arrive incrementally during a MinION run and you want to merge SPAdes/Illumina contigs into completed scaffolds on-the-fly, using streaming bwa mem alignments piped into jsa.np.npscarf -realtime.

## When NOT to use

- The long-read data are already fully sequenced and base-called and you do not need incremental assembly metrics — use npScarf in batch mode (same bwa mem parameters, no -realtime) instead
- The draft assembly consists of long-read-only contigs or the input is already a finished/scaffolded assembly — npScarf scaffolds short-read draft assemblies from SPAdes-style contigs
- npScarf is unmaintained and you are starting a new pipeline rather than reproducing this method — the README points to npGraph (github.com/hsnguyen/assembly) as the replacement

## Inputs

- SPAdes contigs FASTA from Illumina MiSeq paired-end reads (sorted descending by length with jsa.seq.sort -r -n, bwa-indexed)
- Base-called nanopore FASTQ/FASTQ.gz reads, streamed (npReader on Downloads folder) or timestamp-sorted file emulated with jsa.np.timeEmulate -s 100
- SAM stream from bwa mem using the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y [-K 3000]

## Outputs

- <prefix>.fin.fasta — final real-time scaffolded assembly in FASTA
- <prefix>.fin.japsa — scaffolding result in JAPSA format
- <prefix>.anno.japsa — detected features (e.g. plasmid/pathogenicity-island AMR genes) when annotation analysis is enabled in realtime mode
- QUAST report of contig count, N50, and completeness for the final scaffolds

## How to apply

Assemble trimmed Illumina paired-end reads with SPAdes (--careful, e.g. -t 16), sort contigs descending by length with jsa.seq.sort -r -n, and build a bwa index on the sorted FASTA. Stream base-called nanopore reads with jsa.np.npreader --realtime --folder Downloads --fail --stat --number --output - (or emulate with jsa.np.timeEmulate -s 100 on timestamp-sorted reads), align incoming blocks block-wise with bwa mem using the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000, and pipe the SAM stream into jsa.np.npscarf -realtime -input - -format sam -seq <draft.fasta> -prefix <prefix>. npScarf determines contig multiplicity by comparing per-contig short-read coverage to whole-genome coverage (uniqueness threshold θ = 1.5), builds a backbone from unique contigs via bridges of reads aligning to two unique contigs, merges contigs greedily (Kruskal-based) once a bridge reaches a pre-defined scoring threshold, fills gaps with the consensus of bridging reads, inserts repetitive contigs aligned to the gap consensus, and breaks the lowest-scoring bridge when a new bridge contradicts previous ones. Evaluate the resulting prefix.fin.fasta with QUAST, comparing contig count and N50 against the pre-scaffolding draft (e.g. S. Typhi H58: 34 contigs → 9 real-time/8 batch contigs, N50 ~864 kb) and the published reports at data.genomicsresearch.org/Projects/npScarf.

## Related tools

- **npScarf** (Scaffolds and completes the draft in real time from the streamed SAM alignments; invoked as jsa.np.npscarf -realtime) — https://github.com/mdcao/npScarf
- **npReader** (Streams base-called nanopore reads from the Downloads folder in real time (--realtime --folder Downloads --fail --stat --number --output -)) — https://github.com/mdcao/npReader
- **BWA-MEM** (Aligns incoming long-read blocks to the draft contigs with the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000)
- **japsa** (Package bundling jsa.np.npscarf, jsa.np.npreader, and jsa.np.timeEmulate utilities; installed via git clone and make install) — https://github.com/mdcao/japsa
- **SPAdes** (Assembles trimmed Illumina paired-end reads into the draft contigs (>= 3.5 required))
- **QUAST** (Evaluates final scaffolds (contig count, N50, completeness) against the pre-scaffolding draft)

## Examples

```
jsa.np.npreader --realtime --folder Downloads --fail --stat --number --output - | bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 Kp2146_spades.fasta - | jsa.np.npscarf -realtime -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-realtime
```

## Evaluation signals

- prefix.fin.fasta exists after the run and QUAST shows the draft collapsed to the expected contig count (e.g. K. pneumoniae Kpn2146 → 5 contigs; S. Typhi H58 → 9 contigs real-time, 8 batch)
- N50 of the real-time scaffolds matches the published QUAST report (e.g. S. Typhi H58 N50 = 864 kb; Kpn2146 N50 5.4 Mb with no misassemblies)
- Real-time and batch scaffold metrics agree closely; real-time should approach the batch result (9 vs 8 contigs for S. Typhi H58)
- Uniqueness statistic uses up to 20 of the largest contigs longer than 20 kb and multiplicity threshold θ = 1.5; contigs declared unique should have contig coverage within θ-fold of the genome depth estimate
- QUAST output consistent with the deposited report at data.genomicsresearch.org/Projects/npScarf/results/QUAST/<sample>/report.html

## Limitations

- Real-time is at the temporal resolution of minutes, not the millisecond level required to update with each base added
- The pipeline still requires short-read data, which cannot be generated in real-time with current technology, and relies on a base-caller whose latency depends on fast real-time base-callers (Nanocall, DeepNano)
- The bridge scoring threshold is only described as 'pre-defined' with no value given, and θ = 1.5 was set from authors' experience rather than a principled derivation
- npScarf is no longer maintained; npGraph (hsnguyen/assembly) is the replacement, requiring Java 11+ and preferably minimimap2
- Real-time npReader requires npReader and the HDF library installed properly

## Evidence

- [abstract] Here we present npScarf, which can scaffold and complete short read assemblies while the long read sequencing run is in progress.: "Here we present npScarf, which can scaffold and complete short read assemblies while the long read sequencing run is in progress."
- [discussion] npScarf employs a mapping method that supports streaming processing such as BWA-MEM40 to align a small number of long reads to the existing assembly as they arrive.: "npScarf employs a mapping method that supports streaming processing such as BWA-MEM40 to align a small number of long reads to the existing assembly as they arrive."
- [discussion] npScarf uses a greedy strategy based on Kruskal's algorithm39: "npScarf uses a greedy strategy based on Kruskal's algorithm39"
- [readme] bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 Kp2146_spades.fasta -: "| bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 Kp2146_spades.fasta -"
- [readme] Normally the tool generate two files: prefix.fin.fasta and prefix.fin.japsa which indicate the result scaffolders in FASTA and JAPSA format.: "Normally the tool generate two files: *prefix*.fin.fasta and *prefix*.fin.japsa which indicate the result scaffolders in FASTA and JAPSA format."
