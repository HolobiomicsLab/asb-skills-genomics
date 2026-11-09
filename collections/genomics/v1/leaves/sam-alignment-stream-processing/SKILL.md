---
name: sam-alignment-stream-processing
description: Use when long nanopore reads must be aligned to a draft short-read (e.g.
  SPAdes) assembly and consumed incrementally by a streaming scaffolder like npScarf,
  rather than written to intermediate SAM/BAM files.
when_to_use_negative:
- Your assembly graph is the primary input and you want the maintained successor npGraph
  (npScarf is no longer maintained); npGraph uses minimap2/bwa with SAM/BAM or raw
  FASTA/FASTQ inputs instead.
- You only need a static alignment file for downstream non-streaming tools — a plain
  'bwa mem ... > alignments.sam' run without the jsa.np.npscarf pipe is appropriate.
- The pipeline must update at millisecond resolution with each base added — npScarf
  streaming is only real-time at the temporal resolution of minutes.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: BWA-MEM
  role: Aligns nanopore long reads to the draft assembly with the fixed parameter
    set (-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y), streaming SAM output into npScarf;
    supports block-wise streaming via -K.
- name: jsa.np.npscarf
  role: Consumes the SAM stream from stdin (-input - -format sam), classifies contigs,
    builds scored bridges, and greedily merges contigs in batch or -realtime mode.
  repo: https://github.com/mdcao/npScarf
- name: jsa.np.npreader
  role: In real-time mode, streams base-called reads (--realtime --folder Downloads)
    from the ongoing MinION run into the BWA-MEM pipe.
  repo: https://github.com/mdcao/npReader
- name: SPAdes
  role: Produces the draft short-read contigs (--careful, -t 16) that form the -seq
    input to the alignment/scaffolding stream.
- name: jsa.seq.sort
  role: Sorts SPAdes contigs by length (-r -n) to produce the FASTA used as the BWA
    index and npScarf reference.
  repo: https://github.com/mdcao/japsa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:ac4a4c62a04d3822b23ec5464b7418feb11e9902eb52df2023ae1defe2bdd580
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - Japsa (jsa.seq.sort)
  - SPAdes
  - jsa.np.npreader
  - jsa.np.npscarf
  - jsa.seq.sort
  - npScarf
  - npScarf (jsa.np.npscarf)
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# sam-alignment-stream-processing

## Summary

Streams BWA-MEM SAM alignments of nanopore reads against a short-read draft assembly directly into jsa.np.npscarf via a Unix pipe, enabling Kruskal-style greedy contig merging (npScarf scaffolding) either in batch or real-time mode without intermediate alignment files.

## When to use

Use when long nanopore reads must be aligned to a draft short-read (e.g. SPAdes) assembly and consumed incrementally by a streaming scaffolder like npScarf, rather than written to intermediate SAM/BAM files.

## When NOT to use

- Your assembly graph is the primary input and you want the maintained successor npGraph (npScarf is no longer maintained); npGraph uses minimap2/bwa with SAM/BAM or raw FASTA/FASTQ inputs instead.
- You only need a static alignment file for downstream non-streaming tools — a plain 'bwa mem ... > alignments.sam' run without the jsa.np.npscarf pipe is appropriate.
- The pipeline must update at millisecond resolution with each base added — npScarf streaming is only real-time at the temporal resolution of minutes.

## Inputs

- Draft assembly FASTA (e.g. SPAdes spades/contigs.fasta, length-sorted with jsa.seq.sort -r -n)
- Nanopore long-read FASTQ/FASTA (e.g. Kp2146_ONT.fastq.gz; ENA run accession ERR868296 for Kpn2146)
- BWA index of the draft assembly
- Optionally, a real-time stream of base-called reads from jsa.np.npreader (HDF5 folder input)

## Outputs

- SAM alignment stream (stdout of bwa mem, consumed via stdin by jsa.np.npscarf)
- prefix.fin.fasta — final scaffolded assembly
- prefix.fin.japsa — scaffold in JAPSA format
- prefix.anno.japsa — detected features when annotation analysis is enabled in real-time mode

## How to apply

