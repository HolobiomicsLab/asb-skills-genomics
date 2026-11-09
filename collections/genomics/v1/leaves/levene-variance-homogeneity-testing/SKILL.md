---
name: levene-variance-homogeneity-testing
description: Use when comparing abundance-recovery bias between two sequencing platforms
  and you need a formal test of whether the spread of per-taxon fold-change values
  differs, not just the means.
when_to_use_negative:
- Comparing mean recovery rather than variance of recovery — Levene's test addresses
  homogeneity of variance, not central tendency
- Qualitative presence/absence assessment only (e.g., percentage of input taxa recovered),
  where no fold-change distribution exists
- Single-sample, single-taxon comparisons with too few fold-change values to support
  a variance test
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3305
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed Nanopore reads into per-sample files (-F) and trims
    barcodes/primers (-T) before read counting and fold-change computation
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library module used by minibar for edit-distance matching of barcode
    indexes and primers during demultiplexing
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Scripting environment for the minibar pipeline and downstream fold-change/Levene's-test
    analysis (compatible with Python 3)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:cf376fc307009649bd38a239d7f072154c6b0fde21f44eace6c03e78bf84ca25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - Illumina
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# levene-variance-homogeneity-testing

## Summary

Apply Levene's test to compare the variance of per-taxon fold-change distributions (known input DNA amount vs recovered read counts) across sequencing platforms, quantifying whether quantitative recovery bias differs between Nanopore long-amplicon and Illumina short-amplicon metabarcoding of mock communities.

## When to use

Use when comparing abundance-recovery bias between two sequencing platforms and you need a formal test of whether the spread of per-taxon fold-change values differs, not just the means.

## When NOT to use

- Comparing mean recovery rather than variance of recovery — Levene's test addresses homogeneity of variance, not central tendency
- Qualitative presence/absence assessment only (e.g., percentage of input taxa recovered), where no fold-change distribution exists
- Single-sample, single-taxon comparisons with too few fold-change values to support a variance test

## Inputs

- Demultiplexed, barcode/primer-trimmed Nanopore FASTQ files per mock community (from minibar -F -T)
- Illumina MiSeq FASTQ files for the same four mock communities
- Reference taxonomy / amplicon sequence table mapping sequences to taxa
- Table of known input DNA amounts per taxon per mock community

## Outputs

- mock_community_bias_results.csv containing per-taxon fold changes, per-platform qualitative recovery percentages, and the Levene's-test statistic
- Per-taxon read count tables per mock community for Nanopore and Illumina

## How to apply

First demultiplex the Nanopore mock-community FASTQ files with minibar using the dual-index barcode file (default Method 3, barcode edit distance 4, primer edit distance 11, search length 80), binning reads per sample with -F and trimming barcodes/primers with -T so only the amplicon remains. Assign trimmed reads to taxa by matching amplicon sequences against the reference taxonomy and count reads per taxon per mock community, repeating for the Illumina MiSeq FASTQ. Compute the per-taxon fold change between the known input DNA amount and recovered read counts for each taxon in each mock community and platform. Run Levene's test on these fold-change distributions to compare variance of quantitative recovery between platforms; in this study Nanopore long-amplicon sequencing produced far more variable fold changes than the Illumina 18S protocol, which recovered all nine input taxa from all four mock communities with accurate abundance predictions. Write fold changes, recovery percentages, and the Levene statistic to mock_community_bias_results.csv.

## Related tools

- **MiniBar** (Demultiplexes dual-indexed Nanopore reads into per-sample files (-F) and trims barcodes/primers (-T) before read counting and fold-change computation) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library module used by minibar for edit-distance matching of barcode indexes and primers during demultiplexing) — https://github.com/Martinsos/edlib
- **Python 2.7** (Scripting environment for the minibar pipeline and downstream fold-change/Levene's-test analysis (compatible with Python 3))

## Examples

```
./minibar.py -T -F Demultiplex.txt my_sequences.fastq
```

## Evaluation signals

- Nanopore qualitative recovery per mock community averages ~83.33% of input taxa, while Illumina 18S recovers all nine taxa from all four mock communities (100%)
- Levene's test on the fold-change distributions yields an outcome consistent with the reported result: significantly greater variance (quantitative bias) for Nanopore than for Illumina
- Fold changes are computed as recovered read counts divided by known input DNA amount for every taxon in every mock community on both platforms
- Output CSV schema includes per-taxon fold changes, recovery percentages, and the Levene's-test statistic in mock_community_bias_results.csv
- minibar run summary reports plausible demultiplexing counts (e.g., HH/Hh/hh categories with Mult_IDs 0) before downstream statistics

## Limitations

- The source of the observed taxonomic bias in bulk samples is undetermined; further research is required before long rDNA amplicon sequencing can be used reliably for bulk sample analysis
- Long rDNA amplicon Nanopore sequencing recovers highly skewed diversity estimates, so fold-change variance may reflect platform-specific amplification/sequencing artifacts
- minibar primer misses (lowercase 'h' hits, -1 edit distance) mean some reads carry barcode-only matches and may be excluded, affecting downstream counts

## Evidence

- [results] on average only 83.33% of taxa were recovered per Nanopore sequenced mock community, whereas the Illumina 18S protocol recovered all nine taxa from all four mock communities with accurate abundance predictions: [evidence span withheld — anchor status: not_found]
- [methods] Compute the per-taxon fold change between the known input DNA amount and the recovered read counts for each taxon in each mock community and platform.: [evidence span withheld — anchor status: not_found]
- [methods] run Levene's test on the fold-change distributions to compare the variance of quantitative recovery between Nanopore and Illumina: [evidence span withheld — anchor status: not_found]
- [intro] We compare the qualitative (species richness) and quantitative (species abundance) recovery of taxa in simple mock communities: "We compare the qualitative (species richness) and quantitative (species abundance) recovery of taxa in simple mock communities"
- [discussion] Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource: "Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples."
