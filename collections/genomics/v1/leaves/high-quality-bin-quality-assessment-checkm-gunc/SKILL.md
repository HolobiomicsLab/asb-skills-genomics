---
name: high-quality-bin-quality-assessment-checkm-gunc
description: 'Use when benchmarking metagenomic binners on real short-read projects:
  bins (FASTA) from SemiBin2, SemiBin1, and VAMB must be scored with CheckM and GUNC
  to count high-quality bins per project.'
when_to_use_negative:
- Input is raw reads or assembled contigs that have not yet been binned — this skill
  evaluates existing bins, not the binning step itself.
- The goal is taxonomic classification of contigs or MAGs rather than quality screening
  — use CAT/BAT (CAT_pack) for taxonomic annotation instead.
- The data is long-read sequencing — the reported high-quality-bin gains (8.3-21.5%
  vs SemiBin1) apply to real short-read samples; long-read results use the ensemble-based
  DBSCAN clustering comparison instead.
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CheckM
  role: Estimates completeness and contamination of every reconstructed bin to define
    high-quality bins per project
- name: GUNC
  role: Detects chimerism in bins as a second quality criterion for high-quality bin
    classification
- name: SemiBin2
  role: Produces the bins being assessed via multi_easy_bin in default self-supervised
    training mode
  repo: https://github.com/BigDataBiology/SemiBin/
- name: VAMB
  role: Comparison binner run on identical inputs so all three binners' bins are quality-assessed
    uniformly
- name: barrnap
  role: rRNA annotation used in the SemiBin2 tutorial's MAG quality workflow surrounding
    bin assessment
  repo: https://github.com/tseemann/barrnap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:c22e293b9a3c68257767262680540d1d98d1913aa6e98c08168b0ce54cd031b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - CheckM
  - GUNC
  - SemiBin1
  - SemiBin2
  - VAMB
  - barrnap
  - mmseqs2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# high-quality-bin-quality-assessment-checkm-gunc

## Summary

Assesses metagenome-assembled genomes reconstructed by binners such as SemiBin2, SemiBin1, and VAMB using CheckM for completeness/contamination and GUNC for chimerism, in order to classify high-quality bins and quantify per-project binner performance.

## When to use

Use when benchmarking metagenomic binners on real short-read projects: bins (FASTA) from SemiBin2, SemiBin1, and VAMB must be scored with CheckM and GUNC to count high-quality bins per project.

## When NOT to use

- Input is raw reads or assembled contigs that have not yet been binned — this skill evaluates existing bins, not the binning step itself.
- The goal is taxonomic classification of contigs or MAGs rather than quality screening — use CAT/BAT (CAT_pack) for taxonomic annotation instead.
- The data is long-read sequencing — the reported high-quality-bin gains (8.3-21.5% vs SemiBin1) apply to real short-read samples; long-read results use the ensemble-based DBSCAN clustering comparison instead.

## Inputs

- Reconstructed bin FASTA files from each binner (SemiBin2 multi_easy_bin, SemiBin1 semi-supervised mode, VAMB)
- Sorted BAM files / contig FASTA inputs that produced the bins (for traceability across the 82, 129, 109, and 101-sample projects)
- CheckM and GUNC reference databases

## Outputs

- Per-bin completeness, contamination (CheckM) and chimerism (GUNC) scores
- Counts of high-quality bins per project per binner
- Percentage gains of SemiBin2 over SemiBin1 (8.3-21.5%) and VAMB (27.3-229.9%)
- Wilcoxon signed-rank P-values comparing paired per-sample bin quality
- benchmark_results.csv with all reported values

## How to apply

Collect every reconstructed bin from each binner (e.g., SemiBin2's multi_easy_bin output, SemiBin1 semi-supervised output, VAMB output run on identical inputs). Run CheckM on each bin to obtain completeness and contamination estimates, and GUNC to detect chimerism (taxonomic homogeneity) levels. Apply the high-quality classification combining the completeness/contamination and chimerism criteria to count high-quality bins per project per binner. For the four real short-read projects (82, 129, 109, and 101 samples from human gut, dog gut, ocean, and soil), expected SemiBin2 counts are 1678, 3776, 631, and 254 high-quality bins. Use the counts to compute SemiBin2's percentage gains over SemiBin1 (expected 8.3-21.5%) and VAMB (expected 27.3-229.9%), and write all values to benchmark_results.csv. Validate statistical significance with the Wilcoxon signed-rank test on paired per-sample quality scores between binners.

## Related tools

- **CheckM** (Estimates completeness and contamination of every reconstructed bin to define high-quality bins per project)
- **GUNC** (Detects chimerism in bins as a second quality criterion for high-quality bin classification)
- **SemiBin2** (Produces the bins being assessed via multi_easy_bin in default self-supervised training mode) — https://github.com/BigDataBiology/SemiBin/
- **VAMB** (Comparison binner run on identical inputs so all three binners' bins are quality-assessed uniformly)
- **barrnap** (rRNA annotation used in the SemiBin2 tutorial's MAG quality workflow surrounding bin assessment) — https://github.com/tseemann/barrnap

## Examples

```
SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : --self-supervised -o SemiBin2
```

## Evaluation signals

- Per-project SemiBin2 high-quality bin counts match the reported values 1678, 3776, 631, and 254.
- SemiBin2's percentage gains fall within the reported 8.3-21.5% range vs SemiBin1 and 27.3-229.9% range vs VAMB.
- Every bin receives both CheckM (completeness/contamination) and GUNC (chimerism) scores before classification — a bin classified high-quality passed both screens.
- Wilcoxon signed-rank P-values on paired per-sample quality scores indicate statistically significant improvement of SemiBin2.
- All values are written to benchmark_results.csv.

## Limitations

- High-quality bin counts depend on the combination of CheckM completeness/contamination and GUNC chimerism thresholds; different classification criteria will change the counts and the percentage-gain comparisons.
- SemiBin's operations are probabilistic — re-running the binning that feeds this assessment can return different bins, so exact count reproduction is not guaranteed (per the SemiBin2 tutorial README).
- The reported gains were measured in real short-read sequencing samples; they do not directly transfer to long-read comparisons, which used ensemble-based DBSCAN clustering metrics (13.1-26.3% vs second-best binner).

## Evidence

- [abstract] Compared to SemiBin1, SemiBin2 can reconstruct 8.3-21.5% more high-quality bins and requires only 25% of the running time and 11% of peak memory usage in real short-read sequencing samples.: "Compared to SemiBin1, SemiBin2 can reconstruct 8.3-21.5% more high-quality bins and requires only 25% of the running time and 11% of peak memory usage in real short-read sequencing samples."
- [methods] multi_easy_bin requires combined contig file and BAM inputs: "The command `multi_easy_bin` requires the combined contig file from several samples, BAM files (reads mapping to the combined contig) as inputs"
- [readme] Benchmark README shows SemiBin command producing the bins assessed: [evidence span withheld — anchor status: external_doc]
