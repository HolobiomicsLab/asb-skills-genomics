---
name: read-alignment-mappability-assessment
description: Use when you have predicted exon sequences (e.g., from EXFI, ChopStitch,
  or GMAP) and a reference genome, and need to quantify how usable those exons are
  for downstream SNP genotyping or read recruitment analysis.
when_to_use_negative:
- No reference genome is available — the exon predictions themselves are reference-free
  (EXFI), so mappability cannot be computed; use precision/recall against an annotation
  instead
- The comparison target is exon-boundary accuracy rather than downstream usability
  — high mappability is more optimistic than exon precision and does not indicate
  correct IEB prediction
- Input is raw WGS reads rather than predicted exon sequences — mappability here is
  defined for predicted exon sequences aligned to the reference, not raw reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: BWA MEM
  role: aligns predicted exon sequences to each genomic reference to test whether
    exons are mappable
- name: SAMTools
  role: stores the BWA MEM alignment results in BAM format for CIGAR inspection
- name: bedtools
  role: used upstream in the benchmark to compute exon precision/recall against reference
    GFF3 (95% mutual overlap intersect), complementing mappability
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:9f38006709f8177c1cd9985fb2e9448f58fc8fdc8872ae50e18c5645e7e56095
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# read-alignment-mappability-assessment

## Summary

Assesses the mappability of predicted exon sequences by aligning them to a genomic reference with BWA MEM and computing the fraction of mapped exons and the fraction with a perfect CIGAR string (all matches or small indels, no base clipping).

## When to use

Use when you have predicted exon sequences (e.g., from EXFI, ChopStitch, or GMAP) and a reference genome, and need to quantify how usable those exons are for downstream SNP genotyping or read recruitment analysis.

## When NOT to use

- No reference genome is available — the exon predictions themselves are reference-free (EXFI), so mappability cannot be computed; use precision/recall against an annotation instead
- The comparison target is exon-boundary accuracy rather than downstream usability — high mappability is more optimistic than exon precision and does not indicate correct IEB prediction
- Input is raw WGS reads rather than predicted exon sequences — mappability here is defined for predicted exon sequences aligned to the reference, not raw reads

## Inputs

- Predicted exon sequences (e.g., GFA1 splice graph output from EXFI's build_splice_graph)
- Reference genome FASTA (e.g., Ensembl release 91 GRCz10/GRCh38, or NCBI GCA_000233375.4 S. salar, GCA_000966335.1 C. harengus)

## Outputs

- BAM file of exon alignments (via SAMTools)
- Mappability metrics: fraction of mapped exons and fraction of exons with a perfect CIGAR string

## How to apply

Align the predicted exon sequences (e.g., EXFI's GFA1-encoded exon sequences) to the appropriate genomic reference using BWA MEM, and store results in BAM format with SAMTools. Compute two mappability metrics: (1) the fraction of predicted exons that map to the reference at all, and (2) the fraction of mapped exons with a perfect CIGAR string, defined as a CIGAR containing all matches or small indels and no base clipping. Judge correctness by checking that mappability is higher (more optimistic) than exon precision — in the EXFI benchmark this reached ~99.5% perfect mapping on Atlantic herring, the highest across all compared methods. Note that mappability covers only interior exon sections rather than full exon sequences, so it complements, but does not replace, precision/recall computed via BEDTools 95% mutual-overlap intersect against the reference GFF.

## Related tools

- **BWA MEM** (aligns predicted exon sequences to each genomic reference to test whether exons are mappable)
- **SAMTools** (stores the BWA MEM alignment results in BAM format for CIGAR inspection)
- **bedtools** (used upstream in the benchmark to compute exon precision/recall against reference GFF3 (95% mutual overlap intersect), complementing mappability)

## Examples

```
bwa mem reference.fa predicted_exons.fa | samtools sort -o exons_alignments.bam
```

## Evaluation signals

- Mappability should exceed exon precision — the article states the high mappability result is 'more optimistic than the exon precision'
- For EXFI predictions, expect ~99.5% perfect-mapping rate (highest across all datasets, including Atlantic herring)
- A 'perfect' alignment must have a CIGAR string of all matches or small indels with no base clipping; count only those exons toward the perfect-mapping fraction
- Mappability is expected to fall for all methods when the input transcriptome was de novo-assembled from a single tissue (the herring case), so lower values there are expected rather than a pipeline failure

## Limitations

- High mappability scores cover only interior exon sections, not full exon sequences, and are more optimistic than exon precision
- In the Atlantic herring case, EXFI was less accurate predicting exons than comparators (k-mer-based methods obtained worse-than-expected F1 scores than GMAP), even though mappability was highest (99.5%)
- Mappability of all methods fell when the input transcriptome was de novo-assembled from a single tissue, where lowly expressed transcripts appear fragmented

## Evidence

- [discussion] Mappability definition and computation: "In that case, EXFI was less accurate predicting exons, but in terms of mappability, it achieved the highest results across all datasets (99.5%)."
- [discussion] Mappability is optimistic relative to precision: "This high mappability result, although more optimistic than the exon precision, means that even if the exon prediction is not precise, it is still usable for downstream analysis."
- [discussion] De novo transcriptome input degrades mappability: "In the herring case, the mappability scores fell for all methods."
- [results] Expected EXFI mappability across datasets: [evidence span withheld — anchor status: not_found]
- [abstract] Prediction output format that feeds the alignment: "The results are returned in GFA1 format, which encodes both the predicted exon sequences and how they are connected to form transcripts"
