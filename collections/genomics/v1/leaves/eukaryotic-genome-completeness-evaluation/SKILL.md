---
name: eukaryotic-genome-completeness-evaluation
description: Use when judging how complete a hybrid-scaffolded eukaryotic genome assembly
  is, e.g. comparing contig count and N50 (npScarf 19 contigs / 913 Kb on S. cerevisiae
  W303) against competitor scaffolders via QUAST.
when_to_use_negative:
- The sample is prokaryotic/bacterial — the completeness statistic is described as
  stable for bacterial genomes (up to 20 largest contigs > 20 Kb), so eukaryotic-scale
  evaluation logic and the circularized-mitochondrial check do not apply.
- Input is already a finished or reference-complete assembly and no competitor scaffolds
  exist to compare — QUAST-vs-next-best-method reasoning adds nothing.
- Nanopore data are raw signal without base-calling and no real-time base-caller (e.g.
  Nanocall, DeepNano) is available, so the streaming scaffold feed cannot be produced.
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: npScarf
  role: Real-time scaffolding and completion of the short-read draft assembly; produces
    .fin.fasta/.fin.japsa whose contig count and N50 are the completeness metrics
  repo: https://github.com/mdcao/npScarf
- name: SPAdes
  role: Assembles Illumina MiSeq paired-end reads (--careful, 16 threads) into the
    draft contigs that npScarf scaffolds
- name: BWA-MEM
  role: Aligns the nanopore stream to the draft assembly with the fixed parameter
    set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000
- name: jsa.np.timeEmulate
  role: Emulates the real-time nanopore stream (reads sorted by timestamp, -s 100)
    for offline reproduction
  repo: https://github.com/mdcao/japsa
- name: jsa.seq.sort
  role: Sorts SPAdes contigs by length ('-r -n') to produce the draft assembly
  repo: https://github.com/mdcao/japsa
- name: QUAST
  role: Generates the comparative assembly report (quast_w303_report.html) used to
    judge completeness against the next best method
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:76d694ab34fb1eab4c54de4c95a9ae73ca1703abad56dd8965f9f5dcf0edffb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - QUAST
  - SPAdes
  - jsa.np.timeEmulate
  - jsa.seq.sort
  - npScarf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# eukaryotic-genome-completeness-evaluation

## Summary

Evaluates the completeness of a real-time hybrid-scaffolded eukaryotic assembly by comparing contig number and N50 against the next best scaffolding method, and by checking hallmarks such as a circularized mitochondrial contig. Applied to S. cerevisiae W303, where npScarf achieved 19 contigs with N50 913 Kb versus 108 contigs (N50 600 Kb) for Nanocorr.

## When to use

Use when judging how complete a hybrid-scaffolded eukaryotic genome assembly is, e.g. comparing contig count and N50 (npScarf 19 contigs / 913 Kb on S. cerevisiae W303) against competitor scaffolders via QUAST.

## When NOT to use

- The sample is prokaryotic/bacterial — the completeness statistic is described as stable for bacterial genomes (up to 20 largest contigs > 20 Kb), so eukaryotic-scale evaluation logic and the circularized-mitochondrial check do not apply.
- Input is already a finished or reference-complete assembly and no competitor scaffolds exist to compare — QUAST-vs-next-best-method reasoning adds nothing.
- Nanopore data are raw signal without base-calling and no real-time base-caller (e.g. Nanocall, DeepNano) is available, so the streaming scaffold feed cannot be produced.

## Inputs

- Illumina MiSeq paired-end FASTQ reads for the eukaryotic sample (e.g. S. cerevisiae W303)
- Oxford Nanopore long reads (base-called FASTQ/FASTA, timestamped for real-time emulation)
- Draft SPAdes assembly FASTA (spades/contigs.fasta sorted by length)
- Competing-method assemblies (e.g. Nanocorr scaffolds) for QUAST comparison
- Reference genome for QUAST evaluation

