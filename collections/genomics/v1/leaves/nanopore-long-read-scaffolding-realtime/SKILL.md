---
name: nanopore-long-read-scaffolding-realtime
description: Use when a SPAdes short-read draft assembly needs finishing with nanopore
  long reads, especially when streaming MinION data in real time or when minimizing
  long-read coverage (e.g. ~30-fold for E. coli, ~14-fold for Kpn2146 plasmids).
when_to_use_negative:
- The input long reads are already error-corrected and intended for a de novo long-read
  assembler (e.g. Celera Assembler) rather than scaffolding a short-read draft
- Only short-read data are available and no nanopore long reads can be generated or
  streamed
- npScarf is unmaintained and a modern streaming assembler is required — the README
  directs users to npGraph as the replacement
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: npScarf
  role: 'Core scaffolder: consumes SAM stream from BWA-MEM to scaffold, complete,
    and annotate the draft assembly in real time'
  repo: https://github.com/mdcao/npScarf
- name: npReader
  role: Streams base-called MinION reads in real time from the run folder into the
    alignment step (jsa.np.npreader --realtime)
  repo: https://github.com/mdcao/npReader
- name: BWA-MEM
  role: Aligns incoming nanopore reads block-wise to the draft assembly with fixed
    parameters (-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y) supporting streaming
- name: SPAdes
  role: Assembles trimmed Illumina paired-end reads into the draft contigs that npScarf
    scaffolds
- name: trimmomatic
  role: Trims Illumina adapters and low-quality bases (V0.32) before SPAdes assembly
- name: QUAST
  role: Evaluates the final scaffold against the reference genome (contig count, assembly
    size)
- name: japsa
  role: Package bundling jsa.np.npscarf, jsa.np.npreader, jsa.seq.sort, and jsa.np.timeEmulate
    utilities
  repo: https://github.com/mdcao/japsa
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:2ebc8c852655b5ed1a4a92272d713743567887386b494630a838c5384b2081fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - QUAST
  - SPAdes
  - japsa
  - jsa.np.npreader
  - jsa.np.timeEmulate
  - jsa.seq.sort
  - npReader
  - npScarf
  - trimmomatic
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# nanopore-long-read-scaffolding-realtime

## Summary

Scaffold and complete a short-read draft assembly in real time using streaming Oxford Nanopore long reads with npScarf (jsa.np.npscarf), reporting assembly metrics during the sequencing run so the run can be terminated once sufficient quality is reached.

## When to use

Use when a SPAdes short-read draft assembly needs finishing with nanopore long reads, especially when streaming MinION data in real time or when minimizing long-read coverage (e.g. ~30-fold for E. coli, ~14-fold for Kpn2146 plasmids).

## When NOT to use

- The input long reads are already error-corrected and intended for a de novo long-read assembler (e.g. Celera Assembler) rather than scaffolding a short-read draft
- Only short-read data are available and no nanopore long reads can be generated or streamed
- npScarf is unmaintained and a modern streaming assembler is required — the README directs users to npGraph as the replacement

## Inputs

- Trimmed Illumina MiSeq paired-end FASTQ reads (e.g. ENA ERR654977 for E. coli K12 MG1655; Kp2146_paired_1/2.fastq.gz)
- MinION nanopore reads in FASTQ/FASTA (e.g. ENA ERR637419 R7.3 run at 67-fold coverage; ENA ERR868296 for Kpn2146)
- SPAdes draft assembly FASTA (contigs.fasta, length-sorted)
- Optional FASTA databases: ResFinder AMR genes and plasmid origins of replication
- Reference genome FASTA for QUAST evaluation

## Outputs

- Completed scaffold FASTA (prefix.fin.fasta) — target is one circular contig
- Scaffolding layout in JAPSA format (prefix.fin.japsa)
- Real-time annotation file (prefix.anno.japsa) when -resistGene/-oriRep are enabled
- Real-time assembly metrics log (e.g. QUAST report.html; per-gene plasmid assignment timeline vs cumulative Mb)

## How to apply

