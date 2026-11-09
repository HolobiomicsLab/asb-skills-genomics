---
name: amr-gene-identification-resfinder
description: Use when a bacterial draft assembly needs AMR gene annotation against
  ResFinder, or when AMR genes must be localized to plasmids/replicons in real time
  during nanopore scaffolding with npScarf.
when_to_use_negative:
- Input is raw short reads with no assembly — ResFinder screening here is performed
  on a draft assembly, not directly on reads
- The organism/AMR determinants of interest are not represented in the ResFinder database,
  so gene calls would be missed
- Only a finished closed genome with plasmid assignments already available is being
  annotated — real-time scaffolding-based localization adds no value
edam_operation: http://edamontology.org/operation_3482
edam_topics:
- http://edamontology.org/topic_3305
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: npScarf
  role: Real-time scaffolder that takes -resistGene and -oriRep FASTA annotations
    and reports plasmid-encoded AMR genes to prefix.anno.japsa as scaffolding proceeds
  repo: https://github.com/mdcao/npScarf
- name: ResFinder database
  role: Reference database used to identify AMR genes in the Illumina draft assembly
- name: SPAdes
  role: Assembles trimmed Illumina MiSeq reads (--careful, 16 threads) into the draft
    assembly that is screened for AMR genes
- name: BWA-MEM
  role: Aligns streaming nanopore reads to the indexed draft assembly with the fixed
    parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000
- name: npReader
  role: Streams base-called MinION reads in real time (jsa.np.npreader) into the scaffolding/annotation
    pipeline
  repo: https://github.com/mdcao/npReader
- name: jsa.seq.sort
  role: Sorts SPAdes contigs (-r -n) to produce Kp2146_spades.fasta prior to indexing
    and AMR screening
  repo: https://github.com/mdcao/japsa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:cb7547230dc007f937cd1e0c345e35b462f8092bf18bc28a2f689e70459dd219
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - ResFinder database
  - SPAdes
  - jsa.np.npreader
  - jsa.np.timeEmulate
  - jsa.seq.sort
  - npReader
  - npScarf
  edam_operation: http://edamontology.org/operation_3482
  edam_topics:
  - http://edamontology.org/topic_3305
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# amr-gene-identification-resfinder

## Summary

Identifies antimicrobial resistance (AMR) genes in a bacterial draft assembly (e.g. K. pneumoniae ATCC BAA-2146 assembled with SPAdes) by searching contigs against the ResFinder database, and — within the npScarf real-time pipeline — supplying these genes (plus plasmid origins of replication) as FASTA annotations so plasmid-encoded AMR determinations can be streamed during a MinION run.

## When to use

Use when a bacterial draft assembly needs AMR gene annotation against ResFinder, or when AMR genes must be localized to plasmids/replicons in real time during nanopore scaffolding with npScarf.

## When NOT to use

- Input is raw short reads with no assembly — ResFinder screening here is performed on a draft assembly, not directly on reads
- The organism/AMR determinants of interest are not represented in the ResFinder database, so gene calls would be missed
- Only a finished closed genome with plasmid assignments already available is being annotated — real-time scaffolding-based localization adds no value

## Inputs

- Draft bacterial assembly FASTA (e.g. Kp2146_spades.fasta from SPAdes --careful on trimmed Illumina MiSeq reads)
- ResFinder AMR gene database (FASTA)
- Plasmid origin-of-replication database (FASTA, for npScarf -oriRep)
- Base-called nanopore FASTQ reads (e.g. Kp2146_ONT.fastq.gz, ENA ERR868296), streamed via jsa.np.npreader --realtime or jsa.np.timeEmulate -s 100

## Outputs

- AMR gene calls for the draft assembly (which of the 27 ResFinder hits are present)
- Real-time per-gene annotation of plasmid-encoded vs chromosomal AMR genes in prefix.anno.japsa
- Cumulative long-read volume (Mb) at which each AMR gene is reported plasmid-encoded, until all plasmids are circularized

