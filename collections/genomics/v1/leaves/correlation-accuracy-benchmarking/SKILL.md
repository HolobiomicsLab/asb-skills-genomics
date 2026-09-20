---
name: correlation-accuracy-benchmarking
description: Use when per-read or per-site 5mC predictions from nanopore sequencing
  need to be quantitatively compared against WGBS or other orthogonal methylation
  ground truth to rank tools or consensus models.
when_to_use_negative:
- Input is already a methylation frequency table with no underlying per-read nanopore
  predictions — the skill requires tool-level per-read scores to aggregate
- The task is single-read methylation calling for a new sample rather than benchmarking/ranking
  tools against an orthogonal WGBS reference
- Fast5 files are multi-read format — they must first be converted with multi_to_single_fast5
  from ont_fast5_api before the pipelines can run
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Nanopolish
  role: Per-read CpG methylation caller run via its METEORE Snakefile to generate
    score files
- name: DeepSignal
  role: Per-read CpG methylation caller; also one of the two tools combined by the
    METEORE consensus
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Tombo
  role: Modified-base prediction pipeline; requires fast5 annotation with fastq basecalls
    (tombo preprocess annotate_raw_with_fastqs)
- name: Guppy
  role: Modified basecaller using the dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg
    config
- name: Megalodon
  role: Single-command basecall, map, and modified-base detection with Guppy/Remora
    backend; best individual performer
  repo: https://github.com/nanoporetech/megalodon
- name: DeepMod
  role: Deep-learning per-read CpG methylation caller included in the benchmark
  repo: https://github.com/WGLab/DeepMod
- name: METEORE
  role: Snakemake pipelines plus random-forest and multiple-linear-regression consensus
    models whose accuracy is benchmarked here
  repo: https://github.com/comprna/METEORE
- name: Snakemake
  role: Workflow engine used to execute the per-tool methylation-calling pipelines
    reproducibly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:abaf91663d552aaf32cd4e52cca4e3c2cf815b9a5235e08d10f9e241afda39e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinfomaticsCSU/deepsignal
  tools:
  - DeepMod
  - DeepSignal
  - Guppy
  - METEORE
  - Megalodon
  - Nanopolish
  - Snakemake
  - Tombo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# correlation-accuracy-benchmarking

## Summary

Benchmark CpG methylation callers (Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, DeepMod) and METEORE consensus models by correlating per-site methylation frequencies against orthogonal ENCODE WGBS CpG frequencies, quantifying agreement with Pearson r, r^2, Spearman rho, and RMSE.

## When to use

Use when per-read or per-site 5mC predictions from nanopore sequencing need to be quantitatively compared against WGBS or other orthogonal methylation ground truth to rank tools or consensus models.

## When NOT to use

- Input is already a methylation frequency table with no underlying per-read nanopore predictions — the skill requires tool-level per-read scores to aggregate
- The task is single-read methylation calling for a new sample rather than benchmarking/ranking tools against an orthogonal WGBS reference
- Fast5 files are multi-read format — they must first be converted with multi_to_single_fast5 from ont_fast5_api before the pipelines can run

## Inputs

- nCATS (Cas9-targeted) NA12878 single-read fast5 files
- Reference genome FASTA
- ENCODE WGBS CpG methylation frequency files (ENCFF279HCL, ENCFF835NTC)
- Per-read score files (ID, Chr, Pos, Strand, Score) from Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, DeepMod
- METEORE training data on figshare for the regression consensus (WGBS-anchored, with per-tool trunc_min/trunc_max ranges)

## Outputs

- Combined-strand per-site methylation frequency tables (augmented BED: chr, start, end, coverage, methylation frequency)
- Agreement metrics table (table1_agreement_metrics.csv) with N, Pearson r, r^2, Spearman rho, RMSE per method

## How to apply

