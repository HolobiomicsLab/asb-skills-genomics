---
name: classification-metrics-computation
description: Use when benchmarking exon or splice graph predictions against a reference
  GFF3 annotation (e.g. Ensembl release 91 for GRCz10/GRCh38) and you need precision/recall/F1
  plus BF FPR as classification metrics.
when_to_use_negative:
- No reference annotation exists (draft-genome or de novo transcriptome cases like
  sugar pine or axolotl), so precision/recall against a GFF3 ground truth cannot be
  computed — use mappability instead
- Evaluating mappability of predicted exons for downstream SNP genotyping rather than
  exon prediction accuracy — the article notes mappability scores are more optimistic
  than exon precision
- The input is already a metrics table or summary.tsv from biobloomcategorizer rather
  than exon predictions needing classification evaluation
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EXFI
  role: produces predicted exons and splice graphs (build_splice_graph) and supplies
    the compare_to_gff3 script used to compute classification metrics
  repo: https://github.com/jlanga/exfi
- name: bedtools
  role: intersect with mutual overlap of at least 95% between predicted and reference
    exons
- name: Pandas
  role: used to assemble BF FPR and classification metrics across runs into a combined
    table
- name: BioBloom Tools
  role: provides the BF FPR values and biobloomcategorizer read filtering that influence
    the classification metrics
  repo: https://github.com/bcgsc/biobloom
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:159d4877137db469e609677468d7596676a2873a82606d92effa3040f3dc0a49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BioBloom Tools
  - EXFI
  - Pandas
  - Python
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# classification-metrics-computation

## Summary

Computes exon-level precision, recall, and F1 for EXFI splice graph predictions by comparing predicted exons to a reference annotation via BEDTools intersect with a mutual overlap of at least 95%, alongside the Bloom filter FPR across memory settings.

## When to use

Use when benchmarking exon or splice graph predictions against a reference GFF3 annotation (e.g. Ensembl release 91 for GRCz10/GRCh38) and you need precision/recall/F1 plus BF FPR as classification metrics.

## When NOT to use

- No reference annotation exists (draft-genome or de novo transcriptome cases like sugar pine or axolotl), so precision/recall against a GFF3 ground truth cannot be computed — use mappability instead
- Evaluating mappability of predicted exons for downstream SNP genotyping rather than exon prediction accuracy — the article notes mappability scores are more optimistic than exon precision
- The input is already a metrics table or summary.tsv from biobloomcategorizer rather than exon predictions needing classification evaluation

## Inputs

- Predicted exon sequences/coordinates from EXFI build_splice_graph (GFA1 splice graph output)
- Reference GFF3 annotation (Ensembl release 91, GRCz10/GRCh38)
- Reference exon coordinates converted to transcript-based BED
- BF FPR values recorded from build_baited_bloom_filter per Bloom filter memory setting

## Outputs

- Exon precision, recall, and F1 per species and per Bloom filter memory setting
- BF FPR per run (e.g. 8.1% at 4 GB vs 0.4% at 60 GB in zebrafish)
- Combined metrics table exfi_memory_benchmark_metrics.csv

## How to apply

Convert the reference GFF3 exon coordinates (e.g. Ensembl release 91 annotations for D. rerio GRCz10 and H. sapiens GRCh38) to transcript-based BED, then compare predicted exons to the reference using BEDTools intersect with a mutual overlap of at least 95%, computing precision, recall, and F1 via EXFI's compare_to_gff3 script. Record the Bloom filter FPR for each run alongside the classification metrics (the article fixes k-mer length at 25 bp and varies Bloom filter memory, e.g. 4 GB vs 60 GB). Because exons are a tiny fraction of the genome (4.24% in human, 5.68% in zebrafish), precision/recall must be computed on exon calls, not raw bases. Aggregate all runs (2 species x 2 memory settings) into a single metrics table such as exfi_memory_benchmark_metrics.csv. Note that read filtering with biobloomcategorizer reduces BF FPR by 68%–75% while slightly improving classification metrics, so filtered/unfiltered runs should be tracked separately.

## Related tools

- **EXFI** (produces predicted exons and splice graphs (build_splice_graph) and supplies the compare_to_gff3 script used to compute classification metrics) — https://github.com/jlanga/exfi
- **bedtools** (intersect with mutual overlap of at least 95% between predicted and reference exons)
- **Pandas** (used to assemble BF FPR and classification metrics across runs into a combined table)
- **BioBloom Tools** (provides the BF FPR values and biobloomcategorizer read filtering that influence the classification metrics) — https://github.com/bcgsc/biobloom

## Examples

```
bedtools intersect -f 0.95 -r -a predicted_exons.bed -b reference_exons.bed
```

## Evaluation signals

- Reproduced zebrafish metrics fall near reported values: 95.8% precision / 93.8% recall at 4 GB and 96.6% precision / 94.6% recall at 60 GB
- BF FPR values fall in the 8.1%–0.4% range across the 4 GB to 60 GB memory settings
- BEDTools intersect uses mutual overlap of at least 95% (both -f and reverse overlap thresholds set to 0.95)
- Metrics table contains all 2 species x 2 memory settings = 4 runs with both BF FPR and precision/recall populated
- Precision and recall should improve only marginally as memory increases (small BF FPR effect on exon metrics despite 8.1% -> 0.4% FPR drop)

## Limitations

- Exons occupy only 4.24% (human) and 5.68% (zebrafish) of genome bases, so base-level accounting inflates apparent accuracy; exon-level precision/recall is stricter
- Mappability is more optimistic than exon precision because it covers only interior exon sections, not full exon sequences
- Metrics depend on transcriptome quality: in the single-tissue herring transcriptome, lowly expressed transcripts were fragmented and EXFI was less accurate than comparators, so metrics cannot be generalized across transcriptome sources

## Evidence

- [results] classification metrics improve marginally with memory: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [methods] exon fraction of genome: "In the human and zebrafish genomes, only 4.24% and 5.68% of the bases are exons, respectively (Table 1)"
- [discussion] mappability vs precision caveat: "This high mappability result, although more optimistic than the exon precision, means that even if the exon prediction is not precise, it is still usable for downstream analysis."
- [readme] toolchain used for exon extraction and comparison: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"