## How to apply

First assemble trimmed Illumina MiSeq reads (e.g. Kp2146 paired-end data trimmed with trimmomatic V0.32) with SPAdes --careful using 16 threads, sort contigs with jsa.seq.sort -r -n, and build a bwa index on the sorted draft (Kp2146_spades.fasta). Identify AMR genes by searching the assembly against the ResFinder database, and optionally identify plasmid origins of replication against a plasmid-origin database. Supply both annotation sets to npScarf via -resistGene and -oriRep so that, as nanopore reads are streamed block-wise through bwa mem (-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000), scaffolding proceeds and resistance genes are classified plasmid-encoded or chromosomal, written incrementally to prefix.anno.japsa. Validation benchmark: npScarf confirmed 18 of 27 AMR genes as plasmid-encoded with 83 Mb (~14-fold) of nanopore data, with no additional genes reported by 103 Mb (~18-fold) when all four plasmids were circularized.

## Related tools

- **npScarf** (Real-time scaffolder that takes -resistGene and -oriRep FASTA annotations and reports plasmid-encoded AMR genes to prefix.anno.japsa as scaffolding proceeds) — https://github.com/mdcao/npScarf
- **ResFinder database** (Reference database used to identify AMR genes in the Illumina draft assembly)
- **SPAdes** (Assembles trimmed Illumina MiSeq reads (--careful, 16 threads) into the draft assembly that is screened for AMR genes)
- **BWA-MEM** (Aligns streaming nanopore reads to the indexed draft assembly with the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000)
- **npReader** (Streams base-called MinION reads in real time (jsa.np.npreader) into the scaffolding/annotation pipeline) — https://github.com/mdcao/npReader
- **jsa.seq.sort** (Sorts SPAdes contigs (-r -n) to produce Kp2146_spades.fasta prior to indexing and AMR screening) — https://github.com/mdcao/japsa

## Examples

```
jsa.np.npreader --realtime --folder Downloads --fail --stat --number --output - | bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y -K 3000 Kp2146_spades.fasta - | jsa.np.npscarf -realtime -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-realtime
```

## Evaluation signals

- AMR gene count in the draft assembly matches the expected 27 ResFinder hits for K. pneumoniae ATCC BAA-2146
- 18 of the 27 AMR genes are reported plasmid-encoded by the time 83 Mb (~14-fold) of nanopore data has been streamed
- No additional plasmid-encoded AMR genes are reported between 83 Mb and 103 Mb (~18-fold), when all four plasmids are circularized
- Real-time mode produces the prefix.anno.japsa annotation file with detected resistance genes and plasmid origins
- Log records, per AMR gene, the cumulative long-read volume at which plasmid-encoded status was declared

## Limitations

- npScarf is no longer maintained; npGraph is the intended replacement
- The pipeline is real-time only at minute-level temporal resolution, not per-base, and still requires short-read data plus a base-caller, which add latency
- Bridge scoring thresholds and annotation calls depend on alignment quality of noisy nanopore reads; determinations are only as good as ResFinder/plasmid-origin database coverage

## Evidence

- [discussion] AMR genes from the assembly were identified using the ResFinder database42.: "AMR genes from the assembly were identified using the ResFinder database42."
- [discussion] npScarf real-time AMR plasmid identification: [evidence span withheld — anchor status: not_found]
- [readme] npScarf outputs annotation file in realtime mode: "In realtime mode, if any annotation analysis is enabled, a file named *prefix*.anno.japsa is generated instead. This file contains features detected after scaffolding."
- [readme] npScarf detects mobile element genes: "It also facilitates the real-time analysis of positional information such as gene ordering and the detection of genes from mobile elements (plasmids and genomic islands)."
- [readme] Real-time streaming command with npreader and npscarf: "jsa.np.npscarf -realtime -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-realtime > log.out 2>&1"
