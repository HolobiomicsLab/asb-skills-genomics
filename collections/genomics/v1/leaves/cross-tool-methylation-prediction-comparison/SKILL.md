---
name: cross-tool-methylation-prediction-comparison
description: Use when nanopore fast5 data from a controlled methylation mixture (e.g.
  dataset 1 with 100 selected CpG sites) must be processed through multiple methylation
  callers and their per-site predictions compared quantitatively.
when_to_use_negative:
- Input data is individual-read accuracy assessment, Cas9-targeted (nCATS), or WGBS
  comparison rather than a controlled methylation mixture with known expected frequencies
- The goal is a consensus/combined prediction (METEORE random forest or regression
  model) rather than per-tool comparative benchmarking
- Fast5 files are already single-read and basecalled through a pipeline that outputs
  reference-anchored methylation calls (e.g. Dorado/modkit workflows); the R9.4.1-era
  Snakefiles here do not apply
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: ont_fast5_api
  role: Convert multi-read fast5 to single-read fast5 format as pipeline input
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Nanopolish
  role: Methylation caller run via its Snakemake Snakefile to produce per-site frequencies
    in augmented BED format
  repo: https://github.com/comprna/METEORE
- name: DeepSignal
  role: Deep-learning methylation caller run in two Conda environments via its Snakefile
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Tombo
  role: Modified base detection via Snakefile after tombo preprocess annotate_raw_with_fastqs
  repo: https://github.com/comprna/METEORE
- name: Guppy
  role: Modified basecalling with dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg config,
    followed by gcf52ref conversion to reference-anchored calls
  repo: https://github.com/kpalin/gcf52ref
- name: Megalodon
  role: Combined basecalling and modified base detection with Guppy and Remora backend
    on GPU
  repo: https://github.com/nanoporetech/megalodon
- name: Remora
  role: Remora model backend for Megalodon modified base detection
  repo: https://github.com/nanoporetech/remora
- name: DeepMod
  role: Deep-learning detection run with the trained conmodC model, with strand merging
    via run_deepmod.R
  repo: https://github.com/WGLab/DeepMod
- name: Snakemake
  role: Workflow manager orchestrating each tool's pipeline for reproducibility
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:f5e6c952c34ef114569d2591b98f460a315bacd9fe2f7a1e40e17130b6fa9f6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/ont_fast5_api
  tools:
  - DeepMod
  - DeepSignal
  - Guppy
  - METEORE
  - Megalodon
  - Nanopolish
  - Remora
  - Snakemake
  - Tombo
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# cross-tool-methylation-prediction-comparison

## Summary

Reproduce per-site benchmarking of six nanopore 5mC methylation-calling tools (Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, DeepMod) on controlled methylation mixtures by comparing predicted per-site CpG methylation frequencies against expected mixture frequencies using Pearson r, r^2, RMSE, and out-of-window proportions.

## When to use

Use when nanopore fast5 data from a controlled methylation mixture (e.g. dataset 1 with 100 selected CpG sites) must be processed through multiple methylation callers and their per-site predictions compared quantitatively.

## When NOT to use

- Input data is individual-read accuracy assessment, Cas9-targeted (nCATS), or WGBS comparison rather than a controlled methylation mixture with known expected frequencies
- The goal is a consensus/combined prediction (METEORE random forest or regression model) rather than per-tool comparative benchmarking
- Fast5 files are already single-read and basecalled through a pipeline that outputs reference-anchored methylation calls (e.g. Dorado/modkit workflows); the R9.4.1-era Snakefiles here do not apply

## Inputs

- Single-read fast5 files from nanopore sequencing of controlled methylation mixtures
- Reference genome in FASTA format (e.g. ecoli_k12_mg1655.fasta)
- FASTQ basecall files for annotating raw fast5 (Tombo)
- List of 100 selected CpG sites with expected methylation frequencies from the mixture set

## Outputs

- Per-site predicted methylation frequencies per tool in augmented BED format (freq-perCG files)
- Aggregated benchmark metrics table (methylation_tool_benchmark_metrics.tsv) with Pearson r, r^2, RMSE, and out-of-window site proportions per tool

## How to apply

