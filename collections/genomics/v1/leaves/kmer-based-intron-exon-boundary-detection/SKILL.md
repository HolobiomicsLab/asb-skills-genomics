---
name: kmer-based-intron-exon-boundary-detection
description: Use when you have an assembled transcriptome (FASTA) and raw WGS reads
  or a prebuilt abyss-bloom filter (e.g. genome_k25_m100M_l1.bloom, k=25) and need
  exons, IEBs and a splice graph in GFA1 without a reference genome.
when_to_use_negative:
- A reference genome and annotation exist and a splice-aware aligner (e.g. GMAP) can
  align transcripts directly to extract IEB coordinates
- The input is raw RNA-Seq reads rather than an assembled transcriptome — EXFI consumes
  a transcriptome assembly plus raw WGS reads
- The WGS k-mer set is unavailable and cannot be built, since IEB detection relies
  entirely on transcript k-mers missing from the genomic k-mer set
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: EXFI
  role: 'build_splice_graph script: k-mer scan against Bloom filter, IEB marking,
    exon merging/polishing, GFA1 output'
  repo: https://github.com/jlanga/exfi
- name: biobloom (BioBloom Tools)
  role: biobloomcategorizer filters WGS reads that do not share at least one k-mer
    with the transcriptome before filter construction
  repo: https://github.com/bcgsc/biobloom
- name: abyss-bloom (ABySS)
  role: builds the cascading Bloom filter of WGS k-mers used for IEB detection
  repo: https://github.com/bcgsc/abyss/
- name: exfi-docker
  role: Docker container packaging EXFI and its dependencies for running build_splice_graph
  repo: https://github.com/jlanga/exfi-docker
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:3561b8445b194e613cd657b6c4a6c9b2260e99aee51d1419275928c1ba018f52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BioPython
  - EXFI
  - Pandas
  - Python
  - abyss-bloom (ABySS)
  - biobloom (BioBloom Tools)
  - exfi-docker
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# kmer-based-intron-exon-boundary-detection

## Summary

K-mer-based detection of potential intron–exon boundaries (IEBs) and exon construction from a de novo transcriptome assembly against a Bloom filter of whole-genome sequencing k-mers, as implemented in EXFI's build_splice_graph. Because Bloom filters return no false negatives, transcript k-mers absent from the genomic k-mer set flag where an exon ends and the next starts, without a reference genome.

## When to use

Use when you have an assembled transcriptome (FASTA) and raw WGS reads or a prebuilt abyss-bloom filter (e.g. genome_k25_m100M_l1.bloom, k=25) and need exons, IEBs and a splice graph in GFA1 without a reference genome.

## When NOT to use

- A reference genome and annotation exist and a splice-aware aligner (e.g. GMAP) can align transcripts directly to extract IEB coordinates
- The input is raw RNA-Seq reads rather than an assembled transcriptome — EXFI consumes a transcriptome assembly plus raw WGS reads
- The WGS k-mer set is unavailable and cannot be built, since IEB detection relies entirely on transcript k-mers missing from the genomic k-mer set

## Inputs

- Assembled transcriptome FASTA (de novo assembly; quality/tissue breadth matters)
- Baited/cascading Bloom filter from raw WGS reads (e.g. genome_k25_m100M_l1.bloom), built with the same k-mer length
- k-mer length matching the filter (k=25 in the paper)
- Optional: raw WGS FASTQ reads for build_baited_bloom_filter (filtering via biobloomcategorizer + ABySS cascading Bloom filter)

## Outputs

- GFA1 splice graph (H/S/L/C/P records): exon segments with sequences and coordinates (S), exon connections (L), transcripts as paths (P)
- Draft exome (merged exon sequences) after k+q length filtering and relaxed merging
- Optionally polished exon boundaries (GU/AG donor/acceptor trimming)

## How to apply

Load the transcriptome FASTA together with the baited/cascading Bloom filter built by build_baited_bloom_filter (biobloomcategorizer first discards WGS reads sharing no k-mer with the transcriptome; ABySS then builds the cascading filter) using the identical k-mer length used for the filter (k=25 in the paper). Scan transcriptomic k-mers sequentially against the filter: k-mers overlapping two different exons but absent from the genomic k-mer set mark potential IEBs. Merge consecutive positive k-mers overlapping by k-1 bases into draft exons, remove exons shorter than k+q (q=5) to eliminate Bloom-filter false positives and disconnected k-length exons, then apply a relaxed merge for exons overlapping by more than 10 bases; optionally run --polish to inspect overlapping exon pairs for GU/AG donor/acceptor sites and trim. Write the splice graph as GFA1: S lines with exon sequences and coordinates, L lines for exon connections, and P lines encoding transcripts as paths of exons.

## Related tools

- **EXFI** (build_splice_graph script: k-mer scan against Bloom filter, IEB marking, exon merging/polishing, GFA1 output) — https://github.com/jlanga/exfi
- **biobloom (BioBloom Tools)** (biobloomcategorizer filters WGS reads that do not share at least one k-mer with the transcriptome before filter construction) — https://github.com/bcgsc/biobloom
- **abyss-bloom (ABySS)** (builds the cascading Bloom filter of WGS k-mers used for IEB detection) — https://github.com/bcgsc/abyss/
- **exfi-docker** (Docker container packaging EXFI and its dependencies for running build_splice_graph) — https://github.com/jlanga/exfi-docker

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_splice_graph --help
```

## Evaluation signals

- Output test.gfa exists and parses as valid GFA1 containing H/S/L/C/P records
- S-line exon coordinates and P-line transcript paths are consistent with the input transcriptome
- All retained exons are at least k+q bases long (k=25, q=5) and merged exon pairs do not overlap by more than 10 bases
- With --polish, overlapping exon pairs exhibit GU/AG donor/acceptor motifs at trimmed boundaries
- Graph is inspectable in Bandage and, where a reference annotation exists, scoreable via compare_to_gff3

## Limitations

- Bloom filters produce tunable false positives (BF FPR) with no false negatives; filtering reads reduced BF FPR by 68%–75% while slightly improving all classification metrics
- Mappability fell for all methods when the input transcriptome was de novo-assembled (Atlantic herring); single-tissue transcriptomes miss other tissues' transcripts and fragment lowly expressed ones, so special care must be taken regarding input transcriptome quality
- Exon prediction was less accurate for Atlantic herring than comparators, although mappability was still the highest across datasets (99.5%)

## Evidence

- [intro] This tool works by comparing transcriptomic k‐mers with those sequenced in a WGS experiment, marking potential IEBs wherever a section of a transcript is not found in it: "This tool works by comparing transcriptomic k‐mers with those sequenced in a WGS experiment, marking potential IEBs wherever a section of a transcript is not found in it"
- [methods] The exon and splice graph prediction procedure is carried out by the build_splice_graph script: "The exon and splice graph prediction procedure is carried out by the build_splice_graph script"
- [methods] each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer: "each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer"
- [methods] exons of length less than k + q (q by default five) are filtered out: "exons of length less than k + q (q by default five) are filtered out"
- [results] Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [abstract] The results are returned in GFA1 format, which encodes both the predicted exon sequences and how they are connected to form transcripts: "The results are returned in GFA1 format, which encodes both the predicted exon sequences and how they are connected to form transcripts"
- [readme] Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"
- [discussion] Another lesson learned is that special care has to be taken regarding the input transcriptome.: "Another lesson learned is that special care has to be taken regarding the input transcriptome."
