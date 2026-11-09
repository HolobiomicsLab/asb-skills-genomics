---
name: nanopore-modified-basecalling
description: Use when raw Nanopore fast5 single-read files must be converted into
  per-read or per-site CpG 5mC methylation calls, or when comparing methylation-calling
  tools against a WGBS/truth reference.
when_to_use_negative:
- Input is already per-site methylation frequency data (e.g., WGBS CpG tables) — this
  skill calls methylation from raw fast5 signal, not from bisulfite or pre-aggregated
  data.
- Running times without GPU support are prohibitive — combining multiple tools (e.g.,
  Megalodon plus DeepMod) may be impractical for routine analyses, per the discussion.
- Targeting RNA modifications or non-CpG (e.g., 6mA) contexts without adapting the
  tool configs — the pipelines here target 5-methylcytosine at CG dinucleotides specifically.
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: METEORE
  role: Snakemake pipelines wrapping the six tools and providing the random-forest
    and multiple-linear-regression consensus models that combine per-read predictions
  repo: https://github.com/comprna/METEORE
- name: Snakemake
  role: Workflow engine executing the per-tool Snakefiles for reproducible methylation
    calling
  repo: https://github.com/comprna/METEORE
- name: Nanopolish
  role: Methylation-calling tool run via the Nanopolish Snakefile on single-read fast5
  repo: https://github.com/comprna/METEORE
- name: DeepSignal
  role: Deep-learning methylation caller run via the Deepsignal1 Snakefile; also one
    of the two inputs to the best-performing METEORE consensus
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Tombo
  role: Modified base prediction via the Tombo Snakefile after annotating raw fast5
    with fastq basecalls
  repo: https://github.com/comprna/METEORE
- name: Guppy
  role: Modified basecalling from signal data using the dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg
    config via the Guppy Snakefile
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Single-command basecalling, mapping and modified-base detection with Guppy
    or Remora backend; the best-performing individual tool (GPU-recommended)
  repo: https://github.com/nanoporetech/megalodon
- name: DeepMod
  role: LSTM/RNN-based per-read and per-site modification detection; one of the six
    benchmarked tools
  repo: https://github.com/WGLab/DeepMod
- name: ont_fast5_api
  role: Provides multi_to_single_fast5 to convert multi-read fast5 into the single-read
    format required by the pipelines
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Remora
  role: Modified base model backend for Megalodon modified-base detection
  repo: https://github.com/nanoporetech/remora
- name: gcf52ref
  role: Scripts converting guppy methylation calls from fast5 files to reference-anchored
    calls
  repo: https://github.com/kpalin/gcf52ref
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:e5ca16f5016d620cb8f1116ebc1cbed5a901fa021b9321dbc67fbd0fd123a0c9
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
  - gcf52ref
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# nanopore-modified-basecalling

## Summary

Detect 5-methylcytosine at CpG sites from Oxford Nanopore fast5 reads using the six benchmarked methylation-calling tools (Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, DeepMod) orchestrated via METEORE Snakemake pipelines, and optionally combine their per-read predictions into a higher-accuracy METEORE consensus.

## When to use

Use when raw Nanopore fast5 single-read files must be converted into per-read or per-site CpG 5mC methylation calls, or when comparing methylation-calling tools against a WGBS/truth reference.

## When NOT to use

- Input is already per-site methylation frequency data (e.g., WGBS CpG tables) — this skill calls methylation from raw fast5 signal, not from bisulfite or pre-aggregated data.
- Running times without GPU support are prohibitive — combining multiple tools (e.g., Megalodon plus DeepMod) may be impractical for routine analyses, per the discussion.
- Targeting RNA modifications or non-CpG (e.g., 6mA) contexts without adapting the tool configs — the pipelines here target 5-methylcytosine at CG dinucleotides specifically.

## Inputs

- Single-read Nanopore fast5 files (converted from multi-read fast5 via ont_fast5_api)
- Reference genome in .fasta format (placed in data/ and re-defined in the Snakefile)
- Basecalled fastq files (for Tombo raw-with-fastq annotation)
- Per-read methylation score files with ID, Chr, Pos, Strand, Score columns (for METEORE consensus)
- DeepSignal trained model CpG.R9.4_1D.human_hx1.bn17.sn360.v0.1.7+

## Outputs

- Per-read methylation score files (ID, Chr, Pos, Strand, Score) per tool
- Per-site methylation frequency tables in augmented BED format (chrom, start, end, coverage, methylation frequency, strandedness)
- Combined-strand per-site files (chrom, start, end, summed coverage, averaged methylation frequency)
- METEORE consensus per-read predictions (random forest or regression) and per-site frequencies at 0.5 threshold

## How to apply

First convert multi-read fast5 to single-read fast5 with ont_fast5_api's multi_to_single_fast5, and for Tombo annotate raw fast5 with basecalls via 'tombo preprocess annotate_raw_with_fastqs'. Then run each tool through its dedicated Snakefile (Nanopolish, Deepsignal1, Tombo, Guppy) or a single Megalodon command with Guppy/Remora backend using the dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg Guppy config. Produce per-read score files (ID, Chr, Pos, Strand, Score) per tool, derive per-site methylation frequency tables, and merge both strands per CpG by averaging methylation frequencies and summing coverage. For higher accuracy, combine per-read scores from two or more tools with METEORE: combination_model_prediction.py (random forest, with Megalodon + DeepSignal as the best balance) or meteore_reg.py (multiple linear regression with trunc_min/trunc_max score ranges, default --filter, and MinMax scaling), converting predictions to per-site frequencies with prediction_to_mod_frequency.py at the default 0.5 threshold. Rationale: the benchmark found individual tools trade off false positives against false negatives (e.g., Guppy underpredicts methylated sites; Nanopolish and Tombo have high false positives at unmethylated sites), while the METEORE consensus improves accuracy over individual tools.