First convert multi-read fast5 to single-read format with ont_fast5_api's multi_to_single_fast5 and annotate raw fast5 with fastq basecalls via tombo preprocess annotate_raw_with_fastqs. Run each tool under its own Conda/Snakemake environment as defined in METEORE: Nanopolish and Tombo via their Snakefiles, DeepSignal in two environments (Deepsignal1 with tensorflow 1.13.1 CPU, then Deepsignal2), Guppy with the dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg config plus the gcf52ref-based Snakefile, Megalodon v2.5.0 with Guppy v6.1.1 and Remora v1.0.0 (--remora-modified-bases dna_r9.4.1_e8 hac 0.0.0 5mc CG 0, --mod-motif m CG 0, GPU), and DeepMod with the trained conmodC model followed by Rscript script/run_deepmod.R to merge strands. Extract each tool's predicted methylation frequency for the 100 selected CpG sites from the freq-perCG (augmented BED) outputs, then compute Pearson r, r^2, RMSE, and the proportion of sites outside the expected +/-10% window against the expected mixture frequencies. The rationale is that the tools trade off false positives and false negatives, so aggregate metrics reveal which callers (e.g. Megalodon) achieve the highest correlation and lowest RMSE versus which (e.g. Guppy) show the greatest dispersion.

## Related tools

- **ont_fast5_api** (Convert multi-read fast5 to single-read fast5 format as pipeline input) — https://github.com/nanoporetech/ont_fast5_api
- **Nanopolish** (Methylation caller run via its Snakemake Snakefile to produce per-site frequencies in augmented BED format) — https://github.com/comprna/METEORE
- **DeepSignal** (Deep-learning methylation caller run in two Conda environments via its Snakefile) — https://github.com/bioinfomaticsCSU/deepsignal
- **Tombo** (Modified base detection via Snakefile after tombo preprocess annotate_raw_with_fastqs) — https://github.com/comprna/METEORE
- **Guppy** (Modified basecalling with dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg config, followed by gcf52ref conversion to reference-anchored calls) — https://github.com/kpalin/gcf52ref
- **Megalodon** (Combined basecalling and modified base detection with Guppy and Remora backend on GPU) — https://github.com/nanoporetech/megalodon
- **Remora** (Remora model backend for Megalodon modified base detection) — https://github.com/nanoporetech/remora
- **DeepMod** (Deep-learning detection run with the trained conmodC model, with strand merging via run_deepmod.R) — https://github.com/WGLab/DeepMod
- **Snakemake** (Workflow manager orchestrating each tool's pipeline for reproducibility) — https://github.com/comprna/METEORE

## Examples

```
snakemake -s Nanopolish data/example/example.freq-perCG.tsv --cores all
```

## Evaluation signals

- All tools except Tombo should achieve Pearson correlations above 0.8 against expected mixture frequencies (p < 2.2e-16), reproducing the Fig. 2a,b pattern
- Megalodon should attain the highest Pearson correlation and lowest RMSE, followed by DeepMod and DeepSignal
- Guppy should show the highest RMSE with most sites predicted outside the expected +/-10% window
- Each tool's per-site output must contain the augmented BED fields: chromosome, start, end, coverage, methylation frequency, and strandedness
- All metrics must be aggregated into a single methylation_tool_benchmark_metrics.tsv with one row (or column set) per tool

## Limitations

- Tools show a tradeoff between false positives and false negatives: Guppy correctly identified unmethylated sites but failed at fully methylated sites, while Nanopolish and Tombo recovered fully methylated sites but had high false-positive rates at unmethylated sites
- Predictions per site generally showed high dispersion and did not generally agree with expected methylation frequency; the M.SssI methylated control only has 95% efficiency, a possible reason for low agreement
- No single method predicts correctly across all ranges of methylation frequency; Megalodon performs best but relies on GPUs to be time efficient
- Tombo did not reach significance (correlation below the p < 2.2e-16 threshold achieved by other tools)

## Evidence

- [other] On mixture dataset 1, all tools except Tombo achieved Pearson correlations above 0.8 (p < 2.2e-16); Megalodon attained the highest correlation and lowest RMSE: [evidence span withheld — anchor status: not_found]
- [methods] Please use the `multi_to_single_fast5` command from the ont_fast5_api package: [evidence span withheld — anchor status: not_found]
- [methods] A Snakefile named `Nanopolish` contains all rules for the Snakemake workflow: "A Snakefile named `Nanopolish` contains all rules for the Snakemake workflow. Run the snakemake to create the output files"
- [discussion] Extreme cases were Guppy, which correctly identified unmethylated sites but failed at fully methylated sites: "Extreme cases were Guppy, which correctly identified unmethylated sites but failed at fully methylated sites"
- [abstract] We found that tools have a tradeoff between false positives and false negatives: "We found that tools have a tradeoff between false positives and false negatives and present a high dispersion with respect to the expected methylation frequency values"
- [readme] The first output file contains the following fields: 1. Reference chromosome 2. Start position in chromosome: "The first output file contains the following fields: 1. Reference chromosome 2. Start position in chromosome 3. End position in chromosome 4. Read coverage 5. Methylation (i.e. methylation frequency)"
