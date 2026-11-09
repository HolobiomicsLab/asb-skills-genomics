---
name: metabarcoding-taxon-read-counting
description: Use when mock-community or bulk-sample amplicon FASTQ reads have been
  demultiplexed and you need per-taxon read counts, taxon-recovery percentages, or
  fold-change bias statistics (e.g. Levene's test) to compare sequencing platforms'
  quantitative fidelity.
when_to_use_negative:
- Input reads are raw, undemultiplexed Nanopore output — run MiniBar demultiplexing
  (dual-index barcode + primer identification) before counting taxa
- Input is already a per-taxon count or feature table — skip straight to fold-change
  computation and Levene's test
- Assessing Nanopore consensus-sequence error profiles or phylogenetic resolution
  rather than community composition and quantitative recovery bias
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed MinION reads using dual barcodes and primers, bins
    reads into per-sample files (-F), and trims barcodes/primers (-T) to yield clean
    amplicon sequences for taxon counting
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python/C++ edit-distance library minibar.py imports to compute approximate
    barcode and primer matches during demultiplexing
  repo: https://github.com/Martinsos/edlib
- name: MinION
  role: Oxford Nanopore platform that sequenced the long rDNA amplicons being quantified
    for taxon recovery
- name: Illumina
  role: Short-read MiSeq platform providing the optimized 18S metabarcoding comparison
    data with accurate abundance predictions
- name: Python 2.7
  role: Language in which minibar.py (and the downstream counting/bias analysis) is
    written; also runs in Python 3
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:74d1c6111984c5e073db409eb8771d0d037a0c2784113873981571eef1ea416e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - Illumina
  - MinION
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabarcoding-taxon-read-counting

## Summary

Counts recovered reads per taxon from demultiplexed metabarcoding FASTQ files (Nanopore long-amplicon rDNA vs Illumina 18S), computes per-taxon fold change between known input DNA amount and recovered reads, and quantifies qualitative recovery (percentage of input taxa recovered) to expose platform-specific abundance bias in mock arthropod communities.

## When to use

Use when mock-community or bulk-sample amplicon FASTQ reads have been demultiplexed and you need per-taxon read counts, taxon-recovery percentages, or fold-change bias statistics (e.g. Levene's test) to compare sequencing platforms' quantitative fidelity.

## When NOT to use

- Input reads are raw, undemultiplexed Nanopore output — run MiniBar demultiplexing (dual-index barcode + primer identification) before counting taxa
- Input is already a per-taxon count or feature table — skip straight to fold-change computation and Levene's test
- Assessing Nanopore consensus-sequence error profiles or phylogenetic resolution rather than community composition and quantitative recovery bias

## Inputs

- Demultiplexed, barcode/primer-trimmed Nanopore amplicon FASTQ files (one per mock community, from minibar.py -F -T)
- Illumina MiSeq 18S amplicon FASTQ files for the same mock communities
- Reference taxonomy file mapping amplicon sequences to taxa
- Table of known input DNA amounts per taxon per mock community

## Outputs

- Per-sample, per-taxon read-count table for each platform
- Per-taxon fold-change values (known input DNA amount vs recovered reads) per mock community and platform
- Qualitative recovery percentages (fraction of input taxa recovered per community)
- Levene's-test statistic comparing fold-change variance between Nanopore and Illumina
- mock_community_bias_results.csv containing fold changes, recovery percentages, and Levene's-test statistic

## How to apply

After MiniBar demultiplexing with default Method 3 (barcode edit distance 4, primer edit distance 11, search length 80), bin reads into per-sample files with -F and trim barcodes/primers with -T so only the amplicon remains. Match each trimmed amplicon sequence to the reference taxonomy and count recovered reads per taxon for every mock community, repeating the same counting for the Illumina MiSeq fastq files. For each taxon in each mock community and platform, compute the fold change between the known input DNA amount and the recovered read count. Summarize qualitative recovery as the percentage of input taxa recovered per community (reported: 83.33% average for Nanopore long-amplicon vs 100% for Illumina, which recovered all nine taxa from all four mock communities), and run Levene's test on the fold-change distributions to compare variance of quantitative recovery between platforms. Write per-taxon fold changes, recovery percentages, and the Levene's-test statistic to mock_community_bias_results.csv.

## Related tools

- **MiniBar** (Demultiplexes dual-indexed MinION reads using dual barcodes and primers, bins reads into per-sample files (-F), and trims barcodes/primers (-T) to yield clean amplicon sequences for taxon counting) — https://github.com/calacademy-research/minibar
- **Edlib** (Python/C++ edit-distance library minibar.py imports to compute approximate barcode and primer matches during demultiplexing) — https://github.com/Martinsos/edlib
- **MinION** (Oxford Nanopore platform that sequenced the long rDNA amplicons being quantified for taxon recovery)
- **Illumina** (Short-read MiSeq platform providing the optimized 18S metabarcoding comparison data with accurate abundance predictions)
- **Python 2.7** (Language in which minibar.py (and the downstream counting/bias analysis) is written; also runs in Python 3)

## Examples

```
./minibar.py -T -F Demultiplex.txt my_sequences.fastq
```

## Evaluation signals

- Qualitative recovery reproduces the reported contrast: Nanopore long-amplicon recovers ~83.33% of input taxa per mock community on average, while Illumina 18S recovers all nine taxa from all four mock communities (100%)
- Levene's-test outcome on the fold-change distributions is consistent with the reported higher quantitative bias (greater fold-change variance) for Nanopore vs Illumina
- Every taxon read count is traceable to a demultiplexed, primer/barcode-trimmed amplicon file (minibar -F -T output), with minibar summary line confirming IDs assigned (e.g. HH/Hh hit classes)
- mock_community_bias_results.csv is populated with per-taxon fold changes, recovery percentages, and the Levene's-test statistic for both platforms
- Illumina-side counts yield accurate abundance predictions relative to known input DNA amounts, matching the paper's reported quantitative agreement

## Limitations

- Long rDNA amplicon Nanopore metabarcoding shows highly skewed diversity estimates and very biased qualitative and quantitative taxon recovery of mock communities
- The source of the observed taxonomic bias in bulk samples is undetermined; the paper states further research is required before long rDNA amplicon sequencing can be used as a reliable resource for bulk sample analysis
- Barcode/primer matching tolerances are bounded by minibar defaults (edit distance 4 for barcodes, 11 for 27-base primers, search length 80); reads failing these thresholds are not assigned to samples and are silently excluded from counts

## Evidence

- [results] on average only 83.33% of taxa were recovered per Nanopore sequenced mock community, whereas the Illumina 18S protocol recovered all nine taxa from all four mock communities with accurate abundance predictions: [evidence span withheld — anchor status: not_found]
- [other] Compute the per-taxon fold change between the known input DNA amount and the recovered read counts for each taxon in each mock community and platform: [evidence span withheld — anchor status: not_found]
- [readme] Typically you'll want to group your records by identified sample. This can be done by using the -F option: "Typically you'll want to group your records by identified sample. This can be done by using the -F option"
- [readme] For downstream processing, you'll probably want to trim off the barcode and primer part of the sequence. This is done for you if you supply the -T trim option: "For downstream processing, you'll probably want to trim off the barcode and primer part of the sequence. This is done for you if you supply the -T trim option"
- [discussion] Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples: "Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples"
