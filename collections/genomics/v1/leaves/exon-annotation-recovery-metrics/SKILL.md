---
name: exon-annotation-recovery-metrics
description: Use when a de novo exon or splice-graph prediction (EXFI, ChopStitch,
  GMAP) must be quantitatively scored against a reference GFF3 annotation and assessed
  for downstream usability via mappability.
when_to_use_negative:
- No reference genome or GFF3 annotation exists for the target species — coordinate-based
  precision/recall and genomic mappability cannot be computed at all
- Predictions are still raw GFA1 splice graph nodes and have not been extracted into
  exon coordinate sets comparable to BED/GFF3 intervals
- The question is SNP/indel genotyping suitability (quality and distance filtering
  of variants) rather than exon recovery benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bedtools
  role: intersect subcommand for ≥95% mutual-overlap coordinate comparison between
    reference and predicted exons
- name: EXFI
  role: source of predicted exons (build_baited_bloom_filter k=25, then build_splice_graph);
    also provides compare_to_gff3 for precision/recall/F1
  repo: github.com/jlanga/exfi
- name: ChopStitch
  role: comparator k-mer-based exon predictor run with its default 50 bp k-mer length
- name: GMAP
  role: comparator splice-aware aligner (version 2018.07.04, default parameters) producing
    predicted exon coordinates
- name: BWA MEM
  role: aligns predicted exons back to each genomic reference for mappability computation
- name: SAMTools
  role: stores and processes BWA MEM alignment results in BAM for CIGAR-based perfect-mapping
    statistics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:11633ffc69909ae2b24612838a243aa9a6e9d02e01cbb35fc11e22016a377dd3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA MEM
  - ChopStitch
  - EXFI
  - GMAP
  - SAMTools
  - bedtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# exon-annotation-recovery-metrics

## Summary

Benchmarks predicted exon sets (from EXFI, ChopStitch, or GMAP) against reference GFF3 annotations by computing precision, recall, and F1 from coordinate overlaps, and mappability from BWA MEM alignments of predicted exons back to the genomic reference.

## When to use

Use when a de novo exon or splice-graph prediction (EXFI, ChopStitch, GMAP) must be quantitatively scored against a reference GFF3 annotation and assessed for downstream usability via mappability.

## When NOT to use

- No reference genome or GFF3 annotation exists for the target species — coordinate-based precision/recall and genomic mappability cannot be computed at all
- Predictions are still raw GFA1 splice graph nodes and have not been extracted into exon coordinate sets comparable to BED/GFF3 intervals
- The question is SNP/indel genotyping suitability (quality and distance filtering of variants) rather than exon recovery benchmarking

## Inputs

- Predicted exon coordinates (BED) from EXFI build_splice_graph, ChopStitch v1.0.0 (default 50 bp k-mer), or GMAP 2018.07.04 with default parameters
- Reference GFF3 exon annotations (e.g. Ensembl release 91 for D. rerio GRCz10 and H. sapiens GRCh38; NCBI assemblies GCA_000233375.4 for S. salar and GCA_000966335.1 for C. harengus)
- Assembled genomic reference FASTA per species for BWA MEM alignment

## Outputs

- Transcript-based BED file of reference exons (strand- and order-aware conversion of GFF3 coordinates)
- Precision, recall, and F1 per species and per method from compare_to_gff3
- BWA MEM BAM of predicted exons aligned to the genomic reference, processed with SAMTools
- Mappability metrics: fraction of mapped exons and fraction with perfect CIGAR strings
- Benchmark comparison table across species (zebrafish, human, Atlantic salmon, Atlantic herring)

## How to apply

Transform the reference GFF3 exon coordinates into transcript-based BED format, accounting for strand and exon order as described in the methods. Compare reference and predicted coordinates with the BEDTools intersect subcommand requiring a mutual overlap of at least 95% of the coordinates, then compute precision, recall, and F1 via the compare_to_gff3 script. Independently, align predicted exons to each genomic reference with BWA MEM, store results in BAM with SAMTools, and compute mappability as the fraction of mapped exons and the fraction with a perfect CIGAR string (all matches or small indels, no base clipping). Assemble precision, recall, F1, and mappability into a single benchmark table per species so that coordinate-level accuracy and downstream usability can be judged separately.

## Related tools

- **bedtools** (intersect subcommand for ≥95% mutual-overlap coordinate comparison between reference and predicted exons)
- **EXFI** (source of predicted exons (build_baited_bloom_filter k=25, then build_splice_graph); also provides compare_to_gff3 for precision/recall/F1) — github.com/jlanga/exfi
- **ChopStitch** (comparator k-mer-based exon predictor run with its default 50 bp k-mer length)
- **GMAP** (comparator splice-aware aligner (version 2018.07.04, default parameters) producing predicted exon coordinates)
- **BWA MEM** (aligns predicted exons back to each genomic reference for mappability computation)
- **SAMTools** (stores and processes BWA MEM alignment results in BAM for CIGAR-based perfect-mapping statistics)

## Examples

```
bedtools intersect -a predicted_exons.bed -b reference_exons.bed -f 0.95 -r > recovered.bed; compare_to_gff3 predicted_exons.bed reference.gff3
```

## Evaluation signals

- BEDTools intersect must enforce mutual overlap ≥95% of coordinates for a predicted exon to count as recovered
- F1 values must be consistent with the precision and recall reported by compare_to_gff3
- Mappability is reported as two fractions: mapped exons and exons with perfect CIGAR (all matches or small indels, no base clipping)
- EXFI should attain the highest mappability across all datasets (~99.5% on Atlantic herring), even where its exon precision is lower than GMAP
- Reference BED conversion must preserve strand and exon order per transcript; spot-check a few GFF3-to-BED conversions before intersecting

## Limitations

- Mappability is more optimistic than exon precision: it covers only interior exon sections, not full exon sequences, so imprecise predictions can still look highly mappable
- With a de novo-assembled input transcriptome (e.g. Atlantic herring from a single tissue, where lowly expressed transcripts are fragmented), mappability fell for all methods, so scores depend strongly on transcriptome quality and tissue breadth
- Full WGS read accession numbers and some comparison details (Table 3, Figures 3 and 6) are deferred to supplementary material, so benchmark reproduction may require SI access

## Evidence

- [methods] only 4.24% and 5.68% of the bases are exons: "In the human and zebrafish genomes, only 4.24% and 5.68% of the bases are exons, respectively (Table 1)"
- [discussion] EXFI less accurate on herring but highest mappability: "In that case, EXFI was less accurate predicting exons, but in terms of mappability, it achieved the highest results across all datasets (99.5%)."
- [discussion] mappability is optimistic and covers interior sections: "This high mappability result, although more optimistic than the exon precision, means that even if the exon prediction is not precise, it is still usable for downstream analysis."
- [intro] splice-aware comparator GMAP: "align either transcriptome or RNA‐Seq reads with a splice‐aware aligner such as GMAP (Wu & Watanabe, 2005), and extract from the results the IEB coordinates"
- [readme] BBT filter creation and categorization context: [evidence span withheld — anchor status: not_found]
