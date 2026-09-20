---
name: multi-sample-short-read-binning-workflow
description: Use when reconstructing MAGs from dozens of short-read metagenomic samples
  (e.g., human gut, dog gut, ocean, soil projects with 82–129 samples) for which assembled
  contigs and sorted BAM files are available and joint binning is desired.
when_to_use_negative:
- Only one sample is being binned — the paper prescribes `single_easy_bin` for that
  case instead of the combined multi-sample inputs
- Input is long-read data — SemiBin2 instead uses its ensemble-based DBSCAN clustering
  path for long reads, a separate workflow from multi_easy_bin short-read binning
- Bins already exist and the goal is only taxonomic annotation or quality re-assessment
  rather than de novo binning
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SemiBin/SemiBin2
  role: 'Primary binner: runs `multi_easy_bin` in self-supervised mode on the combined
    contig FASTA plus BAM files'
  repo: https://github.com/BigDataBiology/SemiBin
- name: SemiBin1
  role: Semi-supervised comparison binner run via generate_cannot_links (mmseqs2/GTDB),
    train_semi, and bin_short on identical inputs
  repo: https://github.com/BigDataBiology/SemiBin
- name: VAMB
  role: Third binner benchmarked on the same contigs and BAM files (e.g., --minfasta
    200000 -m 2000)
- name: CheckM
  role: Scores completeness and contamination of every bin to define high-quality
    bins per project
- name: GUNC
  role: Flags chimeric bins so high-quality classification accounts for chimerism
- name: strobealign
  role: Optional all-by-all cross-sample abundance estimation via strobealign --aemb
    (version 0.13 or newer)
  repo: https://github.com/ksahlin/strobealign
- name: samtools
  role: External dependency used in the mapping step to produce sorted BAM files from
    reads mapped to contigs
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:48476ccfdaaddb154263f7f1263e30828b59322e13a0a4e284a5d53bcf901124
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - CheckM
  - GUNC
  - SemiBin/SemiBin2
  - SemiBin1
  - SemiBin2
  - VAMB
  - mmseqs2
  - samtools
  - strobealign
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# multi-sample-short-read-binning-workflow

## Summary

Reconstructs metagenome-assembled genomes from many short-read samples in one joint SemiBin2 run, using cross-sample co-abundance features (multi_easy_bin, self-supervised mode). It is the paper's core short-read benchmarking path that yielded 8.3–21.5% more high-quality bins than SemiBin1 at 25% of the runtime and 11% of the peak memory.

## When to use

Use when reconstructing MAGs from dozens of short-read metagenomic samples (e.g., human gut, dog gut, ocean, soil projects with 82–129 samples) for which assembled contigs and sorted BAM files are available and joint binning is desired.

## When NOT to use

- Only one sample is being binned — the paper prescribes `single_easy_bin` for that case instead of the combined multi-sample inputs
- Input is long-read data — SemiBin2 instead uses its ensemble-based DBSCAN clustering path for long reads, a separate workflow from multi_easy_bin short-read binning
- Bins already exist and the goal is only taxonomic annotation or quality re-assessment rather than de novo binning

## Inputs

- Per-sample short-read sequencing files (FASTQ) from multi-sample projects
- Per-sample assembled contigs FASTA files
- Sorted BAM files of reads mapped back to the (combined) contigs
- GTDB reference genomes (only for the SemiBin1 semi-supervised cannot-link generation)

## Outputs

- Bins (MAG FASTA files) per sample from the SemiBin2 multi_easy_bin output directory
- CheckM completeness/contamination table for every reconstructed bin
- GUNC chimerism report for every reconstructed bin
- benchmark_results.csv with per-project high-quality bin counts, percentage gains vs SemiBin1/VAMB, and Wilcoxon signed-rank P-values

## How to apply

