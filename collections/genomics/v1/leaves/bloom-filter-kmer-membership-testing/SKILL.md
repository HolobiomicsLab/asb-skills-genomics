---
name: bloom-filter-kmer-membership-testing
description: Use when an assembled transcriptome and raw WGS reads are available without
  a reference genome and exon boundaries or splice-graph nodes must be inferred by
  k-mer membership testing.
when_to_use_negative:
- A reliable reference genome and annotation exist — a splice-aware aligner such as
  GMAP can extract IEB coordinates directly from alignments
- The input transcriptome is low-quality or from a single tissue, so lowly expressed
  transcripts are fragmented and absent — IEB calls will be unreliable
- The goal requires actual mapping locations rather than set membership — Bloom filters
  categorize sequences but do not provide mapping coordinates
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: EXFI
  role: Pipeline whose build_splice_graph script performs the k-mer membership scan
    and IEB marking
  repo: https://github.com/jlanga/exfi
- name: abyss-bloom
  role: Builds the cascading Bloom filter of genomic k-ers from remaining WGS reads
  repo: https://github.com/bcgsc/abyss/
- name: biobloomcategorizer
  role: Discards each WGS read that does not share at least one k-mer with the transcriptome,
    reducing BF FPR
  repo: https://github.com/bcgsc/biobloom
- name: Pandas
  role: Data processing layer under EXFI's core Python programs
- name: BioPython
  role: Bioinformatics layer for parsing/sequences under EXFI
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:4eba9cb4bff9494895f7ffaf495fa639b3569ecb17e690ffc4a450b616bbf41f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BioPython
  - EXFI
  - Pandas
  - Python
  - abyss-bloom
  - biobloomcategorizer
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# bloom-filter-kmer-membership-testing

## Summary

Tests transcriptomic k-mer membership against a baited/cascading Bloom filter built from WGS reads to mark potential intron–exon boundaries (IEBs): k-mers absent from the genomic k-mer set flag where one exon ends and the next begins, exploiting the no-false-negative property of Bloom filters.

## When to use

Use when an assembled transcriptome and raw WGS reads are available without a reference genome and exon boundaries or splice-graph nodes must be inferred by k-mer membership testing.

## When NOT to use

- A reliable reference genome and annotation exist — a splice-aware aligner such as GMAP can extract IEB coordinates directly from alignments
- The input transcriptome is low-quality or from a single tissue, so lowly expressed transcripts are fragmented and absent — IEB calls will be unreliable
- The goal requires actual mapping locations rather than set membership — Bloom filters categorize sequences but do not provide mapping coordinates

## Inputs

- Assembled transcriptome FASTA (de novo or reference-guided)
- Raw WGS reads (FASTQ/FASTA, e.g. baited/cascading Bloom filter input)
- Bloom filter built with abyss-bloom/biobloom at a fixed k-mer length (e.g. k=25)
- k-mer length parameter identical to the one used to build the filter

## Outputs

- Set of marked potential intron–exon boundaries (IEBs)
- Merged exon sequences with coordinates
- GFA1 splice graph (H/S/L/C/P records: exon segments, connections, transcript paths)

## How to apply

Build a baited Bloom filter (e.g. genome_k25_m100M_l1.bloom with abyss-bloom/biobloom, k-mer length 25 and a tunable BF FPR), first discarding WGS reads that share no k-mer with the transcriptome via biobloomcategorizer — read filtering cut the BF FPR by 68–75% and improved all classification metrics. Then scan transcriptomic k-mers sequentially against the filter: a k-mer absent from the genomic set marks a potential IEB, since Bloom filters return no false negatives, only false positives. Merge consecutive positive k-mers overlapping by k-1 bases into draft exons, drop exons shorter than k+q (q default 5) to remove Bloom-filter false-positive nucleotides, apply relaxed merging for exons overlapping by more than 10 bases, and optionally --polish by inspecting GU/AG donor/acceptor sites on overlapping exon pairs. Emit the result as a GFA1 splice graph with S/L/P lines.

## Related tools

- **EXFI** (Pipeline whose build_splice_graph script performs the k-mer membership scan and IEB marking) — https://github.com/jlanga/exfi
- **abyss-bloom** (Builds the cascading Bloom filter of genomic k-ers from remaining WGS reads) — https://github.com/bcgsc/abyss/
- **biobloomcategorizer** (Discards each WGS read that does not share at least one k-mer with the transcriptome, reducing BF FPR) — https://github.com/bcgsc/biobloom
- **Pandas** (Data processing layer under EXFI's core Python programs)
- **BioPython** (Bioinformatics layer for parsing/sequences under EXFI)

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_splice_graph --kmer-length 25 --transcriptome transcriptome.fa --bloom genome_k25_m100M_l1.bloom --output test.gfa
```

## Evaluation signals

- Output test.gfa exists and parses as valid GFA1 containing H/S/L/C/P records
- S-line exon coordinates and P-line transcript paths are consistent with the input transcriptome (inspectable in Bandage)
- Exon precision/coverage compare favourably against GFF3 annotations via compare_to_gff3 when a reference annotation exists
- No k-mer from the transcriptome that is present in the genomic set is ever called absent (no false negatives expected)
- BF FPR after read filtering should be 68–75% lower than without read filtering

## Limitations

- Bloom filters produce false positives at a tunable BF FPR; mitigation is the exon length filter (>= k+q) and read filtering
- Mappability/exon accuracy drops when the input transcriptome is de novo-assembled from a single tissue (Atlantic herring case: lowly expressed transcripts fragmented, other tissues missing)
- Membership testing gives no mapping locations, so results are usable for downstream analysis even when exon prediction is imprecise (up to 99.5% mappability)

## Evidence

- [intro] This tool works by comparing transcriptomic k-mers with those sequenced in a WGS experiment, marking potential IEBs wherever a section of a transcript is not found in it: "This tool works by comparing transcriptomic k‐mers with those sequenced in a WGS experiment, marking potential IEBs wherever a section of a transcript is not found in it"
- [methods] It does not return false negatives, but it can produce false positives with a tunable false positive rate (BF FPR): "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [results] Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [methods] exons of length less than k + q (q by default five) are filtered out: "exons of length less than k + q (q by default five) are filtered out (Figure 2c)"
- [readme] Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"
