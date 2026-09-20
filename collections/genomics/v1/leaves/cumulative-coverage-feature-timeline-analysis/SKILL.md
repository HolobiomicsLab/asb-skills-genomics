---
name: cumulative-coverage-feature-timeline-analysis
description: Use when streaming nanopore reads in npScarf real-time mode with -resistGene/-oriRep
  annotation databases and you need to know how many Mb (or fold coverage) of long
  reads were required before each AMR gene or plasmid origin was reported.
when_to_use_negative:
- All nanopore data are already available and no timeline is needed — run npScarf
  in batch mode instead of real-time streaming
- You need millisecond-level real-time updating with each base added — the pipeline
  is only real-time at minute-level temporal resolution
- There are no Illumina short reads — npScarf requires a short-read draft assembly
  as input, which cannot itself be generated in real time
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: npScarf
  role: Real-time scaffolder whose streaming annotation mode (-realtime, -resistGene,
    -oriRep) reports plasmid-encoded AMR genes as bridges close, enabling the cumulative-data-volume
    timeline
  repo: https://github.com/mdcao/npScarf
- name: npReader / jsa.np.npreader
  role: Streams base-called MinION reads in real time (or emulates timing with jsa.np.timeEmulate
    -s 100) into the BWA-MEM | npScarf pipeline
  repo: https://github.com/mdcao/npReader
- name: BWA-MEM
  role: Aligns incoming blocks of long reads to the indexed draft assembly with the
    fixed parameter set -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000
- name: ResFinder database
  role: Reference FASTA of AMR genes supplied to npScarf via -resistGene so resistance
    genes are identified in real time during scaffolding
- name: Japsa
  role: Java package providing jsa.np.npscarf, jsa.np.npreader, jsa.np.timeEmulate,
    and jsa.seq.sort used throughout the timeline workflow
  repo: https://github.com/mdcao/japsa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:2f243f52eef5ddfb274fc89d887c5c018a5dcc2e7e9b9208656c4a7825bdf7fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - Japsa
  - ResFinder database
  - jsa.np.npreader
  - jsa.np.timeEmulate
  - jsa.seq.sort
  - npReader
  - npReader / jsa.np.npreader
  - npScarf
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Cumulative coverage feature timeline analysis

## Summary

Logs, per genomic feature (e.g. plasmid-encoded AMR gene), the cumulative volume of nanopore long-read data at which npScarf's real-time annotation reports the feature, yielding a data-volume timeline for feature confirmation during an ongoing MinION run.

## When to use

Use when streaming nanopore reads in npScarf real-time mode with -resistGene/-oriRep annotation databases and you need to know how many Mb (or fold coverage) of long reads were required before each AMR gene or plasmid origin was reported.

## When NOT to use

- All nanopore data are already available and no timeline is needed — run npScarf in batch mode instead of real-time streaming
- You need millisecond-level real-time updating with each base added — the pipeline is only real-time at minute-level temporal resolution
- There are no Illumina short reads — npScarf requires a short-read draft assembly as input, which cannot itself be generated in real time

## Inputs

- Draft assembly FASTA from SPAdes on trimmed Illumina MiSeq reads (e.g. Kp2146_spades.fasta, sorted with jsa.seq.sort -r -n and bwa-indexed)
- Streaming nanopore FASTQ reads (e.g. ERR868296 base-called via Metrichor, delivered by jsa.np.npreader --realtime or emulated with jsa.np.timeEmulate -s 100)
- AMR gene FASTA reference database (ResFinder) supplied via -resistGene
- Plasmid origin of replication FASTA database supplied via -oriRep

## Outputs

- prefix.anno.japsa file listing features (AMR genes, plasmid origins) detected after scaffolding, generated incrementally in real-time mode
- Per-feature timeline mapping each plasmid-encoded AMR gene to the cumulative long-read volume (Mb and fold coverage) at which it was confirmed
- Final real-time scaffold (prefix.fin.fasta) with plasmid circularization status

## How to apply

