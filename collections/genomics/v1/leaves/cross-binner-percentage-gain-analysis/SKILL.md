---
name: cross-binner-percentage-gain-analysis
description: Use when benchmarking metagenomic binners across multi-sample short-read
  projects (e.g. human gut, dog gut, ocean, soil) and you need per-binner high-quality
  bin counts plus percentage improvement over a baseline binner.
when_to_use_negative:
- Input is a single-sample binning task — the multi_easy_bin multi-sample workflow
  and its gain figures (82/129/109/101-sample projects) do not transfer to single_easy_bin
  runs
- The goal is long-read binning benchmarking — SemiBin2's long-read advantage (13.1-26.3%
  more high-quality genomes via ensemble-based DBSCAN clustering) is a separate finding
  from these short-read percentage gains
- You lack CheckM and GUNC — without completeness/contamination/chimerism assessment
  you cannot define high-quality bins and the comparison is meaningless
edam_topics:
- http://edamontology.org/topic_3828
tools:
- name: SemiBin2
  role: Binner under test, run in self-supervised mode with multi_easy_bin
  repo: github.com/BigDataBiology/SemiBin
- name: SemiBin1
  role: Baseline binner run in semi-supervised mode (generate_cannot_links, train_semi,
    bin_short)
  repo: github.com/BigDataBiology/SemiBin
- name: VAMB
  role: Competing binner run on identical contig and BAM inputs for fair comparison
- name: CheckM
  role: Classifies bins by completeness/contamination for high-quality bin counting
- name: GUNC
  role: Detects chimeric bins as part of the high-quality classification
- name: mmseqs2
  role: Contig annotation against GTDB for cannot-link generation in SemiBin1's semi-supervised
    mode
- name: samtools
  role: Produces sorted BAM files of reads mapped to contigs as binner inputs
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:a3ba5c7c74153b2862244601e51e19a0a7543190c1ad900d4c28fbb0e9ff81c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - GUNC
  - SemiBin1
  - SemiBin2
  - VAMB
  - mmseqs2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3828
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# Cross-binner percentage gain analysis

## Summary

Quantifies how many more high-quality metagenomic bins one binner (e.g. SemiBin2) reconstructs than competitors (SemiBin1, VAMB) by running all binners on identical contig+BAM inputs, classifying bins with CheckM/GUNC, and computing percentage gains with Wilcoxon signed-rank significance testing.

## When to use

Use when benchmarking metagenomic binners across multi-sample short-read projects (e.g. human gut, dog gut, ocean, soil) and you need per-binner high-quality bin counts plus percentage improvement over a baseline binner.

## When NOT to use

- Input is a single-sample binning task — the multi_easy_bin multi-sample workflow and its gain figures (82/129/109/101-sample projects) do not transfer to single_easy_bin runs
- The goal is long-read binning benchmarking — SemiBin2's long-read advantage (13.1-26.3% more high-quality genomes via ensemble-based DBSCAN clustering) is a separate finding from these short-read percentage gains
- You lack CheckM and GUNC — without completeness/contamination/chimerism assessment you cannot define high-quality bins and the comparison is meaningless

## Inputs

- Contig FASTA files assembled per sample (or concatenated for multi-sample binning)
- Sorted BAM files mapping reads to contigs, used as binner inputs
- SemiBin1 and SemiBin2 commands from the SemiBin package; VAMB
- GTDB reference database for cannot-link generation (SemiBin1 semi-supervised mode)
- CheckM and GUNC for bin quality classification

## Outputs

- High-quality bin counts per binner per project (SemiBin2 targets: 1678, 3776, 631, 254)
- Percentage gains of SemiBin2 over SemiBin1 (8.3-21.5%) and VAMB (27.3-229.9%)
- Wilcoxon signed-rank P-values from paired per-sample quality score comparisons
- benchmark_results.csv

## How to apply

For each multi-sample project (82, 129, 109, and 101 samples in the SemiBin2 study), assemble reads into contig FASTA files and map reads back to produce sorted BAM files shared by all binners. Run SemiBin2 with its default self-supervised mode via multi_easy_bin, run SemiBin1 in semi-supervised mode (generate_cannot_links with mmseqs2/GTDB, train_semi, then bin_short), and run VAMB on the identical inputs. Assess every reconstructed bin with CheckM (completeness/contamination) and GUNC (chimerism) to classify high-quality bins per project, then count high-quality bins per binner. Compute percentage gains (SemiBin2 targets: 8.3-21.5% over SemiBin1; 27.3-229.9% over VAMB), apply the Wilcoxon signed-rank test to paired per-sample quality scores, and write all values to benchmark_results.csv.

## Related tools

- **SemiBin2** (Binner under test, run in self-supervised mode with multi_easy_bin) — github.com/BigDataBiology/SemiBin
- **SemiBin1** (Baseline binner run in semi-supervised mode (generate_cannot_links, train_semi, bin_short)) — github.com/BigDataBiology/SemiBin
- **VAMB** (Competing binner run on identical contig and BAM inputs for fair comparison)
- **CheckM** (Classifies bins by completeness/contamination for high-quality bin counting)
- **GUNC** (Detects chimeric bins as part of the high-quality classification)
- **mmseqs2** (Contig annotation against GTDB for cannot-link generation in SemiBin1's semi-supervised mode)
- **samtools** (Produces sorted BAM files of reads mapped to contigs as binner inputs) — https://github.com/samtools/samtools

## Examples

```
SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : --self-supervised -o SemiBin2
```

## Evaluation signals

- Per-project SemiBin2 high-quality bin counts reproduce the reported values 1678, 3776, 631, and 254 (human gut, dog gut, ocean, soil)
- Percentage gains fall within the reported ranges: 8.3-21.5% vs SemiBin1 and 27.3-229.9% vs VAMB
- Wilcoxon signed-rank P-values indicate statistically significant improvement over both baselines
- All three binners were run on identical contig FASTA and BAM inputs — any input divergence invalidates the comparison
- All counts and gains are written to benchmark_results.csv with per-project rows

## Limitations

- SemiBin runs are probabilistic; running the same step twice can return different bins, so exact count reproduction may vary (per the SemiBin tutorial note)
- The reported gain percentages are specific to the four real short-read projects and their sample sizes (82, 129, 109, 101); they should not be treated as universal constants
- Fair comparison requires running SemiBin1 in its semi-supervised mode with GTDB-based cannot-links and VAMB on the same inputs; using different preprocessing per binner biases the gains

## Evidence

- [abstract] percentage gains vs baselines: "Compared to SemiBin1, SemiBin2 can reconstruct 8.3-21.5% more high-quality bins and requires only 25% of the running time and 11% of peak memory usage in real short-read sequencing samples."
- [methods] multi-sample binner input format: "The command `multi_easy_bin` requires the combined contig file from several samples, BAM files (reads mapping to the combined contig) as inputs"
- [methods] semi-supervised baseline workflow: "`generate_cannot_links` (this is the extra step, compared to the self-supervised mode"
- [readme] VAMB benchmark invocation: [evidence span withheld — anchor status: external_doc]
- [readme] SemiBin2 benchmark invocation: [evidence span withheld — anchor status: external_doc]
