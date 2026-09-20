---
name: nanopore-methylation-calling
description: Use when raw nanopore fast5 files must be converted to per-read 5mC CpG
  calls or per-site methylation frequencies, especially to benchmark or combine tool
  predictions on R9.4.1 flowcell data.
when_to_use_negative:
- Input is already per-read score TSVs or per-site BED files — go directly to consensus/evaluation
  rather than re-running callers
- Data are from flowcells/chemistries other than R9.4.1 (e.g. R10.4), where the cited
  configs, Remora model dna_r9.4.1_e8 hac 0.0.0, and trained models do not apply
- Production modified basecalling is the goal on current ONT hardware — Megalodon
  is deprecated in favor of Dorado/modkit per the tool README
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: ont_fast5_api
  role: Convert multi-read fast5 to single-read fast5 via multi_to_single_fast5 as
    preprocessing for all callers
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Nanopolish
  role: Per-site CpG methylation calling via Snakemake Snakefile producing freq-perCG.tsv
    in augmented BED format
  repo: https://github.com/comprna/METEORE
- name: DeepSignal
  role: Deep-learning per-read CpG methylation detection run in two Conda environments
    within the METEORE pipeline
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Tombo
  role: Modified base prediction via its Snakefile (snakemake 5.3.0, python 3.6, wiggelen,
    h5py 2.10.0); also used for fastq annotation of raw fast5
  repo: https://github.com/comprna/METEORE
- name: Guppy
  role: Modified basecalling from signal data with dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg,
    converted to reference-anchored calls with gcf52ref
  repo: https://github.com/kpalin/gcf52ref
- name: Megalodon
  role: Modified base detection with Guppy backend and Remora (v2.5.0 with Guppy v6.1.1,
    Remora v1.0.0, GPU); best overall performer in the benchmark
  repo: https://github.com/nanoporetech/megalodon
- name: Remora
  role: Modified base model backend used by Megalodon (--remora-modified-bases dna_r9.4.1_e8
    hac 0.0.0 5mc CG 0)
  repo: https://github.com/nanoporetech/remora
- name: DeepMod
  role: Deep-learning detection with the trained conmodC model (bin/DeepMod.py detect,
    10 threads) plus run_deepmod.R to merge strands
  repo: https://github.com/WGLab/DeepMod
- name: METEORE
  role: Snakemake pipelines wrapping all tools and random forest / multiple linear
    regression consensus models that combine per-read scores for higher accuracy
  repo: https://github.com/comprna/METEORE
- name: Snakemake
  role: Workflow engine executing per-tool Snakefiles for reproducible runs on example
    or user datasets
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:609cb2089245866c59212246698525cb9bbd74f2732074d3f22fefa7121c5191
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
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# nanopore-methylation-calling

## Summary

A benchmarked workflow for calling CpG 5-methylcytosine (5mC) from Oxford Nanopore fast5 files using six tools (Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, DeepMod), producing per-read scores and per-site methylation frequencies in augmented BED format for downstream accuracy assessment.

## When to use

Use when raw nanopore fast5 files must be converted to per-read 5mC CpG calls or per-site methylation frequencies, especially to benchmark or combine tool predictions on R9.4.1 flowcell data.

## When NOT to use

- Input is already per-read score TSVs or per-site BED files — go directly to consensus/evaluation rather than re-running callers
- Data are from flowcells/chemistries other than R9.4.1 (e.g. R10.4), where the cited configs, Remora model dna_r9.4.1_e8 hac 0.0.0, and trained models do not apply
- Production modified basecalling is the goal on current ONT hardware — Megalodon is deprecated in favor of Dorado/modkit per the tool README

## Inputs

- Multi-read Oxford Nanopore fast5 files (R9.4.1 flowcell)
- Reference genome in FASTA format (e.g. ecoli_k12_mg1655.fasta)
- Fastq basecalls for annotating raw fast5 (Tombo path)
- Guppy config dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg
- DeepSignal trained model CpG.R9.4_1D.human_hx1.bn17.sn360.v0.1.7+
- DeepMod trained conmodC model
- Per-read score TSVs (columns ID, Chr, Pos, Strand, Score) for consensus model input

## Outputs

- freq-perCG.tsv per-site methylation frequencies in augmented BED format (chromosome, start, end, coverage, methylation frequency, strandedness)
- Strand-merged per-site BED (averaged methylation frequencies, summed coverage)
- Per-read methylation score TSVs (ID, Chr, Pos, Strand, Score)
- METEORE consensus predictions from random forest or multiple linear regression models

## How to apply

First convert multi-read fast5 to single-read fast5 with ont_fast5_api's `multi_to_single_fast5`, and annotate raw fast5 with basecalls via `tombo preprocess annotate_raw_with_fastqs`. Then run each caller per its METEORE Snakemake pipeline: Nanopolish (Snakefile producing freq-perCG.tsv in augmented BED format), DeepSignal in two Conda environments (tensorflow 1.13.1 CPU, then conversion env), Tombo (snakemake==5.3.0, python 3.6, wiggelen, h5py 2.10.0), Guppy with the `dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg` config plus gcf52ref-based conversion, Megalodon v2.5.0 with Guppy v6.1.1 and Remora v1.0.0 on GPU, and DeepMod (`bin/DeepMod.py detect` with the conmodC model, 10 threads, then `Rscript script/run_deepmod.R` to merge strands). Extract per-site predicted methylation frequencies from each tool's freq-perCG output and, when combining, feed per-read score TSVs (ID, Chr, Pos, Strand, Score) into METEORE's random forest or multiple linear regression consensus models. Judge success by comparing per-site predictions to expected methylation frequencies of controlled mixtures via Pearson r, r^2, RMSE, and fraction of sites outside the expected +/-10% window.