Run jsa.np.npscarf with -realtime and supply the draft assembly (-seq), plus FASTA files of AMR genes (-resistGene, ResFinder database) and plasmid origins of replication (-oriRep); detected features are written incrementally to prefix.anno.japsa as scaffolding proceeds. Stream MinION reads via jsa.np.npreader --realtime (or jsa.np.timeEmulate -s 100 for offline replay) into bwa mem using the fixed parameter set -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000. Monitor the annotation file and, for each AMR gene reported plasmid-encoded, record the cumulative base volume of long reads consumed at that point. Continue until the scaffolding terminates (all four plasmids circularized) and check that no additional features appear with further data, giving the minimal data volume per feature. Rationale: npScarf scaffolds and annotates as reads arrive, so the timeline quantifies how little sequencing is needed for real-time biological calls, avoiding over-sequencing.

## Related tools

- **npScarf** (Real-time scaffolder whose streaming annotation mode (-realtime, -resistGene, -oriRep) reports plasmid-encoded AMR genes as bridges close, enabling the cumulative-data-volume timeline) — https://github.com/mdcao/npScarf
- **npReader / jsa.np.npreader** (Streams base-called MinION reads in real time (or emulates timing with jsa.np.timeEmulate -s 100) into the BWA-MEM | npScarf pipeline) — https://github.com/mdcao/npReader
- **BWA-MEM** (Aligns incoming blocks of long reads to the indexed draft assembly with the fixed parameter set -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000)
- **ResFinder database** (Reference FASTA of AMR genes supplied to npScarf via -resistGene so resistance genes are identified in real time during scaffolding)
- **Japsa** (Java package providing jsa.np.npscarf, jsa.np.npreader, jsa.np.timeEmulate, and jsa.seq.sort used throughout the timeline workflow) — https://github.com/mdcao/japsa

## Examples

```
jsa.np.npreader --realtime --folder Downloads --output - | bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 Kp2146_spades.fasta - | jsa.np.npscarf -realtime -input - -format sam -seq Kp2146_spades.fasta -resistGene resfinder.fasta -oriRep plasmid_ori.fasta -prefix Kp2146-realtime
```

## Evaluation signals

- 18 of the 27 AMR genes in the Kpn2146 Illumina assembly are reported plasmid-encoded once ~83 Mb (~14-fold) of nanopore reads have streamed
- No additional AMR genes are reported by 103 Mb (~18-fold), when all four plasmids are circularized
- The annotation file prefix.anno.japsa exists and grows incrementally only in real-time mode when annotation analysis is enabled
- Every logged feature timestamp corresponds to a monotonically increasing cumulative read volume (later detections never report less data)
- Plasmid assignment is corroborated by plasmid origins of replication detected against the -oriRep database

## Limitations

- Real-time is at minute-level temporal resolution, not the millisecond level required to update with each base added
- The pipeline still requires short-read data, which cannot be generated in real time with current technology, and depends on a base-caller (fast real-time base-callers like Nanocall and DeepNano reduce latency)
- npScarf is no longer maintained; npGraph is the designated replacement

## Evidence

- [results] npScarf confirmed 18 AMR genes as plasmid-encoded using 83 Mb (~14-fold coverage) of nanopore sequencing data streamed during the run.: [evidence span withheld — anchor status: not_found]
- [discussion] Log, per AMR gene, the cumulative long-read volume at which the gene is reported plasmid-encoded, until all four plasmids are circularized.: "It reports assembly metrics in real-time so the sequencing run can be terminated once an assembly of sufficient quality is obtained."
- [readme] In real-time mode, if any annotation analysis is enabled, a file named prefix.anno.japsa is generated instead. This file contains features detected after scaffolding.: "In realtime mode, if any annotation analysis is enabled, a file named *prefix*.anno.japsa is generated instead. This file contains features detected after scaffolding."
- [readme] The tool also facilitates real-time analysis of positional information such as gene ordering and the detection of genes from mobile elements (plasmids and genomic islands).: "It also facilitates the real-time analysis of positional information such as gene ordering and the detection of genes from
mobile elements (plasmids and genomic islands)."
- [discussion] real-time at the temporal resolution of minutes, but not at the millisecond level required to update with the addition of each base.: "real-time at the temporal resolution of minutes, but not at the millisecond level required to update with the addition of each base."
