---
name: hybrid-assembly-gap-filling
description: Use when a draft short-read assembly (e.g. SPAdes contigs.fasta) must
  be scaffolded and gap-filled using nanopore long reads, especially in real-time/streaming
  mode or batch BWA-MEM alignment pipelines.
when_to_use_negative:
- The long-read dataset is already assembled de novo with a long-read assembler (e.g.
  Celera Assembler) rather than used to scaffold an existing short-read draft
- Only short-read data are available with no long reads — bridges cannot be formed
  without alignments spanning two unique contigs
- Windows/macOS deployment is required without meeting Japsa's Java/HDF5 installation
  dependencies (npGraph successor targets Linux only)
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: npScarf
  role: 'Core scaffolder (jsa.np.npscarf): classifies contigs by copy number, groups
    supporting reads into scored bridges, merges contigs greedily, fills gaps with
    bridge-read consensus, and inserts repetitive contigs into gaps'
  repo: https://github.com/mdcao/npScarf
- name: BWA-MEM
  role: Aligns nanopore long reads to the draft assembly with the fixed parameter
    set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y, streaming SAM output to npScarf
- name: SPAdes
  role: Assembles trimmed Illumina MiSeq paired-end reads (--careful, 16 threads)
    into the draft contigs.fasta used as scaffolding input
- name: Japsa (jsa.seq.sort)
  role: Sorts SPAdes contigs by length (-r -n) to produce the length-ordered draft
    Kp2146_spades.fasta; jsa.np.npreader streams base-called reads in real-time mode
  repo: https://github.com/mdcao/japsa
- name: npReader
  role: Real-time conversion and streaming of nanopore data (jsa.np.npreader --realtime
    --folder Downloads) feeding the BWA-MEM/npScarf pipeline
  repo: https://github.com/mdcao/npReader
- name: npGraph
  role: Successor streaming assembly tool under development replacing the unmaintained
    npScarf
  repo: https://github.com/hsnguyen/assembly/blob/master/docs/npgraph.md
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:1690ade1a3867ccbba7c9fd6e631485fd421b424beb198bf1d732f6234849591
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - Japsa (jsa.seq.sort)
  - SPAdes
  - npGraph
  - npReader
  - npScarf
  - npScarf (jsa.np.npscarf)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# hybrid-assembly-gap-filling

## Summary

Reconstructs the npScarf hybrid scaffolding workflow: short-read (Illumina MiSeq/SPAdes) contigs are scaffolded by streaming Oxford Nanopore long-read bridges via jsa.np.npscarf, with unique contigs forming the backbone and repetitive contigs inserted into gaps.

## When to use

Use when a draft short-read assembly (e.g. SPAdes contigs.fasta) must be scaffolded and gap-filled using nanopore long reads, especially in real-time/streaming mode or batch BWA-MEM alignment pipelines.

## When NOT to use

- The long-read dataset is already assembled de novo with a long-read assembler (e.g. Celera Assembler) rather than used to scaffold an existing short-read draft
- Only short-read data are available with no long reads — bridges cannot be formed without alignments spanning two unique contigs
- Windows/macOS deployment is required without meeting Japsa's Java/HDF5 installation dependencies (npGraph successor targets Linux only)

## Inputs

- Trimmed Illumina MiSeq paired-end FASTQ files (e.g. Kp2146_paired_1.fastq.gz, Kp2146_paired_2.fastq.gz)
- Nanopore long-read FASTQ/FASTA (e.g. Kp2146_ONT.fastq.gz; raw signal via ENA run ERR868296)
- Sorted draft assembly FASTA (Kp2146_spades.fasta) with a BWA index
- SAM/BAM alignments between draft assembly and nanopore reads (or long reads passed directly with -format fastq)

## Outputs

- prefix.fin.fasta — final scaffolded genome in FASTA format
- prefix.fin.japsa — final scaffold in JAPSA format
- prefix.anno.japsa — annotation features detected after scaffolding (real-time mode with annotation enabled)

## How to apply

