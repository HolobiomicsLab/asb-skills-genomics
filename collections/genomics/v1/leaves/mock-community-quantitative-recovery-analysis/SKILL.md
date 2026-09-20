---
name: mock-community-quantitative-recovery-analysis
description: Use when fastq data from mock (bulk) communities with known taxon composition
  and known input DNA amounts exist, and you must compare platform-specific recovery
  bias, e.g. Nanopore MinION vs Illumina MiSeq.
when_to_use_negative:
- Input data are not from mock communities with known taxon composition and known
  input DNA amounts (fold change cannot be computed without ground truth).
- The goal is consensus calling or error profiling of Nanopore reads rather than cross-platform
  taxon recovery and abundance-bias comparison.
- A per-taxon read count table already exists and only phylogenetic placement of the
  long rDNA amplicons is needed.
edam_operation: http://edamontology.org/operation_3238
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3053
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed MinION reads by dual barcodes and primers, bins
    reads into per-sample files (-F) and trims barcodes/primers (-T) prior to taxon
    counting
  repo: https://github.com/calacademy-research/minibar
- name: MinION
  role: Oxford Nanopore platform producing the long rDNA amplicon reads whose mock-community
    recovery is quantified
- name: Illumina
  role: Short-read MiSeq 18S platform serving as the optimized comparator protocol
    for recovery accuracy
- name: Edlib
  role: Python alignment library used by MiniBar for edit-distance matching of barcodes
    and primers
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Language in which MiniBar runs (compatible with Python 3); used for the counting/fold-change/Levene's-test
    analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:d11030705d97c33e120a8eb4ca0740b7eed68d825465819e3ec579b3ce47bf7a
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
  edam_operation: http://edamontology.org/operation_3238
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mock-community-quantitative-recovery-analysis

## Summary

Quantifies qualitative (taxon richness) and quantitative (abundance/fold-change) recovery of known input taxa from mock communities sequenced on Nanopore long-amplicon rDNA versus Illumina short-read 18S, using MiniBar-demultiplexed FASTQ read counts and Levene's test on fold-change variance.

## When to use

Use when fastq data from mock (bulk) communities with known taxon composition and known input DNA amounts exist, and you must compare platform-specific recovery bias, e.g. Nanopore MinION vs Illumina MiSeq.

## When NOT to use

- Input data are not from mock communities with known taxon composition and known input DNA amounts (fold change cannot be computed without ground truth).
- The goal is consensus calling or error profiling of Nanopore reads rather than cross-platform taxon recovery and abundance-bias comparison.
- A per-taxon read count table already exists and only phylogenetic placement of the long rDNA amplicons is needed.

## Inputs

- Nanopore MinION mock-community fastq files (dual-indexed long rDNA amplicons)
- Illumina MiSeq 18S mock-community fastq files
- MiniBar dual-index barcode file mapping sample ID, forward/reverse indexes and primers
- Reference taxonomy / reference amplicon sequences for taxon assignment
- Known input DNA amounts per taxon per mock community

## Outputs

- mock_community_bias_results.csv with per-taxon fold changes, recovery percentages, and Levene's-test statistic
- Per-sample read count tables per taxon for each platform
- Qualitative recovery percentages (fraction of the nine input taxa recovered per mock community)

## How to apply

Demultiplex the Nanopore fastq with MiniBar using the dual-index barcode file with defaults: Method 3, barcode edit distance 4, primer edit distance 11, search length 80; bin reads per sample with -F and trim barcodes/primers with -T so only the amplicon is retained. Match the trimmed amplicon sequences to the reference taxonomy to count reads per taxon per mock community, doing the same for the Illumina MiSeq fastq files. Compute the per-taxon fold change between known input DNA amount and recovered read counts for each taxon on each platform, and compare fold-change distributions between platforms with Levene's test. Report qualitative recovery as the percentage of input taxa recovered (e.g. 83.33% for Nanopore vs 100% for Illumina in the source study) and write per-taxon fold changes, recovery percentages, and the Levene's statistic to a results CSV.

## Related tools

- **MiniBar** (Demultiplexes dual-indexed MinION reads by dual barcodes and primers, bins reads into per-sample files (-F) and trims barcodes/primers (-T) prior to taxon counting) — https://github.com/calacademy-research/minibar
- **MinION** (Oxford Nanopore platform producing the long rDNA amplicon reads whose mock-community recovery is quantified)
- **Illumina** (Short-read MiSeq 18S platform serving as the optimized comparator protocol for recovery accuracy)
- **Edlib** (Python alignment library used by MiniBar for edit-distance matching of barcodes and primers) — https://github.com/Martinsos/edlib
- **Python 2.7** (Language in which MiniBar runs (compatible with Python 3); used for the counting/fold-change/Levene's-test analysis)

## Examples

```
./minibar.py -T -F Demultiplex.txt mock_community.fastq
```

## Evaluation signals

- Qualitative recovery reproduces the reported contrast: Nanopore recovers on average 83.33% of the nine input taxa per mock community, Illumina 18S recovers all nine taxa from all four mock communities.
- Levene's-test outcome on the fold-change distributions is consistent with the reported greater quantitative bias of Nanopore long-amplicon sequencing versus Illumina.
- Per-sample read counts are computed only from trimmed amplicon sequences (barcode and primer removed via -T), not raw reads.
- Every fold change is computable: each recovered taxon has both a known input DNA amount and a recovered read count for that platform and mock community.
- Output CSV contains per-taxon fold changes, per-platform recovery percentages, and a single Levene's-test statistic.

## Limitations

- Long rDNA amplicon sequencing is not yet reliable for bulk sample analysis; the source of the observed taxonomic bias is undetermined and further research is required.
- Nanopore long-amplicon recovery of mock communities is highly skewed in both qualitative and quantitative terms, so abundance estimates should not be taken at face value.
- Mock-community recovery here was tested on only four mock communities with nine taxa; variance estimates rest on that small design.

## Evidence

- [abstract] Long rDNA amplicon metabarcoding recovers highly skewed diversity estimates: "find that they recover highly skewed diversity estimates"
- [intro] Mock community comparison design: "We compare the qualitative (species richness) and quantitative (species abundance) recovery of taxa in simple mock communities by long-amplicon sequencing with that based on short read Illumina"
- [discussion] Bulk sample reliability caveat: "Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples."
- [readme] MiniBar trimming option: "-T trims barcode and primer from each end of the sequence, then outputs record"
- [readme] MiniBar defaults used in demultiplexing: "-e <int> barcode edit distance value, overrides -p (4)
        -E <int> primer edit distance value (11)
        -l <int> length to search for index and primer at start and end of sequence (80)"
- [readme] MiniBar default invocation: "./minibar.py -T -F Demultiplex.txt my_sequences.fastq"
