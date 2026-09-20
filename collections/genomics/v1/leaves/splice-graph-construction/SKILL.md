---
name: splice-graph-construction
description: Use when you have a de novo transcriptome assembly (FASTA) plus raw whole-genome
  sequencing reads (or an abyss-bloom filter like genome_k25_m100M_l1.bloom) and need
  exon sequences and splice-graph topology without a reference genome.
when_to_use_negative:
- A reference genome and annotation exist and splice-aware alignment is preferable
  — the article notes the alternative is to align transcriptome or RNA-Seq reads with
  a splice-aware aligner such as GMAP and extract IEB coordinates
- The input transcriptome is poor quality or from a single tissue with fragmented
  lowly expressed transcripts (the herring case, where EXFI was less accurate and
  mappability fell for all methods)
- The Bloom filter was built with a different k-mer length than the one passed to
  build_splice_graph
tools:
- name: EXFI
  role: Provides the build_splice_graph script that performs k-mer overlap detection,
    IEB marking, exon merging, filtering, polishing, and GFA1 output
  repo: https://github.com/jlanga/exfi
- name: abyss-bloom
  role: Builds the cascading Bloom filter of genomic k-mers that transcriptomic k-mers
    are scanned against
  repo: https://github.com/bcgsc/abyss/
- name: BioBloom Tools
  role: biobloomcategorizer filters WGS reads that do not share at least one k-mer
    with the transcriptome before the filter is built
  repo: https://github.com/bcgsc/biobloom
- name: Python
  role: Implementation language of EXFI's core programs, on top of Pandas and BioPython
  repo: https://github.com/jlanga/exfi
- name: Pandas
  role: Data-processing dependency used underneath EXFI's core programs
- name: BioPython
  role: Bioinformatics dependency used underneath EXFI's core programs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:e6c03791a86cb46476a4064109271dcdc049810c21fb779e8a777b11c71b658e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BioBloom Tools
  - BioPython
  - EXFI
  - Pandas
  - Python
  - abyss-bloom
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# splice-graph-construction

## Summary

Reconstruct a transcriptome's exon structure as a GFA1 splice graph with EXFI's build_splice_graph: transcriptomic k-mers (k=25) are scanned against a baited/cascading Bloom filter of WGS genomic k-mers, absent k-mers mark potential intron–exon boundaries (IEBs), and overlapping positive k-mers are merged into exons connected into a splice graph.

## When to use

Use when you have a de novo transcriptome assembly (FASTA) plus raw whole-genome sequencing reads (or an abyss-bloom filter like genome_k25_m100M_l1.bloom) and need exon sequences and splice-graph topology without a reference genome.

## When NOT to use

- A reference genome and annotation exist and splice-aware alignment is preferable — the article notes the alternative is to align transcriptome or RNA-Seq reads with a splice-aware aligner such as GMAP and extract IEB coordinates
- The input transcriptome is poor quality or from a single tissue with fragmented lowly expressed transcripts (the herring case, where EXFI was less accurate and mappability fell for all methods)
- The Bloom filter was built with a different k-mer length than the one passed to build_splice_graph

## Inputs

- Assembled transcriptome FASTA (de novo transcriptome assembly)
- Baited/cascading Bloom filter of genomic k-mers built with abyss-bloom (e.g. genome_k25_m100M_l1.bloom), with k-mer length matching the filter
- Optional --polish flag for GU/AG donor/acceptor trimming

## Outputs

- GFA1 splice graph file (test.gfa) with H/S/L/C/P records: exon segments (S lines with sequence and coordinates), exon connections (L lines), and transcripts as paths of exons (P lines)

## How to apply

