---
name: dna-input-vs-read-fold-change-computation
description: Use when mock-community metabarcoding data exist with known per-taxon
  input DNA amounts and per-taxon read counts, and you need to quantify amplification/sequencing
  bias per platform (e.g. Nanopore long-amplicon vs Illumina 18S).
when_to_use_negative:
- The input is already a bias-corrected abundance or composition table with no known
  input DNA amounts to normalize against
- The sample is a natural bulk community with unknown taxon abundances, so no ground-truth
  input-vs-read comparison is possible
- The goal is phylogenetic placement or barcode identification rather than quantitative
  bias assessment
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed Nanopore reads (Method 3, barcode edit distance
    4, primer edit distance 11, search length 80), bins them into per-sample files
    with -F, and trims barcodes/primers with -T to produce the amplicon sequences
    that are counted per taxon
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Provides the edit-distance alignment used by MiniBar to match dual barcode
    indexes and primers at read ends
  repo: https://github.com/Martinsos/edlib
- name: MinION
  role: Oxford Nanopore platform that generated the long rDNA amplicon reads whose
    per-taxon recovery is compared against known input DNA amounts
- name: Illumina
  role: Short-amplicon 18S MiSeq sequencing platform used as the quantitative-recovery
    benchmark for the fold-change and Levene's-test comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:b1198e99b70140d69830b19172413bb4d5da961f0a544055e107a98c847f6236
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# DNA input vs read fold-change computation

## Summary

Quantifies metabarcoding bias by computing, for each taxon in each mock community and each sequencing platform, the fold change between the known input DNA amount and the recovered read counts, enabling variance-based (Levene's test) comparison of quantitative recovery.

## When to use

Use when mock-community metabarcoding data exist with known per-taxon input DNA amounts and per-taxon read counts, and you need to quantify amplification/sequencing bias per platform (e.g. Nanopore long-amplicon vs Illumina 18S).

## When NOT to use

- The input is already a bias-corrected abundance or composition table with no known input DNA amounts to normalize against
- The sample is a natural bulk community with unknown taxon abundances, so no ground-truth input-vs-read comparison is possible
- The goal is phylogenetic placement or barcode identification rather than quantitative bias assessment

## Inputs

- Demultiplexed, trimmed Nanopore fastq reads per mock community (amplicon-only sequences after MiniBar -T/-F)
- Illumina MiSeq fastq files for the same mock communities
- Reference taxonomy mapping amplicon sequences to taxa
- Table of known per-taxon input DNA amounts for each mock community

## Outputs

- Per-taxon fold-change values (input DNA amount vs recovered reads) for each taxon in each mock community per platform
- Per-platform qualitative recovery percentage (e.g. 83.33% for Nanopore vs 100% for Illumina)
- Levene's-test statistic and outcome comparing fold-change variance between platforms
- mock_community_bias_results.csv

## How to apply

First demultiplex raw MinION reads with MiniBar using the dual-index barcode file with default Method 3 (barcode edit distance 4, primer edit distance 11, search length 80), binning reads per sample with -F and trimming barcodes/primers with -T so only the amplicon sequence remains. Then assign trimmed amplicon sequences to taxa by matching against the reference taxonomy and count recovered reads per taxon per mock community, repeating the counting for the Illumina MiSeq fastq files. For each taxon in each mock community and platform, compute the fold change between the known input DNA amount and the recovered read count. Compare qualitative recovery (percentage of the nine input taxa recovered per community) and run Levene's test on the fold-change distributions to test whether the variance of quantitative recovery differs between Nanopore and Illumina. Write per-taxon fold changes, recovery percentages, and the Levene's-test statistic to a results table (mock_community_bias_results.csv).

## Related tools

- **MiniBar** (Demultiplexes dual-indexed Nanopore reads (Method 3, barcode edit distance 4, primer edit distance 11, search length 80), bins them into per-sample files with -F, and trims barcodes/primers with -T to produce the amplicon sequences that are counted per taxon) — https://github.com/calacademy-research/minibar
- **Edlib** (Provides the edit-distance alignment used by MiniBar to match dual barcode indexes and primers at read ends) — https://github.com/Martinsos/edlib
- **MinION** (Oxford Nanopore platform that generated the long rDNA amplicon reads whose per-taxon recovery is compared against known input DNA amounts)
- **Illumina** (Short-amplicon 18S MiSeq sequencing platform used as the quantitative-recovery benchmark for the fold-change and Levene's-test comparison)

## Examples

```
./minibar.py -T -F Demultiplex.txt my_sequences.fastq
```

## Evaluation signals

- Qualitative recovery for the Nanopore long-amplicon mock communities averages 83.33% of input taxa (i.e. ~7.5 of 9 taxa per community), while Illumina 18S recovers all nine taxa from all four mock communities
- Fold changes are computed for every taxon in every mock community on both platforms, with no missing input-taxa entries
- Levene's test on the fold-change distributions yields a variance comparison consistent with the reported stronger quantitative bias of the Nanopore platform
- mock_community_bias_results.csv contains per-taxon fold changes, recovery percentages, and the Levene's-test statistic
- Per-sample read counts derive from MiniBar runs whose summary line shows reads identified to sample IDs (IDs count) with Mult_IDs 0

## Limitations

- The source of the observed taxonomic bias in bulk samples is undetermined; the authors state that further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be used as a reliable resource for bulk-sample analysis
- Long rDNA amplicon sequencing on MinION recovered highly skewed diversity estimates, so fold changes from this platform reflect very biased quantitative recovery
- Fold-change variance comparisons depend on the small number of mock communities (four) and nine input taxa, limiting statistical power

## Evidence

- [results] on average only 83.33% of taxa were recovered per Nanopore sequenced mock community, whereas the Illumina 18S protocol recovered all nine taxa from all four mock communities: [evidence span withheld — anchor status: not_found]
- [intro] We compare the qualitative (species richness) and quantitative (species abundance) recovery of taxa in simple mock communities by long-amplicon sequencing with that based on short read Illumina: "We compare the qualitative (species richness) and quantitative (species abundance) recovery of taxa in simple mock communities by long-amplicon sequencing with that based on short read Illumina"
- [readme] This can be done by using the -F option, optionally with -P to use a file prefix other than the "sample_" default.: "Typically you'll want to group your records by identified sample. This can be done by using the -F option, optionally with -P to use a file prefix other than the "sample_" default."
- [readme] For downstream processing, you'll probably want to trim off the barcode and primer part of the sequence. This is done for you if you supply the -T trim option: "For downstream processing, you'll probably want to trim off the barcode and primer part of the sequence. This is done for you if you supply the -T trim option"
- [discussion] Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples.: "Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples."