## Related tools

- **ont_fast5_api** (Convert multi-read fast5 to single-read fast5 via multi_to_single_fast5 as preprocessing for all callers) — https://github.com/nanoporetech/ont_fast5_api
- **Nanopolish** (Per-site CpG methylation calling via Snakemake Snakefile producing freq-perCG.tsv in augmented BED format) — https://github.com/comprna/METEORE
- **DeepSignal** (Deep-learning per-read CpG methylation detection run in two Conda environments within the METEORE pipeline) — https://github.com/bioinfomaticsCSU/deepsignal
- **Tombo** (Modified base prediction via its Snakefile (snakemake 5.3.0, python 3.6, wiggelen, h5py 2.10.0); also used for fastq annotation of raw fast5) — https://github.com/comprna/METEORE
- **Guppy** (Modified basecalling from signal data with dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg, converted to reference-anchored calls with gcf52ref) — https://github.com/kpalin/gcf52ref
- **Megalodon** (Modified base detection with Guppy backend and Remora (v2.5.0 with Guppy v6.1.1, Remora v1.0.0, GPU); best overall performer in the benchmark) — https://github.com/nanoporetech/megalodon
- **Remora** (Modified base model backend used by Megalodon (--remora-modified-bases dna_r9.4.1_e8 hac 0.0.0 5mc CG 0)) — https://github.com/nanoporetech/remora
- **DeepMod** (Deep-learning detection with the trained conmodC model (bin/DeepMod.py detect, 10 threads) plus run_deepmod.R to merge strands) — https://github.com/WGLab/DeepMod
- **METEORE** (Snakemake pipelines wrapping all tools and random forest / multiple linear regression consensus models that combine per-read scores for higher accuracy) — https://github.com/comprna/METEORE
- **Snakemake** (Workflow engine executing per-tool Snakefiles for reproducible runs on example or user datasets) — https://github.com/comprna/METEORE

## Examples

```
megalodon fast5_dir/ --outputs mod_basecalls --guppy-config dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg --remora-modified-bases dna_r9.4.1_e8 hac 0.0.0 5mc CG 0 --mod-motif m CG 0 --device gpu
```

## Evaluation signals

- On controlled mixture dataset 1, per-site Pearson correlations should exceed 0.8 (p < 2.2e-16) for all tools except Tombo, with Megalodon highest and lowest RMSE
- Guppy should show the highest RMSE and the most sites predicted outside the expected +/-10% window
- For individual-read ROC/PR analysis on 0%/100% methylated sets (~2400 reads each), all five per-read tools should achieve AUC and AUCPR above 0.8 with Megalodon ranked highest
- Output files should be augmented BED files with fields: reference chromosome, start, end, read coverage, methylation frequency, and strandedness (strandedness dropped in the strand-merged file)
- Consensus (METEORE) predictions should improve accuracy over individual tools, reflecting the tradeoff where Guppy misses fully methylated sites while Nanopolish/Tombo show high false positives at unmethylated sites

## Limitations

- Tools exhibit a tradeoff between false positives and false negatives: Guppy correctly identified unmethylated sites but failed at fully methylated sites, while Nanopolish and Tombo recovered fully methylated sites but had a high rate of false positives at unmethylated sites
- No single method predicts correctly across all ranges of methylation frequency; per-site predictions show high dispersion and often disagree with expected methylation frequency
- The M.SssI methylated control has only 95% efficiency, a possible reason for low agreement with expected methylation frequency
- Megalodon provides the best performance but relies on GPUs to be time efficient; combining multiple tools may be impractical for routine analyses due to running times of methods without GPU support

## Evidence

- [discussion] Guppy correctly identified unmethylated sites but failed at fully methylated sites: "Extreme cases were Guppy, which correctly identified unmethylated sites but failed at fully methylated sites"
- [discussion] Megalodon best overall but GPU-dependent: "Megalodon provided overall the best performance but relies on GPUs to be time efficient."
- [methods] fast5 conversion preprocessing: [evidence span withheld — anchor status: not_found]
- [methods] Guppy config: "You can perform modified basecalling from the signal data using the `dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg` guppy config"
- [methods] Per-read score figshare deposit for consensus training/evaluation: "you can download them from figshare (https://figshare.com/articles/dataset/Per-read_scores_from_different_nanopore_methylation_tools_in_controlled_mixture_dataset_1/15081264)"
- [readme] Augmented BED output fields: "1. Reference chromosome
2. Start position in chromosome
3. End position in chromosome
4. Read coverage
5. Methylation (i.e. methylation frequency)
6. Strandedness"
- [methods] Megalodon single-command detection with Guppy/Remora backend: "it can basecall, map, detect base modifications with Guppy (GPU on Linux) and Remora running backend, using a single command"
