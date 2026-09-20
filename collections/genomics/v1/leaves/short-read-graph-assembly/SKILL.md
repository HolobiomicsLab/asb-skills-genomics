---
name: short-read-graph-assembly
description: Use when assembling de novo transcriptomes from Illumina RNA-Seq FASTQ
  reads and the goal is a condensed de Bruijn graph whose branching vertices preserve
  multiple paths through shared k-mers (alternative isoforms) rather than collapsing
  them.
when_to_use_negative:
- The goal is reconstructing a single linear genomic sequence rather than multiple
  alternative isoform paths — unmodified genome-oriented SPAdes lacks an isoform detection
  step and collapses alternative isoforms (on the simulated Mouse dataset it produced
  the same number of 95%-assembled genes and isoforms)
- Input reads are high-error-rate long reads (Oxford Nanopore/PacBio) meant to be
  used directly as complete transcripts — the pipeline is designed for accurate short
  Illumina reads
- The data is metatranscriptomic — the approach was designed for RNA-Seq from a single
  organism and repetitive patterns in homologous genes and extreme mRNA quantity differences
  are unaddressed challenges
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: rnaSPAdes
  role: Transcriptome assembler that implements the condensed de Bruijn graph construction
    and RNA-specific graph simplification (tip trimming, bulge collapsing, chimeric
    edge removal) described here
  repo: github.com/ablab/spades
- name: SPAdes
  role: Genome assembler toolkit providing spades-gbuilder for lower-k de Bruijn graph
    construction and the iterative de Bruijn graph machinery rnaSPAdes adapts
  repo: github.com/ablab/spades
- name: Bandage
  role: GUI visualiser for inspecting the resulting assembly graph, nodes labelled
    with ID, length or depth
  repo: https://github.com/rrwick/Bandage
- name: GFA-spec
  role: Specification of the GFA 1.0/2.0 formats used to serialize the condensed graph
  repo: https://github.com/GFA-spec/GFA-spec/blob/master/GFA1.md
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:79f995ebcf79274bcc9e98e14d67c34f26a3bf33d64c21b95601b7784008cdfd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bandage
  - GFA-spec
  - SPAdes
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# short-read-graph-assembly

## Summary

Reconstruct a condensed de Bruijn assembly graph from short RNA-Seq reads (as rnaSPAdes does on top of the SPAdes framework), selecting k-mer sizes from read length, condensing non-branching paths into edges while retaining every branching vertex so alternatively spliced isoform paths are not collapsed.

## When to use

Use when assembling de novo transcriptomes from Illumina RNA-Seq FASTQ reads and the goal is a condensed de Bruijn graph whose branching vertices preserve multiple paths through shared k-mers (alternative isoforms) rather than collapsing them.

## When NOT to use

- The goal is reconstructing a single linear genomic sequence rather than multiple alternative isoform paths — unmodified genome-oriented SPAdes lacks an isoform detection step and collapses alternative isoforms (on the simulated Mouse dataset it produced the same number of 95%-assembled genes and isoforms)
- Input reads are high-error-rate long reads (Oxford Nanopore/PacBio) meant to be used directly as complete transcripts — the pipeline is designed for accurate short Illumina reads
- The data is metatranscriptomic — the approach was designed for RNA-Seq from a single organism and repetitive patterns in homologous genes and extreme mRNA quantity differences are unaddressed challenges

## Inputs

- Illumina RNA-Seq reads in FASTQ (e.g. NCBI SRA run SRR5133163 converted to FASTQ)
- k-mer sizes derived from read length: lower k ~ read_length/3 (minimum 29), upper k = largest odd number <= read_length/2 - 1

## Outputs

- Condensed de Bruijn assembly graph serialized in FASTG or GFA 1.0 format with DP/KC k-mer-coverage tags
- Edge sequence file before_rr.fasta with NODE_<id>_length_<L>_cov_<C> headers
- Per-edge k-mer coverage values recorded for downstream isoform/path analysis

## How to apply

