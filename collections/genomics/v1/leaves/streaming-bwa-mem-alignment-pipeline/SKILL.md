---
name: streaming-bwa-mem-alignment-pipeline
description: Use when nanopore long reads arrive incrementally during an ongoing MinION
  run and must be aligned block-wise to a pre-assembled draft (e.g. SPAdes contigs)
  to update a real-time npScarf scaffold.
when_to_use_negative:
- All nanopore reads are already available and no real-time scaffolding is needed
  — a plain batch bwa mem + scaffolder run is simpler.
- Input is short-read (Illumina) data rather than long nanopore reads — the k11/-L0
  parameters are tuned for long, error-prone reads and the draft is the BWA target.
- npScarf scaffolding itself is not the goal (e.g. polishing or base-calling) — this
  pipeline produces scaffolds, not corrected reads or consensus.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: BWA-MEM
  role: Streams long reads block-wise against the draft assembly with fixed parameters
    -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y
- name: npScarf (jsa.np.npscarf)
  role: Consumes the SAM stream and merges contigs in real-time when bridge score
    thresholds are reached
  repo: https://github.com/mdcao/npScarf
- name: npReader (jsa.np.npreader)
  role: Streams base-called nanopore reads in real time from the sequencing output
    folder into the BWA-MEM pipe
  repo: https://github.com/mdcao/npReader
- name: SPAdes
  role: Produces the draft Illumina assembly that serves as the BWA index target
- name: npGraph
  role: Under development as the replacement for the now-unsupported npScarf streaming
    pipeline
  repo: https://github.com/hsnguyen/assembly/blob/master/docs/npgraph.md
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:6c1884b56de3d312d81b80fbc2a6d0b4195ccbf6fa79381c067d342bf75fdb9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - SPAdes
  - npGraph
  - npReader
  - npReader (jsa.np.npreader)
  - npScarf
  - npScarf (jsa.np.npscarf)
  - trimmomatic
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# streaming-bwa-mem-alignment-pipeline

## Summary

Streams nanopore reads (live or emulated) through BWA-MEM with a fixed long-read-tuned parameter set (-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y) and pipes the SAM output into jsa.np.npscarf for real-time scaffolding of a short-read draft assembly.

## When to use

Use when nanopore long reads arrive incrementally during an ongoing MinION run and must be aligned block-wise to a pre-assembled draft (e.g. SPAdes contigs) to update a real-time npScarf scaffold.

## When NOT to use

- All nanopore reads are already available and no real-time scaffolding is needed — a plain batch bwa mem + scaffolder run is simpler.
- Input is short-read (Illumina) data rather than long nanopore reads — the k11/-L0 parameters are tuned for long, error-prone reads and the draft is the BWA target.
- npScarf scaffolding itself is not the goal (e.g. polishing or base-calling) — this pipeline produces scaffolds, not corrected reads or consensus.

## Inputs

- Sorted draft assembly FASTA from SPAdes (e.g. contigs sorted with jsa.seq.sort -r -n)
- bwa index built on the draft assembly
- Nanopore long reads in FASTQ/FASTA format (e.g. MinION R7.3 run ENA ERR637419) or base-called SAM/BAM alignments
- Streaming read source from npReader/jsa.np.npreader pointed at the Metrichor output folder

## Outputs

- SAM alignment stream (piped, one block per batch of arriving reads)
- npScarf scaffold files prefix.fin.fasta and prefix.fin.japsa
- Real-time assembly metrics log (e.g. log.out) used to decide run termination

## How to apply

Assemble Illumina paired-end reads with SPAdes, sort contigs longest-first with jsa.seq.sort -r -n, and build a bwa index on the sorted FASTA. Then stream base-called nanopore reads via jsa.np.npreader --realtime (or emulate with jsa.np.timeEmulate), pipe into 'bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000' against the draft, and feed the SAM stream to 'jsa.np.npscarf -realtime'. The short seed length k11 and -L0 are deliberate for error-prone long reads; npScarf aligns each small block of incoming reads and merges contigs once a bridge reaches its pre-defined scoring threshold, so alignment latency must stay low. Alternatively, when all long reads are already base-called, run the same BWA-MEM command in batch mode; newer npScarf versions can also integrate BWA-MEM via -bwaExe/-bwaThread and take FASTQ input directly.

## Related tools

- **BWA-MEM** (Streams long reads block-wise against the draft assembly with fixed parameters -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y)
- **npScarf (jsa.np.npscarf)** (Consumes the SAM stream and merges contigs in real-time when bridge score thresholds are reached) — https://github.com/mdcao/npScarf
- **npReader (jsa.np.npreader)** (Streams base-called nanopore reads in real time from the sequencing output folder into the BWA-MEM pipe) — https://github.com/mdcao/npReader
- **SPAdes** (Produces the draft Illumina assembly that serves as the BWA index target)
- **npGraph** (Under development as the replacement for the now-unsupported npScarf streaming pipeline) — https://github.com/hsnguyen/assembly/blob/master/docs/npgraph.md

## Examples

```
jsa.np.npreader --realtime --folder Downloads --fail --stat --number --output - | bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 Kp2146_spades.fasta - | jsa.np.npscarf -realtime -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-realtime > log.out 2>&1
```

## Evaluation signals

- The exact fixed BWA-MEM parameter string (-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y) is used, matching the README's recommended set.
- SAM records arrive incrementally (streamed), not as a single file, and npScarf is invoked with -realtime -input - -format sam.
- Contig merges occur progressively as bridge scores reach the pre-defined threshold, observable in the npScarf metrics log.
- For the E. coli K12 MG1655 task, QUAST on the final scaffold (EcK12S.fin.fasta) reports one circular contig matching the reference at <30-fold nanopore coverage.
- jsa.np.npscarf --help is checked first, since options and parameter lists change significantly between versions (README warning).

## Limitations

- Real-time is at minute-level temporal resolution, not millisecond resolution per base added.
- Latency depends on availability of fast real-time base-callers (e.g. Nanocall, DeepNano) before reads reach the aligner.
- npScarf is no longer maintained; npGraph in hsnguyen/assembly is the designated replacement.
- The bridge scoring threshold is 'pre-defined' but its value is not stated in the paper text.

## Evidence

- [discussion] npScarf employs a mapping method that supports streaming processing such as BWA-MEM to align a small number of long reads to the existing assembly as they arrive.: "npScarf employs a mapping method that supports streaming processing such as BWA-MEM40 to align a small number of long reads to the existing assembly as they arrive."
- [readme] We use BWA-MEM as the recommended aligner with the fixed parameter set as follow: bwa mem -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y: "We use BWA-MEM as the recommended aligner 
with the fixed parameter set as follow:

	bwa mem -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y"
- [readme] jsa.np.npscarf -realtime -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-realtime: "| jsa.np.npscarf -realtime -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-realtime > log.out 2>&1"
- [discussion] Once the bridge reaches a pre-defined scoring threshold, the two contigs are merged and the gap is filled as above.: "Once the bridge reaches a pre-defined scoring threshold, the two contigs are merged and the gap is filled as above."