Assemble trimmed Illumina MiSeq paired-end reads with SPAdes (--careful, 16 threads) to obtain spades/contigs.fasta, sort contigs by length with jsa.seq.sort -r -n, and build a BWA index on the sorted draft. Align nanopore reads to the draft with the fixed BWA-MEM parameter set (-t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y) and stream the SAM into jsa.np.npscarf -format sam -seq <draft> -prefix <prefix>. npScarf determines each contig's multiplicity by comparing contig short-read coverage to whole-genome coverage (multiplicity threshold θ = 1.5 for declaring a contig unique), builds bridges from reads aligned to two unique contigs, scores bridges and merges contigs greedily (Kruskal-based), fills gaps with the bridging-read consensus, and inserts repetitive contigs aligned to that consensus. Success is judged by npScarf terminating cleanly and producing prefix.fin.fasta and prefix.fin.japsa.

## Related tools

- **npScarf** (Core scaffolder (jsa.np.npscarf): classifies contigs by copy number, groups supporting reads into scored bridges, merges contigs greedily, fills gaps with bridge-read consensus, and inserts repetitive contigs into gaps) — https://github.com/mdcao/npScarf
- **BWA-MEM** (Aligns nanopore long reads to the draft assembly with the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y, streaming SAM output to npScarf)
- **SPAdes** (Assembles trimmed Illumina MiSeq paired-end reads (--careful, 16 threads) into the draft contigs.fasta used as scaffolding input)
- **Japsa (jsa.seq.sort)** (Sorts SPAdes contigs by length (-r -n) to produce the length-ordered draft Kp2146_spades.fasta; jsa.np.npreader streams base-called reads in real-time mode) — https://github.com/mdcao/japsa
- **npReader** (Real-time conversion and streaming of nanopore data (jsa.np.npreader --realtime --folder Downloads) feeding the BWA-MEM/npScarf pipeline) — https://github.com/mdcao/npReader
- **npGraph** (Successor streaming assembly tool under development replacing the unmaintained npScarf) — https://github.com/hsnguyen/assembly/blob/master/docs/npgraph.md

## Examples

```
bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq | jsa.np.npscarf -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-batch
```

## Evaluation signals

- npScarf terminates successfully and emits exactly the two canonical outputs: prefix.fin.fasta and prefix.fin.japsa
- Contig copy-number classification applies the multiplicity threshold θ = 1.5 (contig short-read coverage vs whole-genome coverage) before scaffolding
- Contigs are only merged once a bridge reaches the pre-defined scoring threshold; contradicting bridges resolve by breaking the lowest-scoring bridge
- In real-time mode, the pipeline reports assembly metrics continuously (streaming via jsa.np.npreader | bwa mem -K 3000 | jsa.np.npscarf -realtime) so quality can be monitored during the run
- The SPAdes draft (spades/contigs.fasta) is length-sorted before indexing, so the contig ordering in Kp2146_spades.fasta reflects descending length

## Limitations

- Real-time is at the temporal resolution of minutes, not the millisecond level needed to update with each added base
- The pipeline still requires short-read data, which cannot be generated in real-time with current technology, and relies on a base-caller whose latency affects the run
- The bridge scoring threshold is only described as 'pre-defined' with no published value, and θ = 1.5 was set from authors' experience rather than a principled derivation
- npScarf is no longer maintained; npGraph is the under-development replacement

## Evidence

- [discussion] npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome.: "npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome."
- [discussion] npScarf groups reads that consistently support a particular relative position into a bridge and assigns the bridge a score: "npScarf groups reads that consistently support a particular relative position into a bridge and assigns the bridge a score"
- [discussion] the gap is temporarily filled with the consensus sequence of the reads forming the bridge. npScarf then identifies repetitive contigs that are aligned to this consensus sequence: "the gap is temporarily filled with the consensus sequence of the reads forming the bridge. npScarf then identifies repetitive contigs that are"
- [readme] bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq  | jsa.np.npscarf -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-batch: "bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq  | jsa.np.npscarf -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-batch"
- [readme] Normally the tool generate two files: *prefix*.fin.fasta and *prefix*.fin.japsa which indicate the result scaffolders in FASTA and JAPSA format.: "Normally the tool generate two files: *prefix*.fin.fasta and *prefix*.fin.japsa which indicate the result scaffolders in FASTA and JAPSA format."