Index the draft assembly (e.g. Kp2146_spades.fasta, the length-sorted SPAdes contigs) with 'bwa index'. Run BWA-MEM with the npScarf-tuned parameter set (-t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y), which relaxes scoring for noisy nanopore reads and emits all alignments (-a), and pipe the SAM output ('-input - -format sam') into jsa.np.npscarf. In real-time mode, interpose jsa.np.npreader (or npReader) upstream and pass -K 3000 to BWA-MEM so reads are aligned block-wise as they arrive; npScarf then classifies contigs by copy number (multiplicity threshold θ = 1.5), merges contigs greedily once a bridge reaches its pre-defined scoring threshold, and breaks the lowest-scoring contradicting bridge on conflict. Specify -prefix so the two canonical outputs (prefix.fin.fasta and prefix.fin.japsa) are identifiable. The rationale for streaming is that npScarf's scaffolding decisions update incrementally with each read block, supporting real-time termination once sufficient quality is reached.

## Related tools

- **BWA-MEM** (Aligns nanopore long reads to the draft assembly with the fixed parameter set (-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y), streaming SAM output into npScarf; supports block-wise streaming via -K.)
- **jsa.np.npscarf** (Consumes the SAM stream from stdin (-input - -format sam), classifies contigs, builds scored bridges, and greedily merges contigs in batch or -realtime mode.) — https://github.com/mdcao/npScarf
- **jsa.np.npreader** (In real-time mode, streams base-called reads (--realtime --folder Downloads) from the ongoing MinION run into the BWA-MEM pipe.) — https://github.com/mdcao/npReader
- **SPAdes** (Produces the draft short-read contigs (--careful, -t 16) that form the -seq input to the alignment/scaffolding stream.)
- **jsa.seq.sort** (Sorts SPAdes contigs by length (-r -n) to produce the FASTA used as the BWA index and npScarf reference.) — https://github.com/mdcao/japsa

## Examples

```
bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq | jsa.np.npscarf -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-batch
```

## Evaluation signals

- jsa.np.npscarf terminates successfully and produces exactly the two canonical outputs, prefix.fin.fasta and prefix.fin.japsa.
- The BWA-MEM invocation uses the exact npScarf parameter set (-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y) and reads arrive via stdin ('-input - -format sam'), not from a written file.
- In real-time mode, the log (e.g. log.out) shows scaffolds being updated block-wise (with -K 3000 batching) and assembly metrics reported while the run is in progress.
- The scaffolded output contains merged contigs bridged by long reads, with repetitive contigs inserted into gaps rather than left unplaced.
- Running 'jsa.np.npscarf --help' first confirms option names match the installed Japsa version, since the parameter list 'can be changed significantly from different versions'.

## Limitations

- npScarf is no longer maintained; npGraph is under development as its replacement.
- Real-time is at the temporal resolution of minutes, not the millisecond level required to update with the addition of each base, and the pipeline still requires short read data and a base-caller (latency depends on fast real-time base-callers such as Nanocall and DeepNano).
- The bridge scoring threshold is only described as 'pre-defined' with no value given, and the θ = 1.5 multiplicity threshold is set from the authors' experience rather than a principled derivation.
- Option names and parameters of jsa.np.npscarf can change significantly between versions, so --help must be checked before running; real-time mode additionally requires npReader and a properly installed HDF library.

## Evidence

- [readme] bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq  | jsa.np.npscarf -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-batch: "bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq  | jsa.np.npscarf -input - -format sam"
- [discussion] npScarf employs a mapping method that supports streaming processing such as BWA-MEM40 to align a small number of long reads to the existing assembly as they arrive.: "npScarf employs a mapping method that supports streaming processing such as BWA-MEM40 to align a small number of long reads to the existing assembly as they arrive."
- [discussion] Once the bridge reaches a pre-defined scoring threshold, the two contigs are merged and the gap is filled as above.: "Once the bridge reaches a pre-defined scoring threshold, the two contigs are merged and the gap is filled as above."
- [readme] Normally the tool generate two files: *prefix*.fin.fasta and *prefix*.fin.japsa which indicate the result scaffolders in FASTA and JAPSA format.: "Normally the tool generate two files: *prefix*.fin.fasta and *prefix*.fin.japsa which"
- [discussion] npScarf set the threshold θ to 1.5.: "npScarf set the threshold θ to 1.5."
