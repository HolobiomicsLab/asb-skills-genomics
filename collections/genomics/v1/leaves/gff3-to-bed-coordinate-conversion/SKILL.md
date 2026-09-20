---
name: gff3-to-bed-coordinate-conversion
description: Use when benchmarking predicted exons against an annotated reference
  (e.g. Ensembl release 91 GFF3 for GRCz10/GRCh38) and the predicted coordinates are
  transcript-based, so reference exons must be transformed to the same BED coordinate
  frame before comparison.
when_to_use_negative:
- The reference and predicted exon coordinates are already in the same (transcript-based)
  frame — re-converting would double-transform minus-strand coordinates
- The goal is genomic-coordinate analysis (e.g. variant calling or SNP genotyping
  against the genome), where chromosome-based GFF3 coordinates should be preserved
- Input annotation is not exon-level GFF3 (e.g. protein domains, CDS-only files without
  exon features)
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: bedtools
  role: intersect reference and predicted exon BED files with ≥95% mutual overlap
    to derive TP/FP/FN counts
- name: EXFI (compare_to_gff3)
  role: computes precision, recall, and F1 from the reference-vs-predicted exon comparison
  repo: https://github.com/jlanga/exfi
- name: EXFI (gfa1_to_fasta)
  role: extracts predicted exon sequences/coordinates from the GFA1 splice graph before
    comparison
  repo: https://github.com/jlanga/exfi
- name: Pandas
  role: tabulates per-condition metrics from the comparison into CSV benchmark tables
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_003
  - task_005
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:84d8d27519a745f80717edfa4240d0843d16e148d48ba84fb97f3a7f6a549364
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - ABySS
  - BWA MEM
  - BioPython
  - ChopStitch
  - EXFI
  - EXFI (compare_to_gff3)
  - EXFI (gfa1_to_fasta)
  - GMAP
  - Pandas
  - Python
  - SAMTools
  - Seqtk
  - bedtools
  - biobloomtools
  - wgsim
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# GFF3-to-BED exon coordinate conversion and benchmark comparison

## Summary

Converts chromosome-based GFF3 reference exon coordinates into transcript-based BED intervals (accounting for strand and exon order) so that EXFI-predicted exons can be benchmarked against the reference using bedtools intersect (mutual overlap ≥95%) and the compare_to_gff3 precision/recall/F1 script.

## When to use

Use when benchmarking predicted exons against an annotated reference (e.g. Ensembl release 91 GFF3 for GRCz10/GRCh38) and the predicted coordinates are transcript-based, so reference exons must be transformed to the same BED coordinate frame before comparison.

## When NOT to use

- The reference and predicted exon coordinates are already in the same (transcript-based) frame — re-converting would double-transform minus-strand coordinates
- The goal is genomic-coordinate analysis (e.g. variant calling or SNP genotyping against the genome), where chromosome-based GFF3 coordinates should be preserved
- Input annotation is not exon-level GFF3 (e.g. protein domains, CDS-only files without exon features)

## Inputs

- Reference GFF3 annotation (e.g. Ensembl release 91 GRCz10/GRCh38, or NCBI assemblies GCA_000233375.4, GCA_000966335.1)
- Reference genome/transcriptome FASTA for strand and exon-order bookkeeping
- Predicted exon coordinates in transcript-based BED (from EXFI build_splice_graph / gfa1_to_fasta GFA1 output)

## Outputs

- Transcript-based BED file of reference exons (strand- and exon-order-aware)
- bedtools intersect overlap table (mutual overlap ≥95%)
- Classification metrics per condition: precision, recall, F1 from compare_to_gff3 (e.g. coverage_metrics.csv, exfi_memory_benchmark_metrics.csv)

## How to apply

Read the reference GFF3 annotation and transform each exon's chromosome-based coordinates to transcript-based BED format, taking into account the transcript strand (reverse-complement ordering for minus-strand genes) and exon order within the transcript. Keep the resulting BED alongside the predicted exon BED produced by EXFI's build_splice_graph (extracted via gfa1_to_fasta from the GFA1 splice graph). Run bedtools intersect requiring a mutual overlap of at least 95% of the coordinates between reference and predicted BED files, and feed the intersection counts into the EXFI package's compare_to_gff3 script to compute precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1 (2PR/(P+R)). The same conversion scheme is applied consistently across all benchmarks in the paper (k-mer sweep, memory scaling, coverage titration, and cross-species comparison), so per-condition metric tables are directly comparable.

## Related tools

- **bedtools** (intersect reference and predicted exon BED files with ≥95% mutual overlap to derive TP/FP/FN counts)
- **EXFI (compare_to_gff3)** (computes precision, recall, and F1 from the reference-vs-predicted exon comparison) — https://github.com/jlanga/exfi
- **EXFI (gfa1_to_fasta)** (extracts predicted exon sequences/coordinates from the GFA1 splice graph before comparison) — https://github.com/jlanga/exfi
- **Pandas** (tabulates per-condition metrics from the comparison into CSV benchmark tables)

## Examples

```
bedtools intersect -a reference_transcript_bed.bed -b exfi_predicted_exons.bed -f 0.95 -r
```

## Evaluation signals

- Every exon feature of the input GFF3 appears exactly once in the converted BED — spot-check that minus-strand transcripts have exons ordered/offset with strand taken into account
- bedtools intersect run with a ≥95% reciprocal-overlap threshold yields TP/FP counts consistent with the reported metrics (e.g. zebrafish low-memory precision 95.8% / recall 93.8% at 4 GB)
- compare_to_gff3 F1 values fall in the reported ranges (e.g. F1 94.8% in the filtered low-memory run; near-constant F1 over k = 23–35)
- Re-running the conversion on the same GFF3 is deterministic and produces byte-identical BED output across benchmark conditions

## Limitations

- Conversion must correctly account for strand and exon order; an ordering error on minus-strand transcripts silently corrupts the ≥95% overlap comparison
- The comparison is only as good as the predicted exons: in the Atlantic herring case EXFI obtained worse F1 than GMAP despite the highest perfect-mapping rate (99.5%), and mappability covers only interior exon sections, not full exon sequences
- Metric values depend on the fixed 95% mutual-overlap cutoff; changing the threshold changes precision/recall and breaks comparability with the paper's reported numbers

## Evidence

- [methods] Transform the reference exon coordinates to transcript-based BED format, accounting for strand and exon order.: [evidence span withheld — anchor status: not_found]
- [methods] Compare reference versus predicted coordinates with bedtools intersect requiring a mutual overlap of at least 95%: [evidence span withheld — anchor status: not_found]
- [readme] Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"
- [methods] Genomes, transcriptomes, and GFF3 annotations of D. rerio and H. sapiens were downloaded from Ensembl (release 91: "Genomes, transcriptomes, and GFF3 annotations of D. rerio and H. sapiens were downloaded from Ensembl (release 91"
