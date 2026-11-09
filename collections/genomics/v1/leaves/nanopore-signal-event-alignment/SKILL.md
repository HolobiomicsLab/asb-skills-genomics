---
name: nanopore-signal-event-alignment
description: Use when starting from raw nanopore FAST5 files and you need event-level
  signal-to-reference alignments (test.eventalign.txt) as input for nanoHiMe modification
  calling or model training.
when_to_use_negative:
- Input is already a nanopolish eventalign table — skip directly to nanoHiMe 6mA/mCG
  calling or Model_training
- The target modification caller requires raw signal models directly (e.g. Megalodon
  v2.5.0 runs on FAST5/BAM with res_dna_r941_min_modbases-all-context_v001.cfg and
  does not need the nanopolish eventalign step)
- Reads are not ONT R9.4.1 data (the dna_r9.4.1_450bps_hac.cfg basecalling model and
  k-mer event alignment are signal chemistry specific)
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3295
tools:
- name: Guppy v4.4.2
  role: Basecalls FAST5 signal files to FASTQ using the 'high accuracy' dna_r9.4.1_450bps_hac.cfg
    model with qscore filtering (GPU or 16 CPU threads)
- name: nanopolish v0.13.2
  role: Indexes reads to signal data (nanopolish index -d fast5) and aligns electric
    current events to reference k-mers with --scale-events to produce test.eventalign.txt
- name: Minimap2 v2.17
  role: Aligns basecalled reads to hg19 or E. coli reference with the map-ont preset
    and 8 threads
- name: Samtools v1.9
  role: Sorts/indexes BAMs and filters to mapped reads with mapping quality >=20 (samtools
    view -b -q 20 -F 4) before event alignment
- name: nanoHiMe
  role: Downstream consumer of the eventalign table for 6mA region calling, mCG calling,
    and HMM emission-parameter training
  repo: github.com/YinLabTJ/nanoHiMe
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:bb10a0bc7bbd0f48261857d14128d3e8c4b45be4c24f2a48d0e7a99c1200130c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - Guppy v4.4.2
  - Minimap2
  - Minimap2 v2.17
  - Samtools v1.9
  - nanoHiMe
  - nanopolish
  - nanopolish v0.13.2
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# nanopore-signal-event-alignment

## Summary

Pipeline for converting Oxford Nanopore FAST5 signal data into per-event, k-mer-level alignments: Guppy basecalling, nanopolish read-to-signal indexing, minimap2 alignment with MAPQ≥20 filtering, and nanopolish eventalign with --scale-events to produce the eventalign tables that nanoHiMe requires for 6mA and CpG methylation calling.

## When to use

Use when starting from raw nanopore FAST5 files and you need event-level signal-to-reference alignments (test.eventalign.txt) as input for nanoHiMe modification calling or model training.

## When NOT to use

- Input is already a nanopolish eventalign table — skip directly to nanoHiMe 6mA/mCG calling or Model_training
- The target modification caller requires raw signal models directly (e.g. Megalodon v2.5.0 runs on FAST5/BAM with res_dna_r941_min_modbases-all-context_v001.cfg and does not need the nanopolish eventalign step)
- Reads are not ONT R9.4.1 data (the dna_r9.4.1_450bps_hac.cfg basecalling model and k-mer event alignment are signal chemistry specific)

## Inputs

- Raw nanopore FAST5 signal files (e.g. example_hg19.fast5 from HepG2 nanoHiMe-seq, or E. coli K12 MG1655 PCR-amplicon FAST5)
- Reference genome FASTA (hg19 from UCSC or Ecoli_K12_MG1655)
- Guppy-basecalled FASTQ (test.fastq)

## Outputs

- test.fastq.index files from nanopolish index (test.fastq.index, .fai, .gzi, .readdb)
- test.sorted.q20.mapped.bam (mapped, MAPQ>=20 sorted alignment)
- test.eventalign.txt — nanopolish event-level alignment table used by nanoHiMe

## How to apply

