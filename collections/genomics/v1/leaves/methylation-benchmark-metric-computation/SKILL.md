---
name: methylation-benchmark-metric-computation
description: Use when nanopore methylation tool outputs (per-site methylation frequencies,
  e.g. freq-perCG augmented BED files) must be quantitatively benchmarked against
  known expected methylation frequencies from a control mixture dataset.
when_to_use_negative:
- Input is per-read score data that has not yet been aggregated to per-site methylation
  frequencies (use the tool-specific Snakemake pipelines or METEORE per-site prediction
  step first)
- The dataset lacks known expected methylation frequencies (e.g. WGBS-only or biological
  samples without a controlled mixture), so Pearson r and RMSE against expected values
  cannot be computed
- The goal is consensus prediction accuracy rather than individual tool benchmarking
  — use METEORE's random forest or multiple linear regression combined models instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3674
tools:
- name: Nanopolish
  role: Produces per-site CpG methylation frequencies via its Snakemake Snakefile
    (freq-perCG output in augmented BED format) for benchmarking
  repo: https://github.com/comprna/METEORE
- name: DeepSignal
  role: Methylation caller run in two Conda environments whose default outputs are
    converted to per-site frequencies for the metric comparison
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Tombo
  role: Modified-base prediction tool via its Snakefile (snakemake==5.3.0, python
    3.6, wiggelen, h5py 2.10.0) whose per-site predictions feed the benchmark metrics
  repo: https://github.com/comprna/METEORE
- name: Guppy
  role: Modified basecaller (dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg) whose
    calls are converted with gcf52ref scripts into per-site frequencies for benchmarking
  repo: https://github.com/kpalin/gcf52ref
- name: Megalodon
  role: Best-performing methylation detector (v2.5.0 with Guppy v6.1.1 and Remora
    v1.0.0, --remora-modified-bases dna_r9.4.1_e8 hac 0.0.0 5mc CG 0, --mod-motif
    m CG 0) whose per-site output anchors the benchmark
  repo: https://github.com/nanoporetech/megalodon
- name: DeepMod
  role: Deep-learning methylation detector (bin/DeepMod.py detect with trained conmodC
    model, 10 threads; merge strands via Rscript script/run_deepmod.R) included in
    the per-site comparison
  repo: https://github.com/WGLab/DeepMod
- name: ont_fast5_api
  role: Converts multi-read fast5 files to single-read fast5 (multi_to_single_fast5)
    as input preparation for all methylation callers
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Snakemake
  role: Workflow engine orchestrating each tool's pipeline for reproducible per-site
    methylation frequency generation
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:73cc4b868b80f076e15a4a2f9d7237d126fee806f0ba82fb7b4bcb010fcc7b7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# methylation-benchmark-metric-computation

## Summary

Computes per-site benchmarking metrics (Pearson r, r^2, RMSE, and fraction of sites outside an expected +/-10% window) comparing each nanopore methylation-calling tool's predicted CpG methylation frequencies against expected frequencies from a controlled methylation mixture, as in the METEORE paper's Fig. 2 analysis.

## When to use

Use when nanopore methylation tool outputs (per-site methylation frequencies, e.g. freq-perCG augmented BED files) must be quantitatively benchmarked against known expected methylation frequencies from a control mixture dataset.

## When NOT to use

- Input is per-read score data that has not yet been aggregated to per-site methylation frequencies (use the tool-specific Snakemake pipelines or METEORE per-site prediction step first)
- The dataset lacks known expected methylation frequencies (e.g. WGBS-only or biological samples without a controlled mixture), so Pearson r and RMSE against expected values cannot be computed
- The goal is consensus prediction accuracy rather than individual tool benchmarking — use METEORE's random forest or multiple linear regression combined models instead

## Inputs

- Per-site methylation frequency files in augmented BED format (freq-perCG) from Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, and DeepMod
- Expected methylation frequencies for the selected CpG sites of control mixture dataset 1
- List of selected CpG sites (e.g. 100 sites) with genomic coordinates

## Outputs

- Aggregated metrics table (methylation_tool_benchmark_metrics.tsv) with per-tool Pearson r, r^2, RMSE, correlation p-value, and proportion of sites outside the +/-10% expected window
- Tool ranking by correlation and RMSE suitable for reproducing Fig. 2a,b

## How to apply