Start from RNA-Seq reads in FASTQ (e.g. download from NCBI SRA such as SRR5133163 and convert). Choose the rnaSPAdes k-mer sizes: the upper k as the largest odd number not exceeding read_length/2 - 1, and the lower k approximately read_length/3 with a minimum value of 29, because small k-values dramatically increase erroneous contigs from transcripts sharing k-mers. For the lower k, build the de Bruijn graph from the reads (e.g. with spades-gbuilder or custom code) and condense non-branching paths into edges while retaining every branching vertex, so paths corresponding to alternatively spliced isoforms are not collapsed. Feed the lower-k edges together with the reads into the upper-k iteration (the iterative de Bruijn graph construction that lets SPAdes handle non-uniform coverage) and condense the final graph, recording k-mer coverage per edge. Serialize the final condensed graph in FASTG or GFA 1.0 format with DP/KC coverage tags and export edge sequences to before_rr.fasta using the NODE_<id>_length_<L>_cov_<C> header convention; graph simplification (tip trimming, bulge collapsing, chimeric connection removal, isolated-edge removal for coverage <2 and length <= read length) precedes repeat resolution.

## Related tools

- **rnaSPAdes** (Transcriptome assembler that implements the condensed de Bruijn graph construction and RNA-specific graph simplification (tip trimming, bulge collapsing, chimeric edge removal) described here) — github.com/ablab/spades
- **SPAdes** (Genome assembler toolkit providing spades-gbuilder for lower-k de Bruijn graph construction and the iterative de Bruijn graph machinery rnaSPAdes adapts) — github.com/ablab/spades
- **Bandage** (GUI visualiser for inspecting the resulting assembly graph, nodes labelled with ID, length or depth) — https://github.com/rrwick/Bandage
- **GFA-spec** (Specification of the GFA 1.0/2.0 formats used to serialize the condensed graph) — https://github.com/GFA-spec/GFA-spec/blob/master/GFA1.md

## Examples

```
rnaspades.py -1 SRR5133163_R1.fastq -2 SRR5133163_R2.fastq -o rnaspades_out  # then inspect K55/before_rr.fasta with NODE_<id>_length_<L>_cov_<C> headers
```

## Evaluation signals

- The script runs end-to-end and before_rr.fasta exists, is valid FASTA, and every header carries length and k-mer coverage fields in the NODE_<id>_length_<L>_cov_<C> convention
- The graph retains multiple distinct paths through shared k-mer junctions (branching vertices survive condensation) — the isoform-retention check
- Coverage tags (DP/KC in GFA or cov fields in FASTG) are populated for every edge and consistent with k-mer coverage recorded during the upper-k iteration
- GFA output parses against the GFA 1.0 specification and loads in Bandage/Bandage-NG for visual inspection
- Benchmark comparison: the assembly should approach rnaSPAdes' reported highest fractions of 95%-assembled genes (0.50) and isoforms (0.32) rather than the collapsed gene==isoform counts of unmodified SPAdes

## Limitations

- Unmodified SPAdes lacks an isoform detection step and collapses alternative isoforms instead of preserving them; rnaSPAdes in turn has higher duplication (mean duplication ratio 32% vs 2% for SPAdes) and lower nucleotide precision
- Using small k-mer sizes (e.g. k=21) in the first iteration dramatically increases erroneous contigs from transcripts sharing k-mers; the lower k must stay >= 29
- RNA-Seq read coverage is highly uneven, which is why iterative de Bruijn graph construction (borrowed from SPAdes' single-cell pedigree) is required; no single k-mer size suffices
- The graph simplification choices (tip alignment to alternative edges, overlap removal restricted to exact duplicates and subpaths) are transcriptome-specific adaptations; genome-style simplification would collapse isoform-relevant structure

## Evidence

- [methods] (i) construction of the condensed de Bruijn graph: "(i) construction of the condensed de Bruijn graph"
- [methods] iterative dbg rationale: "the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction"
- [discussion] isoform collapse finding: "rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively)."
- [methods] simplification steps: "(ii) graph simplification, which implies removing chimeric and erroneous edges"
- [readme] spades toolkit scope: "SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data."