Basecall FAST5 files with Guppy v4.4.2 using the 'high accuracy' model (dna_r9.4.1_450bps_hac.cfg) and concatenate the per-file FASTQs into one test.fastq. Index basecalled reads to their signal data with 'nanopolish index -d fast5 test.fastq'. Align reads to the reference (hg19 or E. coli K12 MG1655) with minimap2 v2.17 (-ax map-ont, 8 threads), sort/index with samtools v1.9, then retain only mapped reads with mapping quality >=20 via 'samtools view -b -q 20 -F 4'. Produce the event-level alignment with 'nanopolish eventalign -n -t 16 --scale-events' so that nanopore electrical events are scaled and matched to reference k-mers; this eventalign file is the mandatory input for nanoHiMe's 6mA, mCG, and Model_training modes.

## Related tools

- **Guppy v4.4.2** (Basecalls FAST5 signal files to FASTQ using the 'high accuracy' dna_r9.4.1_450bps_hac.cfg model with qscore filtering (GPU or 16 CPU threads))
- **nanopolish v0.13.2** (Indexes reads to signal data (nanopolish index -d fast5) and aligns electric current events to reference k-mers with --scale-events to produce test.eventalign.txt)
- **Minimap2 v2.17** (Aligns basecalled reads to hg19 or E. coli reference with the map-ont preset and 8 threads)
- **Samtools v1.9** (Sorts/indexes BAMs and filters to mapped reads with mapping quality >=20 (samtools view -b -q 20 -F 4) before event alignment)
- **nanoHiMe** (Downstream consumer of the eventalign table for 6mA region calling, mCG calling, and HMM emission-parameter training) — github.com/YinLabTJ/nanoHiMe

## Examples

```
nanopolish index -d fast5 test.fastq && minimap2 -ax map-ont -t 8 hg19.fa test.fastq | samtools sort -o test.sorted.bam && nanopolish eventalign -n -t 16 --reads test.fastq --bam test.sorted.q20.mapped.bam --genome hg19.fa --scale-events > test.eventalign.txt
```

## Evaluation signals

- nanopolish index produces all four expected index artifacts: test.fastq.index, .fai, .gzi, and .readdb
- The filtered BAM contains only mapped, MAPQ>=20 reads (samtools view -b -q 20 -F 4 was applied and re-indexed)
- test.eventalign.txt is tab-delimited with event-to-reference-k-mer rows and is accepted by nanoHiMe (optionally gzipped) as the -e input to Model_training or the eventalign argument to 6mA/mCG modes
- nanoHiMe downstream calls succeed only when the reference was uppercased (upper.pl) and samtools-faidx indexed, confirming the eventalign pipeline fed a compatible reference
- HMM training on the eventalign file yields EM.out with per-6-mer Gaussian parameters (e.g. AAAAAA omega1 ~0.926, mu1 ~86.45, sigma1 ~1.21)

## Limitations

- nanoHiMe's event-based calling requires reads pre-processed with nanopolish eventalign; Megalodon and Guppy use their own signal models, so eventalign output is not interchangeable between callers
- The full FAST5-to-eventalign chain (basecalling, indexing, alignment, eventalign) must be repeated for each subsampled dataset, e.g. when evaluating nanoHiMe-seq sensitivity at 5x-50x coverage
- Filtering to MAPQ>=20 before event alignment restricts analysis to confidently placed reads; low-mappability regions will lose coverage

## Evidence

- [discussion] Basecalling was performed using Guppy v4.4.2 with the “high accuracy” model.: "Basecalling was performed using Guppy v4.4.2 with the “high accuracy” model."
- [discussion] The alignment of the electric current events to k-mers of a known substring from the reference genome was obtained using nanopolish v0.13.2.: "The alignment of the electric current events to k-mers of a known substring from the reference genome was obtained using nanopolish v0.13.2."
- [readme] samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [readme] nanopolish eventalign -n -t 16 --reads test.fastq --bam test.sorted.q20.mapped.bam --genome hg19.fa --scale-events > test.eventalign.txt: "nanopolish eventalign -n -t 16 --reads test.fastq --bam test.sorted.q20.mapped.bam --genome hg19.fa --scale-events > test.eventalign.txt"