## Outputs

- <prefix>.fin.fasta — final npScarf scaffolds
- <prefix>.fin.japsa — JAPSA-format scaffolding result
- QUAST comparison report (quast_w303_report.html) with contig number, N50 and other assembly statistics

## How to apply

Build the draft assembly from Illumina MiSeq paired-end reads with SPAdes --careful (16 threads), sort contigs by length with 'jsa.seq.sort -r -n', and index the draft with 'bwa index'. Emulate the nanopore stream with 'jsa.np.timeEmulate -s 100' (reads ordered by timestamp), pipe through BWA-MEM with the fixed parameter set '-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000' into 'jsa.np.npscarf -realtime -format sam -seq <draft>'. Take the final scaffolds from <prefix>.fin.fasta, verify the mitochondrial contig is circularized, and check contig number and N50 against the reference values (19 contigs, N50 ≈ 913 Kb for W303). Run QUAST on the scaffolds together with competitive-method assemblies (e.g. Nanocorr at 108 contigs, N50 600 Kb) and save the comparison report as quast_w303_report.html; completeness is judged by fewer contigs and higher N50 relative to the next best method.

## Related tools

- **npScarf** (Real-time scaffolding and completion of the short-read draft assembly; produces .fin.fasta/.fin.japsa whose contig count and N50 are the completeness metrics) — https://github.com/mdcao/npScarf
- **SPAdes** (Assembles Illumina MiSeq paired-end reads (--careful, 16 threads) into the draft contigs that npScarf scaffolds)
- **BWA-MEM** (Aligns the nanopore stream to the draft assembly with the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000)
- **jsa.np.timeEmulate** (Emulates the real-time nanopore stream (reads sorted by timestamp, -s 100) for offline reproduction) — https://github.com/mdcao/japsa
- **jsa.seq.sort** (Sorts SPAdes contigs by length ('-r -n') to produce the draft assembly) — https://github.com/mdcao/japsa
- **QUAST** (Generates the comparative assembly report (quast_w303_report.html) used to judge completeness against the next best method)

## Examples

```
jsa.np.timeEmulate -s 100 <ONT.fastq> | bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 W303_spades.fasta - | jsa.np.npscarf -realtime -format sam -seq W303_spades.fasta -prefix W303-realtime
```

## Evaluation signals

- QUAST report reproduces the paper-reported statistics: 19 contigs, N50 ≈ 913 Kb, and a circularized mitochondrial contig for npScarf on W303
- Contig count and N50 beat the next best method (Nanocorr: 108 contigs, N50 600 Kb)
- <prefix>.fin.fasta and <prefix>.fin.japsa are produced by the real-time run with the chosen output prefix
- The mitochondrial contig is circularized in the final scaffolds
- QUAST report matches reference values published at the npScarf results site (QUAST/W303/report.html)

## Limitations

- The pipeline is real-time only at minute-level temporal resolution, not millisecond resolution.
- It still requires short read data, which cannot be generated in real-time with current technology.
- Latency depends on availability of fast real-time base-callers such as Nanocall and DeepNano.
- npScarf is no longer maintained; npGraph (github.com/hsnguyen/assembly) is the successor.

## Evidence

- [abstract] npScarf abstract claim: "Here we present npScarf, which can scaffold and complete short read assemblies while the long read sequencing run is in progress."
- [discussion] npScarf streaming alignment method: "npScarf employs a mapping method that supports streaming processing such as BWA-MEM40 to align a small number of long reads to the existing assembly as they arrive."
- [discussion] Completeness statistic stability caveat: "In our experience, the statistic is stable with up to 20 of the largest contigs longer than 20 Kb, which are most likely unique contigs in bacterial genomes38."
- [readme] npScarf tool definition: "npScarf (jsa.np.npscarf) is a program that scaffolds and completes draft genomes assemblies in real-time with Oxford Nanopore sequencing."
