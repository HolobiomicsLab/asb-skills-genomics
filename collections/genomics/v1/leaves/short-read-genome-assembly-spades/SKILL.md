---
name: short-read-genome-assembly-spades
description: Use when starting a hybrid assembly from Illumina MiSeq paired-end FASTQ
  (e.g. ERR654977, Kp2146_paired_*.fastq.gz) that will subsequently be completed with
  long nanopore reads via npScarf.
when_to_use_negative:
- When the goal is a pure long-read assembly — the paper instead compares against
  assemblers 'designed for long and accurate reads such as Celera Assembler' for corrected
  long reads
- When reads are untrimmed raw MiSeq output — the pipeline requires adapter- and quality-trimmed
  input before SPAdes
- When the target is npGraph-based streaming assembly — npScarf's README states it
  'is not on maintenance anymore' and npGraph is the replacement
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: SPAdes
  role: Assembles trimmed Illumina MiSeq paired-end reads into the draft contig set
    (spades/contigs.fasta) used as the npScarf scaffold backbone
- name: trimmomatic
  role: Trims Illumina read data (V0.32) to remove adapters and low-quality bases
    prior to SPAdes assembly
- name: japsa (jsa.seq.sort)
  role: Sorts SPAdes contigs in reverse length order ('jsa.seq.sort -r -n') to produce
    the draft FASTA expected by npScarf
  repo: https://github.com/mdcao/japsa
- name: bwa
  role: Builds the alignment index ('bwa index') on the sorted draft so BWA-MEM can
    align streamed nanopore reads for scaffolding (bwa >= 7.11 required)
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:d5b67cc36446bf6e665bfeb41dfd761bcc64eedcd1a63591d4b216cf1bb9c71f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/japsa
  tools:
  - BWA-MEM
  - SPAdes
  - bwa
  - japsa (jsa.seq.sort)
  - npReader
  - npScarf
  - trimmomatic
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# short-read-genome-assembly-spades

## Summary

Assemble trimmed Illumina MiSeq paired-end reads into a draft contig set with SPAdes, then length-sort contigs and build a BWA index so the draft can serve as the backbone for npScarf's real-time nanopore scaffolding.

## When to use

Use when starting a hybrid assembly from Illumina MiSeq paired-end FASTQ (e.g. ERR654977, Kp2146_paired_*.fastq.gz) that will subsequently be completed with long nanopore reads via npScarf.

## When NOT to use

- When the goal is a pure long-read assembly — the paper instead compares against assemblers 'designed for long and accurate reads such as Celera Assembler' for corrected long reads
- When reads are untrimmed raw MiSeq output — the pipeline requires adapter- and quality-trimmed input before SPAdes
- When the target is npGraph-based streaming assembly — npScarf's README states it 'is not on maintenance anymore' and npGraph is the replacement

## Inputs

- Trimmed Illumina MiSeq paired-end FASTQ files (e.g. ENA ERR654977 or Kp2146_paired_1.fastq.gz / Kp2146_paired_2.fastq.gz)
- Trimmed reads produced by trimmomatic V0.32 (adapters and low-quality bases removed)

## Outputs

- SPAdes draft assembly FASTA (spades/contigs.fasta)
- Length-sorted draft contigs FASTA (e.g. Kp2146_spades.fasta, sorted with jsa.seq.sort -r -n)
- BWA index files for the sorted draft (bwa index Kp2146_spades.fasta)

## How to apply

First trim the Illumina reads with trimmomatic V0.32 to remove adapters and low-quality bases, since the README stresses 'It is essential that the reads are trimmed to remove all adaptors.' Assemble the trimmed paired-end FASTQ with SPAdes in multi-threaded mode (>= 3.5), optionally with --careful to reduce errors at some runtime cost — the README notes the improvement 'is not so significant' and it can safely be excluded to save time. Take spades/contigs.fasta, sort the contigs in reverse length order with 'jsa.seq.sort -r -n' (npScarf expects the draft sorted by decreasing length and uses the largest contigs to estimate genome depth), then create the BWA index with 'bwa index'. The resulting draft FASTA becomes the -seq input to jsa.np.npscarf, where contig short-read coverage versus whole-genome coverage is later used to determine contig multiplicity (unique vs repetitive).

## Related tools

- **SPAdes** (Assembles trimmed Illumina MiSeq paired-end reads into the draft contig set (spades/contigs.fasta) used as the npScarf scaffold backbone)
- **trimmomatic** (Trims Illumina read data (V0.32) to remove adapters and low-quality bases prior to SPAdes assembly)
- **japsa (jsa.seq.sort)** (Sorts SPAdes contigs in reverse length order ('jsa.seq.sort -r -n') to produce the draft FASTA expected by npScarf) — https://github.com/mdcao/japsa
- **bwa** (Builds the alignment index ('bwa index') on the sorted draft so BWA-MEM can align streamed nanopore reads for scaffolding (bwa >= 7.11 required))

## Examples

```
spades.py --careful --pe1-1 Kp2146_paired_1.fastq.gz --pe1-2 Kp2146_paired_2.fastq.gz -o spades -t 16; jsa.seq.sort -r -n --input spades/contigs.fasta --output Kp2146_spades.fasta; bwa index Kp2146_spades.fasta
```

## Evaluation signals

- spades/contigs.fasta exists and contains contigs spanning the expected genome size (e.g. ~4.6 Mb for E. coli K12 MG1655, ~5.4 Mb N50 chromosome plus plasmids for K. pneumoniae Kpn2146)
- The sorted draft (e.g. Kp2146_spades.fasta) is ordered by decreasing contig length, matching the 'jsa.seq.sort -r -n' output
- BWA index files (e.g. Kp2146_spades.fasta.bwt, .pac, .ann, .amb, .sa) are generated for the sorted draft
- Downstream npScarf scaffolding completes successfully on the draft, producing a prefix.fin.fasta (e.g. Kp2146-realtime.fin.fasta)
- QUAST metrics of the final scaffold are consistent with the paper's deposited reports (e.g. data.genomicsresearch.org QUAST report for Kpn2146)

## Limitations

- SPAdes --careful 'would help to reduce the errors but the improvement is not so significant' and can be omitted to save running time
- This skill only produces the short-read draft; completion to a single circular contig additionally requires nanopore data and npScarf (competitors LINKS, NaS, Nanocorr failed even on 67-fold nanopore coverage for E. coli)
- npScarf, which consumes this draft, is no longer maintained; npGraph (hsnguyen/assembly) is its replacement

## Evidence

- [discussion] Read data from Illumina sequencing were trimmed with trimmomatic V0.32 (ref. 41): "Read data from Illumina sequencing were trimmed with trimmomatic V0.32 (ref. 41)"
- [discussion] npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome.: "npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome."
- [readme] Assemble the Illumina data with SPAdes using 16 threads in parallel. Option --careful would help to reduce the errors: "Assemble the Illumina data with SPAdes using 16 threads in parallel. Option --careful would help to reduce the errors but the improvement is not so significant."
- [other] Download the Illumina MiSeq paired-end reads (ENA ERR654977) and the R7.3 MinION run (ENA ERR637419): [evidence span withheld — anchor status: not_found]
- [readme] It is essential that the reads are trimmed to remove all adaptors. Low quality bases should also be removed.: "It is essential that the reads are trimmed to remove 
all adaptors. Low quality bases should also be removed."
