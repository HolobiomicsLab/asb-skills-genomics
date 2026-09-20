---
name: telomere-length-distribution-visualization
description: Use when per-read telomap telobait-mode results (sample.telomap.tsv)
  are available and the goal is to visualize or compare telomere length distributions
  across samples to detect shifts or short-telomere enrichment.
when_to_use_negative:
- Data are vanilla whole-genome sequencing long reads without telobait capture (use
  WGS mode instead of telobait mode)
- Target samples are ALT cells whose very long telomeres exceed PacBio HiFi DNA polymerase
  processivity, making HiFi-based length measurement unreliable
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: telomap
  role: Generates the per-read telomere length table (sample.telomap.tsv) and the
    analytic plots and QC plots directories that this skill inspects
  repo: https://github.com/cytham/telomap
- name: PacBio high fidelity (HiFi) sequencing platform
  role: Produces the long-read data from which nucleotide-resolution telomere lengths
    are measured and visualized
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:50a2aff90c7c1ad92e6b899c36f163c2bf88e05b6e96a1516f22c24cb0741441
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - PacBio high fidelity (HiFi) sequencing platform
  - telomap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# telomere-length-distribution-visualization

## Summary

Generate and inspect per-sample telomere length distribution plots from telomap output so that nucleotide-resolution telomere length profiles (e.g., over-representation of extremely short <1000 bp telomeres in T24 cells) can be visually compared across samples.

## When to use

Use when per-read telomap telobait-mode results (sample.telomap.tsv) are available and the goal is to visualize or compare telomere length distributions across samples to detect shifts or short-telomere enrichment.

## When NOT to use

- Data are vanilla whole-genome sequencing long reads without telobait capture (use WGS mode instead of telobait mode)
- Target samples are ALT cells whose very long telomeres exceed PacBio HiFi DNA polymerase processivity, making HiFi-based length measurement unreliable

## Inputs

- sample.telomap.tsv per-read table (columns RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_END_MOTIF, TRF_COUNT, CHROM, TELOMERE_START, TELOMERE_END)
- telobait-captured PacBio HiFi long reads (fasta/fastq/bam) per sample
- capture_oligo.fa and barcodes.fa files bundled with telomap

## Outputs

- plots directory containing analytic plots of telomere length distribution per sample
- plots_QC directory containing QC plots
- comparison of distribution shapes across samples highlighting extremely short (<1000 bp) telomere fractions

## How to apply

After running telomap in telobait mode on the demultiplexed telobait-captured PacBio HiFi reads (with -c capture_oligo.fa and -b barcodes.fa), let telomap generate its analytic plots directory, which summarizes the telomere length distribution per sample. The distributions are built from the per-read TELOMERE_LEN column (motif count × 6 bp, excluding non-telomeric variant sequences within RAW_TELOMERE_LEN) in sample.telomap.tsv. Inspect the plots to detect over-representation of extremely short telomeres (<1000 bp) relative to other samples, as reported for T24 cells versus HCT116, 293T, IMR90 (PD47), and WI38 (PD40). Also review the companion plots_QC directory to confirm read-level metrics are sound before trusting distribution differences.

## Related tools

- **telomap** (Generates the per-read telomere length table (sample.telomap.tsv) and the analytic plots and QC plots directories that this skill inspects) — https://github.com/cytham/telomap
- **PacBio high fidelity (HiFi) sequencing platform** (Produces the long-read data from which nucleotide-resolution telomere lengths are measured and visualized)

## Examples

```
telomap telobait /path/to/T24_reads.fastq.gz /path/to/T24_work_dir -t 24 -c capture_oligo.fa -b barcodes.fa -n T24
```

## Evaluation signals

- sample.telomap.tsv exists with all 13 declared columns populated per read (RNAME through TELOMERE_END)
- plots directory contains analytic plots of telomere length distribution for each sample
- TELOMERE_LEN values equal motif count × 6 bp and are ≤ RAW_TELOMERE_LEN for every read
- T24 sample plot shows over-representation of telomeres <1000 bp relative to HCT116, 293T, IMR90 (PD47), and WI38 (PD40), reproducing the reported finding
- plots_QC directory is present and passes inspection before distribution interpretation

## Limitations

- Interpretation depends on upstream read detection using the TTAGGG motif and TELOMERE_START defined by at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3'); variant-rich reads may be demarcated differently
- Trimming genomic DNA with restriction enzymes that cut near telomeres (e.g., HphI, MnlI) can cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length, which would bias distributions
- ALT cells cannot be assessed because their very long telomeres exceed PacBio HiFi read lengths

## Evidence

- [other] T24 sample shows over-representation of extremely short (<1000 bp) telomeres: "our method has identified the over-representation of extremely short telomeres in T24 cells"
- [methods] telomap generates analytic plots: "Generate QC and analytic plots"
- [readme] plots directory contents: "plots | Directory containing analytic plots"
- [readme] TELOMERE_LEN definition: "Length of telomeric region in bases after excluding all non-telomeric sequences"
- [readme] telobait mode invocation with capture oligo and barcodes: "telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa"
