---
name: plasmid-origin-of-replication-annotation
description: Use when completing a bacterial draft assembly with streaming MinION
  reads and you need to determine, during the run, which AMR genes sit on plasmids
  rather than the chromosome.
when_to_use_negative:
- You have only short-read (Illumina) data — npScarf requires long nanopore reads
  aligned to the draft assembly, and the paper notes the pipeline still requires short
  read data as input
- You need a finished assembly rather than positional annotation of mobile elements
  — use standard batch scaffolding or a successor tool such as npGraph, since npScarf
  is no longer maintained
- Batch analysis after sequencing has finished — the real-time annotation value (stopping
  once genes are confirmed) is lost; use batch bwa mem | jsa.np.npscarf mode instead
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: npScarf
  role: Real-time scaffolder that, when given -resistGene and -oriRep annotation FASTAs,
    reports plasmid origins and AMR genes to prefix.anno.japsa as bridges close plasmids
  repo: https://github.com/mdcao/npScarf
- name: npReader (jsa.np.npreader)
  role: Streams base-called MinION reads (--realtime --folder Downloads) into bwa
    mem so annotation updates as the run progresses
  repo: https://github.com/mdcao/npReader
- name: BWA-MEM
  role: Aligns streamed long reads block-wise to the draft assembly with the fixed
    parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000
- name: ResFinder database
  role: Reference FASTA used to identify AMR genes in the draft assembly prior to
    real-time plasmid assignment
- name: jsa.np.timeEmulate
  role: Emulates real-time streaming (e.g. -s 100) for reproducible offline replay
    of the annotation timeline
  repo: https://github.com/mdcao/japsa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:ccb5ae11fe87ade2eab1b85b0ae4d8522d83a07982625b8677fe53025a346fb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - ResFinder database
  - jsa.np.npreader
  - jsa.np.timeEmulate
  - jsa.seq.sort
  - npReader
  - npReader (jsa.np.npreader)
  - npScarf
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# plasmid-origin-of-replication-annotation

## Summary

Real-time annotation of plasmid origins of replication and AMR genes as npScarf scaffolds an assembly with streaming nanopore reads, allowing plasmid-borne resistance determinants (e.g. 18 of 27 AMR genes in K. pneumoniae Kpn2146) to be confirmed with as little as 83 Mb (~14-fold) of long-read data.

## When to use

Use when completing a bacterial draft assembly with streaming MinION reads and you need to determine, during the run, which AMR genes sit on plasmids rather than the chromosome.

## When NOT to use

- You have only short-read (Illumina) data — npScarf requires long nanopore reads aligned to the draft assembly, and the paper notes the pipeline still requires short read data as input
- You need a finished assembly rather than positional annotation of mobile elements — use standard batch scaffolding or a successor tool such as npGraph, since npScarf is no longer maintained
- Batch analysis after sequencing has finished — the real-time annotation value (stopping once genes are confirmed) is lost; use batch bwa mem | jsa.np.npscarf mode instead

## Inputs

- Draft assembly FASTA from SPAdes on trimmed Illumina MiSeq reads (e.g. Kp2146_spades.fasta), bwa-indexed
- Streamed base-called nanopore FASTQ reads (e.g. ENA ERR868296, Kp2146_ONT.fastq.gz)
- AMR gene sequences from the ResFinder database (supplied as FASTA)
- Plasmid origins of replication sequences (supplied as FASTA)

## Outputs

- prefix.anno.japsa file listing plasmid origins of replication and AMR genes detected after scaffolding
- Real-time timeline of cumulative long-read volume (Mb) at which each AMR gene is confirmed plasmid-encoded
- Completed scaffolds in prefix.fin.fasta with plasmid circularization status

## How to apply

Build the draft assembly (e.g. SPAdes --careful on trimmed Illumina MiSeq reads, contigs sorted with jsa.seq.sort -r -n) and bwa-index it. Identify AMR genes in the assembly against the ResFinder database and plasmid origins of replication against a plasmid-origin database, then supply both as FASTA annotation inputs to jsa.np.npscarf so that as nanopore reads stream in (via jsa.np.npreader --realtime into bwa mem with -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000), scaffolding proceeds and resistance genes and plasmid origins are detected and written to prefix.anno.japsa. Log, per AMR gene, the cumulative long-read volume at which the gene is reported plasmid-encoded, until all plasmids are circularized (four plasmids in Kpn2146, with no additional genes reported by 103 Mb / ~18-fold).

## Related tools

- **npScarf** (Real-time scaffolder that, when given -resistGene and -oriRep annotation FASTAs, reports plasmid origins and AMR genes to prefix.anno.japsa as bridges close plasmids) — https://github.com/mdcao/npScarf
- **npReader (jsa.np.npreader)** (Streams base-called MinION reads (--realtime --folder Downloads) into bwa mem so annotation updates as the run progresses) — https://github.com/mdcao/npReader
- **BWA-MEM** (Aligns streamed long reads block-wise to the draft assembly with the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000)
- **ResFinder database** (Reference FASTA used to identify AMR genes in the draft assembly prior to real-time plasmid assignment)
- **jsa.np.timeEmulate** (Emulates real-time streaming (e.g. -s 100) for reproducible offline replay of the annotation timeline) — https://github.com/mdcao/japsa

## Examples

```
jsa.np.npreader --realtime --folder Downloads --output - | bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 Kp2146_spades.fasta - | jsa.np.npscarf -realtime -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-realtime
```

## Evaluation signals

- With ~83 Mb (~14-fold) of nanopore data, 18 of the 27 AMR genes identified in the Illumina assembly are reported plasmid-encoded
- No additional AMR genes are reported between 83 Mb and 103 Mb (~18-fold), when all four Kpn2146 plasmids are circularized
- A prefix.anno.japsa file exists in real-time mode and contains features detected after scaffolding
- Per-gene timeline logs record the cumulative read volume at which each AMR gene is first called plasmid-encoded, in chronological (streamed) order
- Batch-mode scaffolding (all reads at once) agrees with the real-time mode's final gene-to-plasmid assignments

## Limitations

- Real-time is at the temporal resolution of minutes, not milliseconds, and the pipeline still relies on a base-caller, so latency depends on fast base-callers such as Nanocall or DeepNano
- The pipeline still requires short read data, which cannot be generated in real time with current technology, so plasmid assignment is only as good as the upstream draft assembly
- npScarf is no longer maintained; npGraph is the intended replacement, and command structure/parameters can change significantly between npScarf versions (check --help first)

## Evidence

- [discussion] npScarf demonstrated real-time identification of AMR genes encoded in plasmids and pathogenicity islands.: [evidence span withheld — anchor status: not_found]
- [readme] npScarf README on mobile-element detection: "It also facilitates the real-time analysis of positional information such as gene ordering and the detection of genes from mobile elements (plasmids and genomic islands)."
- [readme] npScarf README on real-time annotation output: "In realtime mode, if any annotation analysis is enabled, a file named *prefix*.anno.japsa is generated instead. This file contains features detected after scaffolding."
- [readme] npScarf README on real-time scaffolding mode: "To run *npScarf* in streaming mode::

   	jsa.np.npscarf -realtime [options]"