Assemble each sample's reads into a contigs FASTA (Step 1), map reads back to the contigs and sort to produce BAM files (Step 2, e.g., with ngless/samtools), then concatenate the per-sample FASTAs with `concatenate_fasta` so every contig name carries its sample name plus a separator. Run SemiBin2 `multi_easy_bin` on the combined contig file plus all BAMs in its default self-supervised mode (e.g., with `--orf-finder fraggenescan`); cross-sample abundance is computed internally, or precomputed all-by-all with `strobealign --aemb`. For a fair comparison, run SemiBin1's semi-supervised pipeline on the identical inputs (`generate_cannot_links` with mmseqs2 against GTDB, `train_semi`, `bin_short`) and VAMB (`vamb --fasta ... --bamfiles *.bam --minfasta 200000 -m 2000`). Assess every reconstructed bin from all binners with CheckM (completeness/contamination) and GUNC (chimerism) to classify high-quality bins per project, then apply the Wilcoxon signed-rank test to paired per-sample quality scores and record P-values in benchmark_results.csv.

## Related tools

- **SemiBin/SemiBin2** (Primary binner: runs `multi_easy_bin` in self-supervised mode on the combined contig FASTA plus BAM files) — https://github.com/BigDataBiology/SemiBin
- **SemiBin1** (Semi-supervised comparison binner run via generate_cannot_links (mmseqs2/GTDB), train_semi, and bin_short on identical inputs) — https://github.com/BigDataBiology/SemiBin
- **VAMB** (Third binner benchmarked on the same contigs and BAM files (e.g., --minfasta 200000 -m 2000))
- **CheckM** (Scores completeness and contamination of every bin to define high-quality bins per project)
- **GUNC** (Flags chimeric bins so high-quality classification accounts for chimerism)
- **strobealign** (Optional all-by-all cross-sample abundance estimation via strobealign --aemb (version 0.13 or newer)) — https://github.com/ksahlin/strobealign
- **samtools** (External dependency used in the mapping step to produce sorted BAM files from reads mapped to contigs) — https://github.com/samtools/samtools

## Examples

```
SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : --self-supervised -o SemiBin2
```

## Evaluation signals

- Reproduced high-quality bin counts per project fall at/near the reported SemiBin2 values: 1678 (human gut), 3776 (dog gut), 631 (ocean), 254 (soil)
- SemiBin2's percentage gains land within the reported ranges: 8.3–21.5% over SemiBin1 and 27.3–229.9% over VAMB
- Wilcoxon signed-rank P-values on paired per-sample quality scores indicate statistically significant improvement
- Every bin from all three binners was assessed by both CheckM (completeness/contamination) and GUNC (chimerism) before classification
- Concatenated contig headers contain the sample name followed by a separator character, and all sample BAM files were supplied to multi_easy_bin

## Limitations

- The SemiBin1 semi-supervised baseline requires the extra generate_cannot_links step (mmseqs2 against GTDB), which is why SemiBin2's self-supervised mode needs only 25% of the running time and 11% of the peak memory
- SemiBin operations are probabilistic — running the workflow twice can return different bins, so exact count reproduction is not guaranteed
- The SemiBin_tutorial repository ships a toy dataset that should not be used to draw biological conclusions; real benchmarking requires the full 82–129-sample projects

## Evidence

- [methods] multi_easy_bin combined inputs: "The command `multi_easy_bin` requires the combined contig file from several samples, BAM files (reads mapping to the combined contig) as inputs"
- [methods] concatenate_fasta: [evidence span withheld — anchor status: not_found]
- [methods] all-by-all abundance: "You will need to generate abundance files for each sample in an _all-by-all_ manner.
This means that each sample will be mapped to the contigs of the other samples."
- [abstract] headline gains vs SemiBin1: "Compared to SemiBin1, SemiBin2 can reconstruct 8.3-21.5% more high-quality bins and requires only 25% of the running time and 11% of peak memory usage in real short-read sequencing samples."
- [readme] benchmark invocation: [evidence span withheld — anchor status: external_doc]
- [readme] probabilistic reproducibility caveat: [evidence span withheld — anchor status: external_doc]
