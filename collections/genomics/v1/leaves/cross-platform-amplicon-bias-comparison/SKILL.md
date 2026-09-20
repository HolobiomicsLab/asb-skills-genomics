---
name: cross-platform-amplicon-bias-comparison
description: Use when comparing qualitative (species richness) and quantitative (read-abundance)
  recovery of the same mock community across two amplicon sequencing platforms, or
  when testing whether long-amplicon Nanopore metabarcoding accurately recovers taxon
  composition.
when_to_use_negative:
- Input is a single-platform dataset with no cross-platform comparison target — there
  is no basis for fold-change or Levene's-test contrast
- Input reads have not been demultiplexed and trimmed — minibar binning/trimming must
  be run first or taxon counts will be confounded by index/primer sequence
- Goal is phylogenetic resolution or consensus error profiling of single isolates
  rather than bulk-sample taxon recovery quantification
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed Nanopore reads by barcode/primer edit distance,
    bins reads into per-sample files (-F) and trims barcodes/primers (-T) before taxon
    assignment
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Edit-distance alignment library imported by minibar to compute barcode and
    primer match distances
  repo: https://github.com/Martinsos/edlib
- name: MinION
  role: Oxford Nanopore platform that generated the long-amplicon rDNA mock-community
    reads being assessed for bias
- name: Illumina
  role: Short-read MiSeq 18S platform serving as the optimized reference for mock-community
    recovery comparison
- name: Python 2.7
  role: Language of minibar.py (compatible with Python 3), used for the demultiplexing/counting
    steps
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:dedfa2e128d6b8bc9aee3485273d88f1c9e9b65ee30af9b87d06f689503d33f9
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
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-platform-amplicon-bias-comparison

## Summary

Reproduces the Krehenwinkel et al. (2019) mock-community comparison of Nanopore long-amplicon rDNA metabarcoding versus Illumina short-amplicon 18S, quantifying qualitative taxon recovery and per-taxon fold-change bias between known input DNA amounts and recovered read counts.

## When to use

Use when comparing qualitative (species richness) and quantitative (read-abundance) recovery of the same mock community across two amplicon sequencing platforms, or when testing whether long-amplicon Nanopore metabarcoding accurately recovers taxon composition.

## When NOT to use

- Input is a single-platform dataset with no cross-platform comparison target — there is no basis for fold-change or Levene's-test contrast
- Input reads have not been demultiplexed and trimmed — minibar binning/trimming must be run first or taxon counts will be confounded by index/primer sequence
- Goal is phylogenetic resolution or consensus error profiling of single isolates rather than bulk-sample taxon recovery quantification

## Inputs

- Nanopore MinION mock-community FASTQ files (dual-indexed long rDNA amplicons)
- Illumina MiSeq FASTQ files from the optimized 18S short-amplicon protocol
- Dual-index barcode file (e.g. Demultiplex.txt) mapping sample IDs to fwd/rev indexes and primers
- Reference taxonomy mapping amplicon sequences to taxa
- Known input DNA amounts per taxon per mock community

## Outputs

- mock_community_bias_results.csv with per-taxon fold changes, per-platform taxon recovery percentages, and Levene's-test statistic
- Per-sample read counts per taxon for each platform

## How to apply

Demultiplex the Nanopore mock-community FASTQ files with MiniBar using the dual-index barcode file at defaults (Method 3, barcode edit distance 4, primer edit distance 11, search length 80), binning reads per sample with -F and trimming barcodes/primers with -T. Assign trimmed amplicon sequences to taxa by matching against the reference taxonomy and count recovered reads per taxon per mock community; repeat the counting for the Illumina MiSeq 18S FASTQ files. For each taxon in each mock community and platform, compute the fold change between the known input DNA amount and the recovered read count. Summarize qualitative recovery as the percentage of input taxa recovered (Nanopore ≈83.33% vs Illumina 100% across four mock communities of nine taxa) and test whether fold-change variance differs between platforms using Levene's test, writing per-taxon fold changes, recovery percentages, and the Levene statistic to mock_community_bias_results.csv.

## Related tools

- **MiniBar** (Demultiplexes dual-indexed Nanopore reads by barcode/primer edit distance, bins reads into per-sample files (-F) and trims barcodes/primers (-T) before taxon assignment) — https://github.com/calacademy-research/minibar
- **Edlib** (Edit-distance alignment library imported by minibar to compute barcode and primer match distances) — https://github.com/Martinsos/edlib
- **MinION** (Oxford Nanopore platform that generated the long-amplicon rDNA mock-community reads being assessed for bias)
- **Illumina** (Short-read MiSeq 18S platform serving as the optimized reference for mock-community recovery comparison)
- **Python 2.7** (Language of minibar.py (compatible with Python 3), used for the demultiplexing/counting steps)

## Examples

```
./minibar.py -T -F Demultiplex.txt mock_community.fastq
```

## Evaluation signals

- Qualitative recovery reproduces the reported contrast: Nanopore recovers on average 83.33% of input taxa per mock community while Illumina 18S recovers all nine taxa in all four mock communities
- Levene's test on fold-change distributions is consistent with the reported quantitative bias result (Nanopore fold-change variance higher than Illumina)
- Per-taxon fold changes are finite and comparable across the four mock communities for both platforms
- Minibar run summary line shows most reads assigned cleanly (HH majority, few Mult_IDs), indicating demultiplexing did not distort sample-level counts
- mock_community_bias_results.csv contains per-taxon rows with recovery percentage and Levene statistic fields populated

## Limitations

- The source of the observed taxonomic bias in bulk samples is undetermined; the authors state further research is required before long rDNA amplicon sequencing can be used as a reliable resource for bulk-sample analysis
- Long rDNA amplicon Nanopore metabarcoding yields highly skewed diversity estimates and should not be treated as quantitatively reliable for community composition without correction
- Demultiplexing quality depends on edit-distance tolerances (barcode ≤4, primer ≤11); reads whose primers exceed the threshold (h- hits) may be lost or misbinned

## Evidence

- [intro] Mock community recovery contrast: "We compare the qualitative (species richness) and quantitative (species abundance) recovery of taxa in simple mock communities by long-amplicon sequencing with that based on short read Illumina"
- [abstract] Biased recovery finding: "find that they recover highly skewed diversity estimates"
- [methods] Minibar demultiplexing step: "Identify MinION sequence by dual barcode indexes and primers."
- [methods] Binning and trimming options: "Typically you'll want to group your records by identified sample. This can be done by using the -F option"
- [discussion] Bias source undetermined: "Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples."
- [readme] Minibar defaults and invocation: "minibar defaults are set so a typical run might look like:
./minibar.py -T -F Demultiplex.txt my_sequences.fastq"
- [readme] Edit distance defaults: "-e <int> barcode edit distance value, overrides -p (4)
        -E <int> primer edit distance value (11)
        -l <int> length to search for index and primer at start and end of sequence (80)"
