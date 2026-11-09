---
name: metagenomic-bin-quality-assessment
description: Use when bins from one or more metagenomic binners must be quantitatively
  compared on simulated data with known reference genomes, e.g. counting high-quality
  bins per binner per CAMI II environment.
when_to_use_negative:
- Data has no ground-truth reference genomes (e.g. real unassembled samples), so AMBER
  precision/recall/F1 cannot be computed.
- The goal is to train or run a binner rather than compare bin outputs across binners.
- Inputs are already finished evaluation tables — this skill covers binning runs and
  AMBER scoring, not re-analyzing existing summaries.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SemiBin2
  role: Focal binner run in long-read mode (--sequencing-type=long_read) or self-supervised
    short-read mode; its ensemble-based DBSCAN clustering yields 13.1-26.3% more high-quality
    genomes than the second-best long-read binner.
  repo: github.com/BigDataBiology/SemiBin
- name: samtools
  role: Sorts read-to-contig alignments into the sorted BAM files used as binner inputs
    for the comparison.
  repo: https://github.com/samtools/samtools
- name: AMBER
  role: Evaluates every binner's bins against CAMI II ground-truth genomes to produce
    precision, recall, F1, and high-quality bin counts.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:a3e9e130272b5a2416bf1fc1d36e2b0a28ff6a4089350b3506a8738d36b8dd7f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AMBER
  - SemiBin1
  - SemiBin2
  - samtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# metagenomic-bin-quality-assessment

## Summary

Assess the quality of metagenomic bins produced by binners such as SemiBin2, MetaBAT2, VAMB, CONCOCT, SemiBin1, MetaDecoder, and MetaBinner by scoring them against ground-truth genomes with AMBER (precision, recall, F1, high-quality bin counts). This is the evaluation backbone of the SemiBin2 benchmark on CAMI II short- and long-read simulated datasets.

## When to use

Use when bins from one or more metagenomic binners must be quantitatively compared on simulated data with known reference genomes, e.g. counting high-quality bins per binner per CAMI II environment.

## When NOT to use

- Data has no ground-truth reference genomes (e.g. real unassembled samples), so AMBER precision/recall/F1 cannot be computed.
- The goal is to train or run a binner rather than compare bin outputs across binners.
- Inputs are already finished evaluation tables — this skill covers binning runs and AMBER scoring, not re-analyzing existing summaries.

## Inputs

- assembled contig FASTA file per sample/environment
- sorted BAM alignments of reads mapped to the contigs (sorted with samtools)
- binner output bins (e.g. output_bins from SemiBin2)
- CAMI II ground-truth reference genomes for the simulated dataset

## Outputs

- AMBER precision, recall, and F1 scores per binner per environment
- count of high-quality bins per binner per environment
- percentage gain of SemiBin2 over the second-best binner per environment
- per-environment comparison summary table (amber_longread_comparison.csv)

## How to apply

Run each binner (e.g. SemiBin2 via `single_easy_bin` with `--sequencing-type=long_read` for PacBio long-read data, or the comparison binners MetaBAT2, MetaDecoder, VAMB, SemiBin1, CONCOCT, MetaBinner) on the same assembled contig FASTA and sorted BAM alignment inputs. Evaluate every binner's bins from all environments with AMBER against the CAMI II ground-truth genomes to obtain precision, recall, F1, and per-binner high-quality bin counts. Tabulate high-quality bins per binner per environment and compute the percentage gain of the focal binner over the second-best binner; for SemiBin2 on CAMI II long-read data the expected gains are 27.1% (airways), 21.5% (gastrointestinal), 52.7% (oral), 47.5% (skin), and 14.4% (urogenital). Save the per-environment comparison summary as a CSV such as amber_longread_comparison.csv. Correctness is judged by consistent inputs across binners, valid AMBER metric outputs, and agreement with the reported percentage-gain ranges.

## Related tools

- **SemiBin2** (Focal binner run in long-read mode (--sequencing-type=long_read) or self-supervised short-read mode; its ensemble-based DBSCAN clustering yields 13.1-26.3% more high-quality genomes than the second-best long-read binner.) — github.com/BigDataBiology/SemiBin
- **samtools** (Sorts read-to-contig alignments into the sorted BAM files used as binner inputs for the comparison.) — https://github.com/samtools/samtools
- **AMBER** (Evaluates every binner's bins against CAMI II ground-truth genomes to produce precision, recall, F1, and high-quality bin counts.)

## Examples

```
SemiBin2 single_easy_bin -i contigs.fasta -b sorted.bam --sequencing-type=long_read -o output_bins
```

## Evaluation signals

- AMBER produces valid precision, recall, and F1 values for each binner's bins against CAMI II ground-truth genomes.
- High-quality bin counts are tabulated per binner per environment (airways, gastrointestinal, oral, skin, urogenital).
- SemiBin2's percentage gain over the second-best binner matches the expected long-read values: 27.1%, 21.5%, 52.7%, 47.5%, 14.4%.
- All binners consume identical per-environment contig FASTA and sorted BAM inputs (no input leakage or divergence).
- The comparison summary file amber_longread_comparison.csv exists with one row per binner per environment.

## Limitations

- Results are on CAMI II simulated (PacBio long-read and short-read) datasets; simulated ground truth may not transfer to real samples.
- SemiBin operations are probabilistic — running it twice can return different results, so bin counts may vary between reruns (noted in the SemiBin_tutorial README).
- The tutorial data is a toy dataset that should not be used to draw biological conclusions.

## Evidence

- [abstract] ensemble-based DBSCAN clustering yields 13.1-26.3% more high-quality genomes than the second best binner for long-read data: "we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data."
- [abstract] SemiBin2 reconstructs 8.3-21.5% more high-quality bins than SemiBin1 on real short-read samples: "Compared to SemiBin1, SemiBin2 can reconstruct 8.3-21.5% more high-quality bins and requires only 25% of the running time and 11% of peak memory usage in real short-read sequencing samples."
- [methods] workflow step 1: assemble reads into a contigs FASTA file: "**Step 1**: Assemble it into a contigs FASTA file."
- [methods] Step 2: map reads to the contig FASTA to produce alignments for binning: "**Step 2**: Map reads to the FASTA file generated in _Step 1_."
- [readme] SemiBin2 runs are probabilistic and reruns give different results: [evidence span withheld — anchor status: external_doc]