## Related tools

- **METEORE** (Snakemake pipelines wrapping the six tools and providing the random-forest and multiple-linear-regression consensus models that combine per-read predictions) — https://github.com/comprna/METEORE
- **Snakemake** (Workflow engine executing the per-tool Snakefiles for reproducible methylation calling) — https://github.com/comprna/METEORE
- **Nanopolish** (Methylation-calling tool run via the Nanopolish Snakefile on single-read fast5) — https://github.com/comprna/METEORE
- **DeepSignal** (Deep-learning methylation caller run via the Deepsignal1 Snakefile; also one of the two inputs to the best-performing METEORE consensus) — https://github.com/bioinfomaticsCSU/deepsignal
- **Tombo** (Modified base prediction via the Tombo Snakefile after annotating raw fast5 with fastq basecalls) — https://github.com/comprna/METEORE
- **Guppy** (Modified basecalling from signal data using the dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg config via the Guppy Snakefile) — https://github.com/comprna/METEORE
- **Megalodon** (Single-command basecalling, mapping and modified-base detection with Guppy or Remora backend; the best-performing individual tool (GPU-recommended)) — https://github.com/nanoporetech/megalodon
- **DeepMod** (LSTM/RNN-based per-read and per-site modification detection; one of the six benchmarked tools) — https://github.com/WGLab/DeepMod
- **ont_fast5_api** (Provides multi_to_single_fast5 to convert multi-read fast5 into the single-read format required by the pipelines) — https://github.com/nanoporetech/ont_fast5_api
- **Remora** (Modified base model backend for Megalodon modified-base detection) — https://github.com/nanoporetech/remora
- **gcf52ref** (Scripts converting guppy methylation calls from fast5 files to reference-anchored calls) — https://github.com/kpalin/gcf52ref

## Examples

```
mamba create -c conda-forge -c bioconda -n meteore_nanopolish_env snakemake; conda activate meteore_nanopolish_env; multi_to_single_fast5 -i multi/ -s single/; snakemake --cores 8
```

## Evaluation signals

- Each tool's per-read output must follow the five-column schema: ID, Chr, Pos, Strand, Score.
- Per-site combined-strand files must contain chromosome, start, end, summed coverage, and averaged methylation frequency per CpG.
- Per-site methylation frequencies must lie in [0, 1]; per-read scores respect each tool's score range (e.g., trunc_min/trunc_max of -5/5 and -50/50 for the regression).
- METEORE consensus predictions should use the default 0.5 threshold via prediction_to_mod_frequency.py, with default --filter and MinMax scaling unless explicitly disabled by --no_scaling.
- Against a WGBS truth reference, the METEORE regression consensus (Megalodon + DeepSignal inputs) should attain the highest Pearson r and lowest RMSE — in the published Table 1 comparison, r = 0.8885 and RMSE = 0.1607.

## Limitations

- Tools exhibit a tradeoff between false positives and false negatives and high dispersion against expected methylation frequencies; no single method predicts correctly across all ranges of methylation frequency.
- Guppy correctly identified unmethylated sites but failed at fully methylated sites; Nanopolish and Tombo recover fully methylated sites but show high false positives at unmethylated sites; DeepMod and Guppy tend to underpredict.
- Megalodon gives the best overall performance but relies on GPUs to be time efficient; combining multiple tools may be impractical without GPU support due to running times.
- Discrepancies with WGBS could stem from detection differences or disparate model types and training regimes of the tools; the M.SssI methylated control only has ~95% efficiency, which may depress agreement with expected frequencies.
- An advantage exists in removing sites with uncertain methylation status; Megalodon is deprecated in favor of Dorado/Remora/modkit per its README.

## Evidence

- [abstract] Tradeoff between false positives and false negatives: "We found that tools have a tradeoff between false positives and false negatives"
- [abstract] Consensus improves accuracy: "a consensus approach, METEORE ( https://github.com/comprna/METEORE ), based on the combination of the predictions from two or more tools"
- [discussion] Guppy failure at methylated sites: "Extreme cases were Guppy, which correctly identified unmethylated sites but failed at fully methylated sites"
- [discussion] Megalodon best performance: "Megalodon provided overall the best performance but relies on GPUs to be time efficient."
- [discussion] Best consensus balance: "The combination of Megalodon and DeepSignal struck a good balance for accuracy and running times but would require a GPU for efficiency."
- [readme] fast5 conversion: "convert the fast5 files to single-read fast5 format"
- [methods] Guppy config: "dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg"
- [methods] METEORE regression options: "--trunc_min and trunc_max parameters can be used to restrict the useful range of log difference scores on a per-tool basis"
- [methods] Megalodon single command: "it can basecall, map, detect base modifications with Guppy (GPU on Linux) and Remora running backend, using a single command"
- [discussion] Uncertain sites removal: "Furthermore, there was an advantage in removing sites with uncertain methylation status."
