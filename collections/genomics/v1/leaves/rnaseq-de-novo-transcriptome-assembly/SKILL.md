---
name: rnaseq-de-novo-transcriptome-assembly
description: Use when starting from FASTQ RNA-Seq reads (single-organism Illumina
  data, e.g. NCBI SRA accessions like SRR5133163) and a de novo transcriptome assembly
  with metrics such as 95%-assembled genes/isoforms is required.
when_to_use_negative:
- Input is metatranscriptomic data — the approach was initially designed for RNA-Seq
  from a single organism and metatranscriptome assembly quality evaluation remains
  unaddressed
- Input is raw long RNA reads (PacBio/Oxford Nanopore) alone — their high error rate
  prevents using output reads directly as complete transcripts; long reads are only
  suitable as supplementary data in hybrid mode
- Goal is short-read-only analysis with zero duplication — SPAdes/rnaSPAdes output
  carries duplicated sequences (mean duplication ratio 32% for rnaSPAdes vs 2% for
  SPAdes) that can harm downstream gene annotation
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: rnaSPAdes
  role: Primary de novo transcriptome assembler; runs the condensed de Bruijn graph
    pipeline with default multi-k schedule on Illumina RNA-Seq reads
  repo: github.com/ablab/spades
- name: SPAdes
  role: Underlying toolkit; used in single-cell mode (k = 21, 33, 55) as a baseline
    and in hybrid mode (--pacbio/--nanopore) for joint short/long-read assembly
  repo: github.com/ablab/spades
- name: rnaQUAST
  role: Quality evaluation of the assembly against a reference transcriptome (95%-assembled
    genes/isoforms, database coverage, misassemblies, duplication ratio)
- name: Bandage
  role: GUI visualization of the SPAdes assembly graph with connections; consumes
    GFA/.paths output
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:db76225c5647d2d54833aa2d277a5ab3de61c35bf6f7bb9688e3d175b3d1059c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bandage
  - BinPacker
  - Bridger
  - IDBA-tran
  - RNA-Bloom
  - SOAPdenovo-Trans
  - SPAdes
  - Trans-ABySS
  - Trinity
  - rnaQUAST
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.782608695652174
n_claims: 23
---

# rnaseq-de-novo-transcriptome-assembly

## Summary

De novo assembly of Illumina RNA-Seq reads into transcript contigs using rnaSPAdes, a de Bruijn graph assembler built on SPAdes that handles non-uniform coverage via iterative multi-k construction, and subsequent quality evaluation with rnaQUAST.

## When to use

Use when starting from FASTQ RNA-Seq reads (single-organism Illumina data, e.g. NCBI SRA accessions like SRR5133163) and a de novo transcriptome assembly with metrics such as 95%-assembled genes/isoforms is required.

## When NOT to use

- Input is metatranscriptomic data — the approach was initially designed for RNA-Seq from a single organism and metatranscriptome assembly quality evaluation remains unaddressed
- Input is raw long RNA reads (PacBio/Oxford Nanopore) alone — their high error rate prevents using output reads directly as complete transcripts; long reads are only suitable as supplementary data in hybrid mode
- Goal is short-read-only analysis with zero duplication — SPAdes/rnaSPAdes output carries duplicated sequences (mean duplication ratio 32% for rnaSPAdes vs 2% for SPAdes) that can harm downstream gene annotation

## Inputs

- Paired-end Illumina RNA-Seq reads in FASTQ (or BAM) format, e.g. NCBI SRA accession SRR5133163
- Reference transcriptome/annotation for rnaQUAST evaluation

## Outputs

- Assembly graph (GFA 1.0 format and .paths file accepted by Bandage)
- Transcript contigs (contigs.fasta) and scaffolds.fasta
- rnaQUAST report with 95%-assembled genes/isoforms, database coverage, misassemblies, duplication ratio (e.g. CSV/TSV mirroring Table 4)

## How to apply