Assemble trimmed Illumina MiSeq reads with SPAdes (--careful, multi-threaded), sort contigs reverse by length with 'jsa.seq.sort -r -n', and build a 'bwa index' on the draft FASTA. Stream (or emulate streaming with jsa.np.timeEmulate) nanopore reads into BWA-MEM using the fixed parameter set '-t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y', piping SAM into 'jsa.np.npscarf -realtime'. npScarf determines each contig's multiplicity by comparing its short-read coverage to genome depth (uniqueness threshold θ = 1.5), builds bridges from reads spanning two unique contigs, merges contigs greedily (Kruskal-based) once a bridge scoring threshold is reached, fills gaps with bridging-read consensus, and inserts repetitive contigs into those gaps; contradicting bridges trigger breaking of the lowest-scoring bridge. Optionally supply ResFinder AMR genes and plasmid origins via -resistGene and -oriRep for real-time positional annotation. Evaluate the final scaffold (prefix.fin.fasta) with QUAST, checking for a single circular contig matching the reference and the cumulative long-read volume needed.

## Related tools

- **npScarf** (Core scaffolder: consumes SAM stream from BWA-MEM to scaffold, complete, and annotate the draft assembly in real time) — https://github.com/mdcao/npScarf
- **npReader** (Streams base-called MinION reads in real time from the run folder into the alignment step (jsa.np.npreader --realtime)) — https://github.com/mdcao/npReader
- **BWA-MEM** (Aligns incoming nanopore reads block-wise to the draft assembly with fixed parameters (-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y) supporting streaming)
- **SPAdes** (Assembles trimmed Illumina paired-end reads into the draft contigs that npScarf scaffolds)
- **trimmomatic** (Trims Illumina adapters and low-quality bases (V0.32) before SPAdes assembly)
- **QUAST** (Evaluates the final scaffold against the reference genome (contig count, assembly size))
- **japsa** (Package bundling jsa.np.npscarf, jsa.np.npreader, jsa.seq.sort, and jsa.np.timeEmulate utilities) — https://github.com/mdcao/japsa

## Examples

```
jsa.np.npreader --realtime --folder Downloads --fail --stat --number --output - | bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 Kp2146_spades.fasta - | jsa.np.npscarf -realtime -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-realtime > log.out 2>&1
```

## Evaluation signals

- QUAST report against the reference shows a single circular contig of expected assembly size (e.g. E. coli K12 MG1655 EcK12S.fin.fasta)
- Circularization/scaffolding completes at or below the expected long-read coverage (≤30-fold nanopore for E. coli; 83 Mb ≈14-fold for the Kpn2146 plasmid determination)
- prefix.fin.fasta and prefix.fin.japsa are both produced with the chosen -prefix
- With -resistGene/-oriRep, prefix.anno.japsa reports AMR genes as plasmid-encoded in a timeline matching the expected counts (18 of 27 AMR genes by 83 Mb; no new genes by 103 Mb when all four plasmids circularize)
- Contig multiplicity classification is stable when restricted to the up-to-20 largest contigs longer than 20 Kb

## Limitations

- Real-time only at minute-level temporal resolution, not millisecond-level updates per base
- Pipeline still requires short-read data, which cannot be generated in real time with current technology, and latency depends on fast base-callers such as Nanocall or DeepNano
- Bridge scoring threshold is only described as 'pre-defined' with no value stated, and θ = 1.5 was set from the authors' experience rather than a principled derivation
- npScarf is no longer maintained; npGraph (hsnguyen/assembly) is the replacement under development
- Competitors LINKS, NaS and Nanocorr failed to resolve the whole E. coli genome even on the 67-fold coverage dataset, so benchmark expectations are method-specific

## Evidence

- [abstract] npScarf can scaffold and complete short read assemblies while the long read sequencing run is in progress, reporting assembly metrics in real-time: "Here we present npScarf, which can scaffold and complete short read assemblies while the long read sequencing run is in progress."
- [discussion] npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome.: "npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome."
- [discussion] npScarf employs a mapping method that supports streaming processing such as BWA-MEM40 to align a small number of long reads to the existing assembly as they arrive.: "npScarf employs a mapping method that supports streaming processing such as BWA-MEM40 to align a small number of long reads to the existing assembly as they arrive."
- [readme] The nanopore sequencing data for the Kpn2164 sample in fastq format is made available: "bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 Kp2146_spades.fasta -"
- [discussion] MinION data for the E. coli K12 MG1655 sample20 were downloaded from the European Nucleotide Archive (ENA): "MinION data for the E. coli K12 MG1655 sample20 were downloaded from the European Nucleotide Archive (ENA) with accession number ERP007108."
