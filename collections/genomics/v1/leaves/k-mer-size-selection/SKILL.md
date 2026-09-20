---
name: k-mer-size-selection
description: Use when assembling RNA-Seq reads with rnaSPAdes or SPAdes-style iterative
  de Bruijn graphs and the k-mer schedule (upper k, lower k) must be derived from
  read length to preserve isoform paths while limiting erroneous contigs.
when_to_use_negative:
- Input is already an assembled transcriptome/contigs rather than raw RNA-Seq reads,
  so no k-mer schedule needs to be derived
- The target is genome (or single-cell/metagenome) assembly where genome-oriented
  k choices apply and isoform retention is not the goal
- Reads are extremely short such that read_length/2 - 1 forces a k below 29, where
  small k-mer sizes would dramatically inflate erroneous contigs from transcripts
  sharing k-mers
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: rnaSPAdes
  role: Transcriptome assembler that applies the iterative k-mer graph construction
    adapted for RNA-Seq isoform reconstruction
  repo: github.com/ablab/spades
- name: spades-gbuilder
  role: Standalone tool to build the de Bruijn graph from reads at a chosen lower
    k before condensation
  repo: github.com/ablab/spades
- name: SPAdes
  role: Genome assembler whose iterative de Bruijn graph framework is adapted; used
    unmodified it collapses alternative isoforms
  repo: github.com/ablab/spades
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:587ad6c94ad1c8eaa3205ad882d9cc274e22f39f44aea89f507c3a3e1346eeed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPAdes
  - rnaSPAdes
  - spades-gbuilder
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# k-mer-size-selection

## Summary

Strategy for choosing iterative de Bruijn graph k-mer sizes when reconstructing alternatively spliced isoforms from RNA-Seq reads with rnaSPAdes: an upper k bounded by read_length/2 - 1 and a lower k near read_length/3, never below 29, to balance graph connectivity against erroneous contigs from shared k-mers.

## When to use

Use when assembling RNA-Seq reads with rnaSPAdes or SPAdes-style iterative de Bruijn graphs and the k-mer schedule (upper k, lower k) must be derived from read length to preserve isoform paths while limiting erroneous contigs.

## When NOT to use

- Input is already an assembled transcriptome/contigs rather than raw RNA-Seq reads, so no k-mer schedule needs to be derived
- The target is genome (or single-cell/metagenome) assembly where genome-oriented k choices apply and isoform retention is not the goal
- Reads are extremely short such that read_length/2 - 1 forces a k below 29, where small k-mer sizes would dramatically inflate erroneous contigs from transcripts sharing k-mers

## Inputs

- RNA-Seq reads in FASTQ format (e.g. paired-end Human dataset SRR5133163 from NCBI SRA)
- read_length value from the FASTQ reads, used to derive upper and lower k

## Outputs

- Condensed de Bruijn graph at the lower k (non-branching paths condensed into edges, branching vertices retained)
- Final condensed graph at the upper k, serialized as FASTG or GFA with DP/KC k-mer coverage tags
- Edge sequences exported to before_rr.fasta with NODE_<id>_length_<L>_cov_<C> headers

## How to apply

From the read length of the dataset (e.g. the Human SRR5133163 FASTQ reads), compute the upper k as the largest odd number not exceeding read_length/2 - 1, and the lower k as approximately read_length/3 with a minimum value of 29. Never go below k=29 for the lower iteration, since small k-values dramatically increase erroneous contigs from transcripts sharing k-mers. Build the de Bruijn graph at the lower k from the reads (e.g. with spades-gbuilder or custom code), condensing non-branching paths into edges while retaining every branching vertex so alternatively spliced paths are not collapsed. Feed the lower-k edges together with the reads into the upper-k iteration and condense the final graph, recording k-mer coverage per edge (DP/KC tags). The iterative construction is the key SPAdes technique for data with non-uniform coverage, which RNA-Seq exhibits across transcripts.

## Related tools

- **rnaSPAdes** (Transcriptome assembler that applies the iterative k-mer graph construction adapted for RNA-Seq isoform reconstruction) — github.com/ablab/spades
- **spades-gbuilder** (Standalone tool to build the de Bruijn graph from reads at a chosen lower k before condensation) — github.com/ablab/spades
- **SPAdes** (Genome assembler whose iterative de Bruijn graph framework is adapted; used unmodified it collapses alternative isoforms) — github.com/ablab/spades

## Examples

```
rnaspades.py -1 SRR5133163_1.fastq -2 SRR5133163_2.fastq -k 55,75 -o rnaspades_out
```

## Evaluation signals

- The k schedule matches the rules: upper k is the largest odd number ≤ read_length/2 - 1; lower k ≈ read_length/3 and never below 29
- before_rr.fasta exists, is valid FASTA, and headers carry the NODE_<id>_length_<L>_cov_<C> length and k-mer coverage fields
- The final condensed graph retains multiple distinct paths through shared k-mer junctions (isoform-retention check) rather than collapsing alternatives
- Every edge in the serialized FASTG/GFA carries DP/KC k-mer coverage tags
- The lower-k graph condensation retains every branching vertex; only non-branching paths are condensed into edges

## Limitations

- Small k-mer sizes (e.g. k=21) used in the first iteration assemble extremely low-covered genes with short read overlaps, but pitfalls of small k in transcriptome assembly (dramatic increase in erroneous contigs from transcripts sharing k-mers) are why k is kept ≥ 29
- Original SPAdes used without modification lacks an isoform detection step and produced the same number of 95%-assembled genes and isoforms on the mouse dataset, collapsing alternative isoforms
- No single assembler/k-mer choice wins across all quality metrics and datasets; k selection may require per-project benchmarking

## Evidence

- [methods] the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction: "the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction"
- [discussion] small k pitfalls in transcriptome assembly: "By using small k = 21 during the first iteration SPAdes manages to assemble extremely low-covered genes, where overlaps between reads are short."
- [readme] SPAdes scope includes transcriptomic pipelines: "SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data."
- [methods] construction of the condensed de Bruijn graph is the first pipeline stage: "(i) construction of the condensed de Bruijn graph"