Run rnaspades.py on paired-end FASTQ reads with default k-mer sizes (lower k ≈ read_length/3 with a minimum of 29; upper k = largest odd number ≤ read_length/2 − 1) and a 200 bp contig cut-off, which triggers the rnaSPAdes pipeline: condensed de Bruijn graph construction, graph simplification (trimming tips, collapsing bulges, removing chimeric connections, and removing isolated edges with coverage <2 and length ≤ read length), mapping read pairs to the graph, and repeat resolution/scaffolding with exSPAnder. Note that read error correction is excluded from the rnaSPAdes pipeline. Evaluate the resulting contigs.fasta/scaffolds.fasta with rnaQUAST against a reference transcriptome to extract 95%-assembled genes/isoforms, database coverage, misassemblies, and duplication ratio. For a baseline comparison, unmodified SPAdes in single-cell mode (k = 21, 33, 55) can be run on the same reads, or a single-k configuration can be tested (fewer genes/isoforms captured but fewer misassemblies).

## Related tools

- **rnaSPAdes** (Primary de novo transcriptome assembler; runs the condensed de Bruijn graph pipeline with default multi-k schedule on Illumina RNA-Seq reads) — github.com/ablab/spades
- **SPAdes** (Underlying toolkit; used in single-cell mode (k = 21, 33, 55) as a baseline and in hybrid mode (--pacbio/--nanopore) for joint short/long-read assembly) — github.com/ablab/spades
- **rnaQUAST** (Quality evaluation of the assembly against a reference transcriptome (95%-assembled genes/isoforms, database coverage, misassemblies, duplication ratio))
- **Bandage** (GUI visualization of the SPAdes assembly graph with connections; consumes GFA/.paths output) — https://github.com/rrwick/Bandage

## Examples

```
rnaspades.py --in1 reads_R1.fastq.gz --in2 reads_R2.fastq.gz -o rnaspades_out && rnaQUAST.py -c rnaspades_out/transcripts.fasta -r transcripts.fa -g genes.gtf -o rnaquast_report
```

## Evaluation signals

- rnaQUAST shows rnaSPAdes at or near the top for fraction of 95%-assembled genes and isoforms (paper-level anchors: 0.50 and 0.32), with Trinity (0.44/0.30) and RNA-Bloom (0.41/0.28) ranked second and third
- On the Human real dataset, rnaSPAdes leads RNA-Bloom and Trinity by ~16% and ~31% in 95%-assembled genes
- Misassembly counts are comparable to competitors (e.g. 309 for rnaSPAdes vs 126 for Trans-ABySS on the simulated Human dataset; Trinity exceeds 5,000 on the real Human dataset)
- Single-k re-runs yield fewer assembled genes/isoforms but fewer misassemblies than the default multi-k run — a consistent directionality check for k-mer ablations
- Default multi-k runs are faster and use less RAM than unmodified SPAdes (~2× speedup)

## Limitations

- No single assembler wins across all quality metrics and datasets (e.g. original SPAdes beats rnaSPAdes on the simulated Mouse dataset, and rnaSPAdes has higher duplication: mean duplication ratio 32% vs 2%)
- Small k-mer sizes such as k = 21 enable assembly of extremely low-covered genes but carry known pitfalls in transcriptome assembly
- High database coverage or nucleotide recall can be achieved by very fragmented assemblies or even raw reads, so these metrics do not reflect contiguity
- Tool choice may require additional per-project benchmarks (Transrate, DETONATE, BUSCO, rnaQUAST); Detonate-vs-rnaQUAST differences (no cutoff on mapped fraction in rnaQUAST) mean contig recall and 95%-assembled isoforms do not perfectly correlate

## Evidence

- [abstract] Herein we describe the novel transcriptome assembler rnaSPAdes, which has been developed on top of the SPAdes genome assembler: "Herein we describe the novel transcriptome assembler rnaSPAdes, which has been developed on top of the SPAdes genome assembler"
- [discussion] rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively).: "rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively)."
- [methods] during graph simplification we additionally remove isolated edges that have both (i) low coverage (<2) and (ii) length smaller than or equal to read length: "during graph simplification we additionally remove isolated edges that have both (i) low coverage (<2) and (ii) length smaller than or equal to read length"
- [methods] In order to run read error correction, reads should be in FASTQ or BAM format: "In order to run read error correction, reads should be in FASTQ or BAM format"
- [readme] SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data.: "SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data."
