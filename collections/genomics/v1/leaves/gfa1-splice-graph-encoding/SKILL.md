---
name: gfa1-splice-graph-encoding
description: Use when a de novo transcriptome must be represented as a splice graph
  (exons = nodes, IEBs = edges, transcripts = paths) for downstream SNP genotyping,
  visualization in Bandage, or comparison against a GFF3 reference annotation.
when_to_use_negative:
- A reference genome and splice-aware alignment (e.g. GMAP) is available to extract
  IEB coordinates directly
- Input is raw RNA-Seq reads rather than an assembled transcriptome (EXFI takes the
  transcriptome assembly plus raw WGS reads)
- Output must be coordinates-only BED/GFF rather than a graph encoding exon sequences
  and connections
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0080
tools:
- name: EXFI
  role: build_splice_graph script performs exon/splice-graph prediction and writes
    the GFA1 output
  repo: https://github.com/jlanga/exfi
- name: abyss-bloom
  role: builds the cascading Bloom filter of genomic k-mers queried during splice
    graph construction
  repo: https://github.com/bcgsc/abyss/
- name: GFA-spec
  role: defines the GFA1 format used to encode exon segments, connections, and transcript
    paths
  repo: https://github.com/GFA-spec/GFA-spec/blob/master/GFA1.md
- name: EXFI docker
  role: containerized exfi:v1.5.6 image for running build_splice_graph reproducibly
  repo: https://github.com/jlanga/exfi-docker
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:b073970effeaba321b62cc32a1d55cfcec8023d8badcd2d8da2d944288790821
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BioPython
  - EXFI
  - EXFI docker
  - GFA-spec
  - Pandas
  - Python
  - abyss-bloom
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# gfa1-splice-graph-encoding

## Summary

Encode EXFI's predicted exons and their connectivity into a GFA1 splice graph file: exon sequences as S segments with coordinates, exon connections as L lines, and transcripts as P-line paths of exons.

## When to use

Use when a de novo transcriptome must be represented as a splice graph (exons = nodes, IEBs = edges, transcripts = paths) for downstream SNP genotyping, visualization in Bandage, or comparison against a GFF3 reference annotation.

## When NOT to use

- A reference genome and splice-aware alignment (e.g. GMAP) is available to extract IEB coordinates directly
- Input is raw RNA-Seq reads rather than an assembled transcriptome (EXFI takes the transcriptome assembly plus raw WGS reads)
- Output must be coordinates-only BED/GFF rather than a graph encoding exon sequences and connections

## Inputs

- Assembled transcriptome FASTA (de novo assembly)
- Baited/cascading Bloom filter built from WGS reads with abyss-bloom at k-mer length 25 (e.g. genome_k25_m100M_l1.bloom)

## Outputs

- GFA1 splice graph file (test.gfa) with H, S, L, C and P records: exon segments with sequence and coordinates, exon connections, and transcript paths

## How to apply

After build_splice_graph scans transcriptomic k-mers (k=25) against the baited/cascading Bloom filter, merges overlapping positives into exons, filters exons shorter than k+q (q default 5), and applies relaxed merging for overlaps >10 bases, write the result as a GFA1 document: S lines carry exon sequence and coordinates, L lines carry exon connections, and P lines encode each transcript as a path of exon nodes. Optionally run the --polish step first, which inspects overlapping exon pairs for donor/acceptor sites (usually GU/AG) and trims them. Validate the output by confirming the .gfa file contains H/S/L/C/P records with coordinates consistent with the input transcriptome, and inspect it in Bandage or score it with compare_to_gff3 against a reference annotation when available.

## Related tools

- **EXFI** (build_splice_graph script performs exon/splice-graph prediction and writes the GFA1 output) — https://github.com/jlanga/exfi
- **abyss-bloom** (builds the cascading Bloom filter of genomic k-mers queried during splice graph construction) — https://github.com/bcgsc/abyss/
- **GFA-spec** (defines the GFA1 format used to encode exon segments, connections, and transcript paths) — https://github.com/GFA-spec/GFA-spec/blob/master/GFA1.md
- **EXFI docker** (containerized exfi:v1.5.6 image for running build_splice_graph reproducibly) — https://github.com/jlanga/exfi-docker

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_splice_graph --kmer-length 25 --bloom genome_k25_m100M_l1.bloom transcriptome.fa -o test.gfa
```

## Evaluation signals

- Output test.gfa exists and parses as a valid GFA1 document containing H, S, L, C and P records
- S-line exon sequences carry coordinates consistent with the input transcriptome, and every transcript appears as a P-line path of exon nodes
- Graph is visualizable in Bandage and its connected components correspond to genes (exons = nodes, IEBs = edges, transcripts = paths)
- When a reference GFF3 annotation exists, compare_to_gff3 scores the predicted exons; exon precision/mappability metrics (e.g. 99.5% mappability) confirm usability
- Exon lengths respect the k+q (q=5) minimum filter, so no disconnected k-length exons remain from Bloom-filter false positives

## Limitations

- High mappability (99.5% in herring) is more optimistic than exon precision: it covers only interior exon sections, not full exon sequences
- Mappability and exon prediction fall when the input transcriptome is de novo-assembled and single-tissue (e.g. Atlantic herring), where lowly expressed transcripts are fragmented
- Bloom filters can produce false positives at a tunable BF FPR; read filtering reduced the BF FPR by 68%-75% but false-positive k-mers must still be removed via the exon length filter
- No software version numbers for EXFI/ChopStitch/GMAP are reported in the comparison sections, complicating exact reproduction

## Evidence

- [abstract] GFA1 output encoding exon segments: "The results are returned in GFA1 format, which encodes both the predicted exon sequences and how they are connected to form transcripts"
- [intro] splice graph mathematical representation: "a mathematical representation of a transcriptome where exons are represented by nodes, IEBs as edges, transcripts as paths, and genes as the different connected components"
- [methods] GFA1 format generation: "resulting splice graph in [GFA1 format](https://github.com/GFA-spec/GFA-spec/blob/master/GFA1.md)"
- [methods] exon prediction rules: "exons of length less than k + q (q by default five) are filtered out (Figure 2c)"
- [readme] GFA1 implementability: [evidence span withheld — anchor status: not_found]
- [abstract] EXFI repository: "the source code is available under the MIT License at https://github.com/jlanga/exfi"
- [readme] EXFI docker invocation: [evidence span withheld — anchor status: not_found]