Extract the per-site predicted methylation frequency for the selected CpG sites (e.g. 100 CpG sites of control mixture dataset 1) from each tool's freq-perCG output in augmented BED format (chromosome, start, end, coverage, methylation frequency, strandedness). Compare each tool's predictions with the expected mixture frequencies by computing Pearson correlation with significance, r^2, RMSE, and the proportion of sites falling outside the expected +/-10% window. Run tools with their default cutoffs so the comparison reflects out-of-the-box behavior, and aggregate all metrics into a single TSV (e.g. methylation_tool_benchmark_metrics.tsv) for ranking tools by correlation and dispersion. The rationale is that no single tool predicts correctly across all methylation frequency ranges, so correlation, RMSE, and out-of-window fraction together capture both accuracy and dispersion.

## Related tools

- **Nanopolish** (Produces per-site CpG methylation frequencies via its Snakemake Snakefile (freq-perCG output in augmented BED format) for benchmarking) — https://github.com/comprna/METEORE
- **DeepSignal** (Methylation caller run in two Conda environments whose default outputs are converted to per-site frequencies for the metric comparison) — https://github.com/bioinfomaticsCSU/deepsignal
- **Tombo** (Modified-base prediction tool via its Snakefile (snakemake==5.3.0, python 3.6, wiggelen, h5py 2.10.0) whose per-site predictions feed the benchmark metrics) — https://github.com/comprna/METEORE
- **Guppy** (Modified basecaller (dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg) whose calls are converted with gcf52ref scripts into per-site frequencies for benchmarking) — https://github.com/kpalin/gcf52ref
- **Megalodon** (Best-performing methylation detector (v2.5.0 with Guppy v6.1.1 and Remora v1.0.0, --remora-modified-bases dna_r9.4.1_e8 hac 0.0.0 5mc CG 0, --mod-motif m CG 0) whose per-site output anchors the benchmark) — https://github.com/nanoporetech/megalodon
- **DeepMod** (Deep-learning methylation detector (bin/DeepMod.py detect with trained conmodC model, 10 threads; merge strands via Rscript script/run_deepmod.R) included in the per-site comparison) — https://github.com/WGLab/DeepMod
- **ont_fast5_api** (Converts multi-read fast5 files to single-read fast5 (multi_to_single_fast5) as input preparation for all methylation callers) — https://github.com/nanoporetech/ont_fast5_api
- **Snakemake** (Workflow engine orchestrating each tool's pipeline for reproducible per-site methylation frequency generation) — https://github.com/comprna/METEORE

## Examples

```
snakemake -s Nanopolish example.freq-perCG.tsv --cores all  # then extract 100 CpG sites and compute Pearson r, r^2, RMSE vs expected frequencies
```

## Evaluation signals

- All tools except Tombo should achieve Pearson correlations above 0.8 against expected frequencies, with significance p < 2.2e-16
- Megalodon should attain the highest Pearson correlation and lowest RMSE, followed by DeepMod and DeepSignal
- Guppy should show the highest RMSE and the largest proportion of sites predicted outside the expected +/-10% window
- The aggregated metrics table reproduces the qualitative pattern of Fig. 2a,b when plotted
- Every tool's per-site output must be extracted for the same 100 selected CpG sites of mixture dataset 1 before computing metrics

## Limitations

- Predictions per site generally show high dispersion and do not generally agree with expected methylation frequency
- The M.SssI treatment used to generate the CpG-methylated control has only ~95% efficiency, a possible reason for low agreement with expected methylation frequencies
- Tools have a tradeoff between false positives and false negatives — e.g. Guppy correctly identified unmethylated sites but failed at fully methylated sites, while Nanopolish and Tombo recovered fully methylated sites but had high false-positive rates at unmethylated sites
- Megalodon provides the best performance but relies on GPUs to be time efficient; combining multiple tools may be impractical for routine analyses due to running times of methods without GPU support

## Evidence

- [methods] Per-site benchmark metrics against expected mixture frequencies: [evidence span withheld — anchor status: not_found]
- [methods] Metric definitions: [evidence span withheld — anchor status: not_found]
- [results] Expected result pattern: [evidence span withheld — anchor status: not_found]
- [discussion] Dispersion finding: "the predictions per site generally showed a high dispersion and did not generally agree with the expected methylation frequency"
- [readme] Output format: "The first output file contains the following fields: 1. Reference chromosome 2. Start position in chromosome 3. End position in chromosome 4. Read coverage 5. Methylation (i.e. methylation frequency)"
