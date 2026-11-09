---
name: exon-merging-and-false-positive-filtering
description: Use when reconstructing exons from a de novo transcriptome against a
  WGS-derived Bloom filter (e.g. k=25 genome_k25_m100M_l1.bloom) and you must merge
  positive k-mers, remove Bloom-filter false-positive exons, and trim splice-site
  overlaps into a GFA1 graph.
when_to_use_negative:
- A reference genome and GFF3 annotation are already available — a splice-aware aligner
  such as GMAP can extract IEB coordinates directly from alignments
- There is no assembled transcriptome — EXFI requires a transcriptome assembly plus
  raw WGS reads, not raw RNA-Seq reads alone
- The goal is read-level sequence classification or host/contamination screening —
  use biobloomcategorizer directly instead of the exon reconstruction pipeline
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: EXFI
  role: build_splice_graph script performs IEB detection, k-mer merging, exon length
    filtering, relaxed merging, and optional polish, then writes GFA1
  repo: https://github.com/jlanga/exfi
- name: abyss-bloom
  role: builds the cascading Bloom filter of WGS genomic k-mers that transcriptomic
    k-mers are compared against
  repo: https://github.com/bcgsc/abyss/
- name: BioBloom Tools
  role: biobloommaker creates the Bloom filter and biobloomcategorizer discards WGS
    reads sharing no k-mer with the transcriptome, reducing the BF FPR before exon
    calling
  repo: https://github.com/bcgsc/biobloom.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:5434de98cff8869b62b301944bb93cad05ac476f61dd07cb4b2e2c56d69fd39e
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
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# exon-merging-and-false-positive-filtering

## Summary

EXFI's build_splice_graph post-detection logic: after transcriptomic k-mers are scored against a WGS Bloom filter, consecutive positive k-mers overlapping by k-1 bases are merged into a draft exome, short false-positive exons are removed, and overlapping exon pairs are relaxed-merged or GU/AG-polished before writing a GFA1 splice graph.

## When to use

Use when reconstructing exons from a de novo transcriptome against a WGS-derived Bloom filter (e.g. k=25 genome_k25_m100M_l1.bloom) and you must merge positive k-mers, remove Bloom-filter false-positive exons, and trim splice-site overlaps into a GFA1 graph.

## When NOT to use

- A reference genome and GFF3 annotation are already available — a splice-aware aligner such as GMAP can extract IEB coordinates directly from alignments
- There is no assembled transcriptome — EXFI requires a transcriptome assembly plus raw WGS reads, not raw RNA-Seq reads alone
- The goal is read-level sequence classification or host/contamination screening — use biobloomcategorizer directly instead of the exon reconstruction pipeline

## Inputs

- Assembled transcriptome FASTA (de novo transcriptome assembly)
- Baited/cascading Bloom filter built from WGS reads (e.g. genome_k25_m100M_l1.bloom from abyss-bloom, biobloommaker)
- k-mer length identical to the one used for the Bloom filter (k=25 in the paper's example)

## Outputs

- GFA1 splice graph file (e.g. test.gfa) with H/S/L/C/P records: exon segments with sequences and coordinates (S), exon connections (L), and transcripts as paths of exons (P)

## How to apply

Run EXFI's build_splice_graph with the transcriptome FASTA and the baited/cascading Bloom filter, keeping the k-mer length identical to the one used to build the filter (e.g. k=25). Scan transcriptomic k-mers sequentially: k-mers that would overlap two different exons but are absent from the genomic k-mer set mark potential intron-exon boundaries (IEBs). Merge consecutive positive k-mers that overlap by k-1 bases into a draft exome, then remove exons shorter than k+q (q default 5) to eliminate Bloom-filter false-positive nucleotides and disconnected k-length exons. Apply the relaxed merging step where exons overlap by an excessive number of bases (10 by default). If --polish is set, inspect each exon pair with a long-enough overlap for donor/acceptor sites (usually GU/AG) and trim accordingly before emitting the GFA1 file with S (exon sequences and coordinates), L (connections), and P (transcript paths) lines.

## Related tools

- **EXFI** (build_splice_graph script performs IEB detection, k-mer merging, exon length filtering, relaxed merging, and optional polish, then writes GFA1) — https://github.com/jlanga/exfi
- **abyss-bloom** (builds the cascading Bloom filter of WGS genomic k-mers that transcriptomic k-mers are compared against) — https://github.com/bcgsc/abyss/
- **BioBloom Tools** (biobloommaker creates the Bloom filter and biobloomcategorizer discards WGS reads sharing no k-mer with the transcriptome, reducing the BF FPR before exon calling) — https://github.com/bcgsc/biobloom.git

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_splice_graph --help
```

## Evaluation signals

- Output test.gfa exists and is a valid GFA1 document containing H/S/L/C/P records
- No exon in the S lines is shorter than k+q (q=5) — the length filter should have removed disconnected k-length and false-positive exons
- Un-polished exon pairs overlap by at most the relaxed-merge threshold (10 bases by default)
- P-line transcript paths are consistent with the input transcriptome, and the graph is inspectable in Bandage
- When a reference GFF3 exists, predicted exons score sensibly via compare_to_gff3 (exon precision/recall and mappability metrics)

## Limitations

- Bloom filters return no false negatives but produce false positives with a tunable BF FPR, so short spurious exons can appear; the k+q length filter mitigates this
- Read filtering reduced the BF FPR by 68%-75%, so skipping the biobloomcategorizer pre-filter degrades exon prediction quality
- Exon prediction accuracy depends on transcriptome quality: in the herring case (single-tissue transcriptome, fragmented lowly expressed transcripts) EXFI was less accurate than comparators, although mappability stayed high (99.5%) and covers only interior exon sections, not full exon sequences

## Evidence

- [abstract] The main algorithm uses Bloom filters to remove reads that are not part of the transcriptome: "The main algorithm uses Bloom filters to remove reads that are not part of the transcriptome"
- [methods] exons of length less than k + q (q by default five) are filtered out (Figure 2c): "exons of length less than k + q (q by default five) are filtered out (Figure 2c)"
- [methods] a more relaxed merging step is applied when exons overlap by an excessive number of bases (10 by default; Figure 2d): "a more relaxed merging step is applied when exons overlap by an excessive number of bases (10 by default; Figure 2d)"
- [methods] each pair of exons with a long‐enough overlap is inspected for the donor/acceptor sites (usually GU/AG; 2e) and correctly trimmed if possible: "each pair of exons with a long‐enough overlap is inspected for the donor/acceptor sites (usually GU/AG; 2e) and correctly trimmed if possible"
- [readme] Build a Docker container with all the tools to execute exfi: [evidence span withheld — anchor status: not_found]
