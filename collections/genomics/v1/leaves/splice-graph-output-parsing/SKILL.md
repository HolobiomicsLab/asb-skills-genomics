---
name: splice-graph-output-parsing
description: Use when you have run EXFI's build_splice_graph (or need to consume its
  output) and must convert the GFA1 splice graph into usable exon sequences/coordinates
  for benchmarking or annotation.
when_to_use_negative:
- The output file is not in GFA1 format (e.g. you need full transcript-to-genome alignments
  with mapping locations — use a splice-aware aligner like GMAP instead)
- You need exon coordinates relative to a reference genome; EXFI/GFA1 segments give
  exon sequences without chromosome coordinates, so reference-based GFF/BED conversion
  must come from the reference annotation
- Segments shorter than the k-mer window (exons < k + q, q default 5) that EXFI's
  own filtering already removed would yield unreliable exon calls
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0114
tools:
- name: EXFI
  role: Source of the GFA1 splice graph output and of the gfa1_to_fasta script that
    extracts exon sequences from it
  repo: https://github.com/jlanga/exfi
- name: exfi-docker
  role: Docker container (exfi:1.5.6) bundling build_splice_graph and gfa1_to_fasta
    for running the parsing step reproducibly
  repo: https://github.com/jlanga/exfi-docker
- name: GFA-spec
  role: Reference specification for interpreting the GFA1 segment/link/path structure
    of the splice graph
  repo: https://github.com/GFA-spec/GFA-spec
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:50369bb881a1b4a5402e2034435a4ba7dd623d29a8d71df5b8fe131b814da44e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BioPython
  - EXFI
  - GFA-spec
  - Pandas
  - Python
  - bedtools
  - exfi-docker
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# splice-graph-output-parsing

## Summary

Parse EXFI's splice graph output, which is emitted in GFA1 format encoding predicted exon sequences as segments and their connections into transcripts as paths, then extract exon sequences (e.g. via gfa1_to_fasta) for downstream use such as BEDTools-based coordinate comparison.

## When to use

Use when you have run EXFI's build_splice_graph (or need to consume its output) and must convert the GFA1 splice graph into usable exon sequences/coordinates for benchmarking or annotation.

## When NOT to use

- The output file is not in GFA1 format (e.g. you need full transcript-to-genome alignments with mapping locations — use a splice-aware aligner like GMAP instead)
- You need exon coordinates relative to a reference genome; EXFI/GFA1 segments give exon sequences without chromosome coordinates, so reference-based GFF/BED conversion must come from the reference annotation
- Segments shorter than the k-mer window (exons < k + q, q default 5) that EXFI's own filtering already removed would yield unreliable exon calls

## Inputs

- GFA1 splice graph file produced by EXFI's build_splice_graph (segments = exon sequences, edges = IEBs, paths = transcripts)
- EXFI installation or exfi:1.5.6 Docker image providing gfa1_to_fasta

## Outputs

- FASTA file of predicted exon sequences extracted from the GFA1 segments
- Splice graph structure (exon connections/paths) usable for transcript and gene-level interpretation
- Exon coordinate sets for downstream comparison (e.g. BEDTools intersect against transcript-based BED references)

## How to apply

Run build_splice_graph, which 'predicts in one step the exon sequences and the splice graph', and it writes the splice graph in GFA1 format (per the GFA-spec: segments, links, and paths describe exons, IEBs, and transcripts). Treat each segment as a predicted exon and paths as transcript routes; connected components correspond to genes. Extract exon sequences from the GFA1 with EXFI's gfa1_to_fasta utility (available in the exfi:1.5.6 Docker container) so they can be compared against reference exons, e.g. with BEDTools intersect requiring mutual overlap of at least 95%. Note that EXFI already filters out exons shorter than k + q (q default 5) and trims overlapping exon pairs at donor/acceptor GU/AG sites, so short or overlap-merged segments should not be re-parsed as independent exons.

## Related tools

- **EXFI** (Source of the GFA1 splice graph output and of the gfa1_to_fasta script that extracts exon sequences from it) — https://github.com/jlanga/exfi
- **exfi-docker** (Docker container (exfi:1.5.6) bundling build_splice_graph and gfa1_to_fasta for running the parsing step reproducibly) — https://github.com/jlanga/exfi-docker
- **GFA-spec** (Reference specification for interpreting the GFA1 segment/link/path structure of the splice graph) — https://github.com/GFA-spec/GFA-spec

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 gfa1_to_fasta --help
```

## Evaluation signals

- Output FASTA sequence headers/lengths correspond 1:1 to segment entries in the GFA1 file
- GFA1 link/path structure reproduces the expected splice graph topology: exons as nodes, IEBs as edges, transcripts as paths, genes as connected components
- No extracted exon is shorter than k + q (q default 5), matching EXFI's exon length filter
- Extracted exon sequences overlap reference exons by ≥ 95% mutually in BEDTools intersect, yielding the precision/recall/F1 values tabulated per k-mer length by compare_to_gff3

## Limitations

- GFA1 encodes exon sequences and connections but not genomic mapping locations, so mappability-style assessment requires downstream alignment
- EXFI's exon prediction (and hence the parsed graph) depends on input transcriptome quality; fragmented or single-tissue transcriptomes yield less accurate exons (e.g. the Atlantic herring case)
- Bloom filter false positives (BF FPR) can propagate into the graph, though read filtering reduced the BF FPR by 68%–75%

## Evidence

- [abstract] The results are returned in GFA1 format, which encodes both the predicted exon sequences and how they are connected to form transcripts: "The results are returned in GFA1 format, which encodes both the predicted exon sequences and how they are connected to form transcripts"
- [methods] The exon and splice graph prediction procedure is carried out by the build_splice_graph script, which predicts in one step the exon sequences: "The exon and splice graph prediction procedure is carried out by the build_splice_graph script, which predicts in one step the exon sequences"
- [methods] exons of length less than k + q (q by default five) are filtered out: "exons of length less than k + q (q by default five) are filtered out"
- [readme] GFA 1.0 spec is maintained in the GFA-spec repository: [evidence span withheld — anchor status: not_found]
- [readme] EXFI Docker container exposes the pipeline commands: [evidence span withheld — anchor status: not_found]
