---
name: de-bruijn-graph-construction
description: Use when assembling de novo transcriptome data and the goal is an isoform-preserving
  assembly graph from Illumina FASTQ reads, rather than a genome-oriented contig set
  that collapses alternative splicing paths.
when_to_use_negative:
- When the input is genomic (single-cell or bacterial) DNA rather than transcriptomic
  RNA-Seq — the standard SPAdes genome pipeline and its graph handling are appropriate
  there
- When the goal is to collapse alternative isoforms into gene-level contigs; this
  graph deliberately preserves branching vertices and multiple distinct paths through
  shared k-mer junctions
- When using k-mer sizes below 29 for transcriptome assembly — small k-values dramatically
  increase erroneous contigs from transcripts sharing k-mers
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: rnaSPAdes
  role: Transcriptome assembler whose pipeline implements the condensed de Bruijn
    graph construction, simplification, paired-read mapping and exSPAnder repeat resolution
    for RNA-Seq data
  repo: github.com/ablab/spades
- name: SPAdes
  role: Genome assembler providing the iterative de Bruijn graph construction and
    spades-gbuilder graph construction tool that rnaSPAdes adapts; unmodified it collapses
    alternative isoforms
  repo: github.com/ablab/spades
- name: spades-gbuilder
  role: Standalone tool used to build the de Bruijn graph from reads at the chosen
    lower k for the condensed graph construction stage
  repo: github.com/ablab/spades
- name: Bandage
  role: GUI visualiser for inspecting the resulting SPAdes assembly graph, including
    connections between contigs, node IDs, lengths and depth
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:e996391c2e10de4d41ea643d6db710d784ea567d71ca012f0d52d626ad21a154
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bandage
  - SPAdes
  - rnaSPAdes
  - spades-gbuilder
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# de-bruijn-graph-construction

## Summary

Builds a condensed de Bruijn assembly graph from RNA-Seq reads at rnaSPAdes-style k-mer sizes, compressing non-branching paths into edges while preserving every branching vertex so that alternatively spliced isoform paths are retained rather than collapsed.

## When to use

Use when assembling de novo transcriptome data and the goal is an isoform-preserving assembly graph from Illumina FASTQ reads, rather than a genome-oriented contig set that collapses alternative splicing paths.

## When NOT to use

- When the input is genomic (single-cell or bacterial) DNA rather than transcriptomic RNA-Seq — the standard SPAdes genome pipeline and its graph handling are appropriate there
- When the goal is to collapse alternative isoforms into gene-level contigs; this graph deliberately preserves branching vertices and multiple distinct paths through shared k-mer junctions
- When using k-mer sizes below 29 for transcriptome assembly — small k-values dramatically increase erroneous contigs from transcripts sharing k-mers

## Inputs

- RNA-Seq reads in FASTQ format (e.g. Human dataset SRR5133163 from NCBI SRA)
- Read length (used to derive upper and lower k-mer sizes)
- Lower-k assembly graph edges (fed with reads into the upper-k iteration)

## Outputs

- Condensed de Bruijn assembly graph serialized as FASTG or GFA 1.0 with DP/KC coverage tags
- before_rr.fasta containing edge sequences with NODE_<id>_length_<L>_cov_<C> headers carrying length and k-mer coverage fields
- Per-edge k-mer coverage values

## How to apply

First select k-mer sizes: the upper k is the largest odd number not exceeding read_length/2 - 1, and the lower k is approximately read_length/3 with a minimum value of 29, since small k-values dramatically increase erroneous contigs from transcripts sharing k-mers. For the lower k, build the de Bruijn graph from the reads (e.g. with spades-gbuilder) and condense non-branching paths into edges while retaining every branching vertex, so paths corresponding to alternatively spliced isoforms are not collapsed. Feed the lower-k edges together with the reads into the upper-k iteration (iterative de Bruijn graph construction, the key SPAdes technique for non-uniform coverage), then condense the final graph and record k-mer coverage per edge. Serialize the condensed graph as FASTG or GFA 1.0 with DP/KC coverage tags and export edge sequences to a FASTA using the NODE_<id>_length_<L>_cov_<C> header convention. Rationale: unlike the genome-oriented SPAdes pipeline, which on a mouse RNA-Seq dataset produced the same number of 95%-assembled genes and isoforms (collapsing alternative isoforms), retaining branching vertices and k-mer coverage enables downstream isoform reconstruction.

## Related tools

- **rnaSPAdes** (Transcriptome assembler whose pipeline implements the condensed de Bruijn graph construction, simplification, paired-read mapping and exSPAnder repeat resolution for RNA-Seq data) — github.com/ablab/spades
- **SPAdes** (Genome assembler providing the iterative de Bruijn graph construction and spades-gbuilder graph construction tool that rnaSPAdes adapts; unmodified it collapses alternative isoforms) — github.com/ablab/spades
- **spades-gbuilder** (Standalone tool used to build the de Bruijn graph from reads at the chosen lower k for the condensed graph construction stage) — github.com/ablab/spades
- **Bandage** (GUI visualiser for inspecting the resulting SPAdes assembly graph, including connections between contigs, node IDs, lengths and depth) — https://github.com/rrwick/Bandage

## Examples

```
spades-gbuilder --input reads.fastq --out graph --k 55
```

## Evaluation signals

- The pipeline runs end-to-end and before_rr.fasta exists and is valid FASTA
- FASTA headers follow the NODE_<id>_length_<L>_cov_<C> convention and carry both length and k-mer coverage fields
- Isoform-retention check: the graph retains multiple distinct paths through shared k-mer junctions (branching vertices preserved, not collapsed)
- Every non-branching path has been condensed into a single edge with recorded k-mer coverage (DP/KC tags in FASTG/GFA output)
- Unmodified SPAdes comparison expectation: on the mouse RNA-Seq dataset it produced the same number of 95%-assembled genes and isoforms, so correct isoform-aware construction should not show this gene/isoform equality artefact

## Limitations

- Small k-mer sizes (e.g. k = 21 in the first iteration) assemble extremely low-covered genes but dramatically increase erroneous contigs from transcripts sharing k-mers; lower k for rnaSPAdes has a minimum of 29
- rnaSPAdes shows higher duplication than original SPAdes (mean duplication ratio 32% vs 2%, duplicated BUSCOs 16% vs 1%), so erroneous and duplicated sequences may negatively affect downstream analysis such as gene annotation
- The approach was designed for RNA-Seq from a single organism; applying it to metatranscriptomes faces unaddressed challenges such as repetitive patterns in homologous genes and extreme differences in mRNA quantities

## Evidence

- [methods] the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction: "the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction"
- [methods] (i) construction of the condensed de Bruijn graph: "(i) construction of the condensed de Bruijn graph"
- [results] Unmodified SPAdes lacks an isoform detection step: on the mouse RNA-Seq dataset it produced the same number of 95%-assembled genes and isoforms: "The simulated Mouse dataset is the only one where the original SPAdes generated more assembled genes and isoforms than rnaSPAdes."
- [intro] we decided to adapt current SPAdes algorithms for RNA-Seq data with the goal of improving the quality of generated assemblies and develop a new transcriptomic assembler called rnaSPAdes: "we decided to adapt current SPAdes algorithms for RNA-Seq data with the goal of improving the quality of generated assemblies and develop a new transcriptomic assembler called rnaSPAdes"
- [readme] SPAdes package includes supplementary tools for efficient k-mer counting and k-mer-based read filtering, assembly graph construction and simplification: "Besides, SPAdes package includes supplementary tools for efficient k-mer counting and k-mer-based read filtering, assembly graph construction and simplification"