Run EXFI's build_splice_graph on the transcriptome FASTA and the baited/cascading Bloom filter, keeping the k-mer length identical to that used to build the filter (e.g. k=25). Scan transcriptomic k-mers sequentially against the filter: a k-mer that would overlap two different exons but is absent from the genomic k-mer set marks where one exon ends and the next starts (a potential IEB). Merge consecutive positive k-mers overlapping by k-1 bases into a draft exome, then filter out exons shorter than k+q (q default 5) to eliminate Bloom-filter false positives and disconnected k-length exons, and apply a relaxed merging step for exons overlapping by more than 10 bases. Optionally set --polish to inspect each pair of exons with a long-enough overlap for donor/acceptor sites (usually GU/AG) and trim accordingly. Output is a GFA1 file: S lines for exon segments with sequence and coordinates, L lines for exon connections, and P lines encoding transcripts as paths of exons.

## Related tools

- **EXFI** (Provides the build_splice_graph script that performs k-mer overlap detection, IEB marking, exon merging, filtering, polishing, and GFA1 output) — https://github.com/jlanga/exfi
- **abyss-bloom** (Builds the cascading Bloom filter of genomic k-mers that transcriptomic k-mers are scanned against) — https://github.com/bcgsc/abyss/
- **BioBloom Tools** (biobloomcategorizer filters WGS reads that do not share at least one k-mer with the transcriptome before the filter is built) — https://github.com/bcgsc/biobloom
- **Python** (Implementation language of EXFI's core programs, on top of Pandas and BioPython) — https://github.com/jlanga/exfi
- **Pandas** (Data-processing dependency used underneath EXFI's core programs)
- **BioPython** (Bioinformatics dependency used underneath EXFI's core programs)

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_splice_graph --help
```

## Evaluation signals

- The output test.gfa exists and is a valid GFA1 document containing H/S/L/C/P records
- S lines carry exon sequences with coordinates, L lines encode exon connections, and P lines encode transcript paths consistent with the input transcriptome
- The graph is inspectable in Bandage (a listed GFA1 implementation)
- When a reference annotation exists, exon predictions can be scored via compare_to_gff3 against the GFF3 annotation
- No exon shorter than k+q (q default 5) survives the filtering step, and excessively overlapping exons (overlap > 10 bases by default) have been relaxed-merged

## Limitations

- Bloom filters produce no false negatives but can produce false positives at a tunable BF FPR; the k+q exon-length filter and read filtering (68%–75% BF FPR reduction) exist to counter this
- EXFI was less accurate predicting exons for Atlantic herring than comparators, though it achieved the highest mappability (99.5%); mappability covers only interior exon sections, not full exon sequences
- Mappability of all methods fell when the input transcriptome was de novo-assembled; special care must be taken regarding input transcriptome quality and tissue breadth

## Evidence

- [intro] EXFI marks potential intron–exon boundaries by comparing transcriptomic k-mers against k-mers sequenced in the WGS experiment, flagging an IEB wherever a section of a transcript is not found in the genomic k-mer set.: "This tool works by comparing transcriptomic k‐mers with those sequenced in a WGS experiment, marking potential IEBs wherever a section of a transcript is not found in it"
- [methods] exons of length less than k + q, q default 5, are filtered out: "exons of length less than k + q (q by default five) are filtered out"
- [methods] relaxed merging of exons overlapping by more than 10 bases: [evidence span withheld — anchor status: not_found]
- [methods] polish: GU/AG donor/acceptor trimming of overlapping exon pairs: "each pair of exons with a long‐enough overlap is inspected for the donor/acceptor sites (usually GU/AG; 2e) and correctly trimmed if possible"
- [abstract] Results are returned in GFA1 format encoding predicted exon sequences and their connections into transcripts: "The results are returned in GFA1 format, which encodes both the predicted exon sequences and how they are connected to form transcripts"
- [intro] Splice graph representation: exons are nodes, IEBs are edges, transcripts are paths, genes are connected components: "a mathematical representation of a transcriptome where exons are represented by nodes, IEBs as edges, transcripts as paths, and genes as the different connected components"
- [readme] Docker container exposes build_splice_graph command: [evidence span withheld — anchor status: not_found]
- [results] Read filtering reduces BF FPR while improving classification metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