Run each tool's methylation-calling pipeline (via the METEORE Snakemake Snakefiles for Nanopolish, DeepSignal, Tombo, and Guppy, and single commands for Megalodon and DeepMod) on the nCATS NA12878 single-read fast5 files, producing per-read score files with ID, Chr, Pos, Strand, Score columns. Aggregate per-read scores into per-site methylation frequencies, combining both strands by averaging methylation frequencies and summing read coverage per CpG (METEORE augmented BED format: chr, start, end, coverage, frequency). Optionally run the METEORE random-forest consensus (combination_model_prediction.py with -m default or -m optimized) or the regression consensus (meteore_reg.py with per-tool trunc_min/trunc_max score ranges such as -5/5 and -50/50, default --filter, and MinMax scaling), then convert predictions to per-site frequencies with prediction_to_mod_frequency.py at the 0.5 default threshold. Match each method's combined-strand CpG frequencies to the ENCODE WGBS (ENCFF279HCL, ENCFF835NTC) frequencies and compute N, Pearson r, r^2, Spearman rho, and RMSE per method, assembling one row per method (e.g. table1_agreement_metrics.csv).

## Related tools

- **Nanopolish** (Per-read CpG methylation caller run via its METEORE Snakefile to generate score files)
- **DeepSignal** (Per-read CpG methylation caller; also one of the two tools combined by the METEORE consensus) — https://github.com/bioinfomaticsCSU/deepsignal
- **Tombo** (Modified-base prediction pipeline; requires fast5 annotation with fastq basecalls (tombo preprocess annotate_raw_with_fastqs))
- **Guppy** (Modified basecaller using the dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg config)
- **Megalodon** (Single-command basecall, map, and modified-base detection with Guppy/Remora backend; best individual performer) — https://github.com/nanoporetech/megalodon
- **DeepMod** (Deep-learning per-read CpG methylation caller included in the benchmark) — https://github.com/WGLab/DeepMod
- **METEORE** (Snakemake pipelines plus random-forest and multiple-linear-regression consensus models whose accuracy is benchmarked here) — https://github.com/comprna/METEORE
- **Snakemake** (Workflow engine used to execute the per-tool methylation-calling pipelines reproducibly)

## Examples

```
python combination_model_prediction.py --input megalodon_scores.txt deepsignal_scores.txt -m default
```

## Evaluation signals

- METEORE (REG) combining Megalodon and DeepSignal attains the highest Pearson correlation (r = 0.8885) and lowest RMSE (0.1607) against WGBS among all methods
- Every method shows positive Pearson correlation with WGBS signals when predictions from both strands are combined
- Each per-site table has both-strand CpGs merged by averaged methylation frequency and summed coverage, matching the METEORE augmented BED schema (chr, start, end, coverage, frequency)
- All methods report a matched site count N against the ENCODE WGBS (ENCFF279HCL, ENCFF835NTC) CpG set
- Metrics file contains exactly one row per method with non-null Pearson r, r^2, Spearman rho, and RMSE

## Limitations

- Megalodon provided overall the best individual performance but relies on GPUs to be time efficient, and combining multiple tools may be impractical for routine analyses due to running times of methods without GPU support
- No single method predicts correctly across all ranges of methylation frequency; Guppy failed at fully methylated sites while Nanopolish and Tombo had high false positives at unmethylated sites
- Discrepancies with WGBS could stem from detection differences between nanopore and WGBS or disparate model types and training regimes of the tools

## Evidence

- [methods] We provide an example dataset `data/example` along with a genome reference `data/ecoli_k12_mg1655.fasta`: "We provide an example dataset `data/example` along with a genome reference `data/ecoli_k12_mg1655.fasta`"
- [readme] Read coverage: "we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage"
- [discussion] Megalodon best: "Megalodon provided overall the best performance but relies on GPUs to be time efficient."
- [abstract] Consensus improves accuracy: "based on the combination of the predictions from two or more tools that shows improved accuracy over individual tools"
- [methods] DeepSignal Snakefile: "A Snakefile named `Deepsignal1` contains rules to generate default output files from DeepSignal in the Snakemake workflow"
