---
name: snakemake-pipeline-orchestration
description: Use when a nanopore fast5 dataset must be processed end-to-end through
  multiple methylation-calling tools with reproducible, dependency-tracked Snakemake
  workflows (Snakefiles per tool) on Linux.
when_to_use_negative:
- Input is already per-site methylation-frequency output (augmented BED) — Snakemake
  methylation calling would be redundant.
- Analysis requires only a single tool run without reproducibility needs — a direct
  tool command (e.g. Megalodon single command) suffices.
- Working on non-Linux hardware without GPU access — Megalodon and DeepSignal pipelines
  rely on GPUs for time efficiency.
tools:
- name: Snakemake
  role: Workflow engine executing per-tool Snakefiles for reproducible methylation
    calling
  repo: https://github.com/comprna/METEORE
- name: ont_fast5_api
  role: multi_to_single_fast5 preprocessing of input fast5 files
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Nanopolish
  role: Methylation calling via its Snakemake Snakefile producing per-CpG frequencies
  repo: https://github.com/comprna/METEORE
- name: DeepSignal
  role: Methylation calling run across two Conda environments via the Deepsignal1/Deepsignal2
    Snakefiles
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Tombo
  role: Modified base prediction via its Snakefile after fast5 basecall annotation
  repo: https://github.com/comprna/METEORE
- name: Guppy
  role: Modified basecalling with dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg,
    post-processed via gcf52ref Snakefile
  repo: https://github.com/kpalin/gcf52ref
- name: Megalodon
  role: Modified base detection with Guppy v6.1.1 and Remora v1.0.0 backend in a single
    command
  repo: https://github.com/nanoporetech/megalodon
- name: Remora
  role: Remora modified-base model backend for Megalodon (--remora-modified-bases
    dna_r9.4.1_e8 hac 0.0.0 5mc CG 0)
  repo: https://github.com/nanoporetech/remora
- name: DeepMod
  role: Deep-learning methylation detection (bin/DeepMod.py detect with conmodC model,
    merged per strand by run_deepmod.R)
  repo: https://github.com/WGLab/DeepMod
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:0270a80c323cd9f5942b92ef5ded5ed9f344579613fb284a117837c51e8b8414
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 21
---

# snakemake-pipeline-orchestration

## Summary

Orchestrate nanopore CpG methylation-calling (Nanopolish, DeepSignal, Tombo, Guppy, Megalodon, DeepMod) via the METEORE Snakemake pipelines, converting multi-read fast5 inputs to per-site methylation-frequency outputs in augmented BED format.

## When to use

Use when a nanopore fast5 dataset must be processed end-to-end through multiple methylation-calling tools with reproducible, dependency-tracked Snakemake workflows (Snakefiles per tool) on Linux.

## When NOT to use

- Input is already per-site methylation-frequency output (augmented BED) — Snakemake methylation calling would be redundant.
- Analysis requires only a single tool run without reproducibility needs — a direct tool command (e.g. Megalodon single command) suffices.
- Working on non-Linux hardware without GPU access — Megalodon and DeepSignal pipelines rely on GPUs for time efficiency.

## Inputs

- Multi-read Oxford Nanopore fast5 files (converted to single-read fast5 via multi_to_single_fast5)
- Reference genome FASTA (e.g. data/ecoli_k12_mg1655.fasta)
- FASTQ basecalls for annotating raw fast5 (tombo preprocess annotate_raw_with_fastqs)
- Guppy config dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg and gcf52ref scripts

## Outputs

- Per-site methylation-frequency files in augmented BED format (chrom, start, end, coverage, methylation frequency, strandedness) per tool
- Combined-strand per-site files averaging methylation frequencies and summing coverage per CpG
- Per-read score files (ID, Chr, Pos, Strand, Score) usable as METEORE combined-model input

## How to apply

Clone the METEORE repository and install dependencies via Conda/Mamba, creating one dedicated environment per tool pipeline (e.g. meteore_nanopolish_env with snakemake; DeepSignal needs two environments, Deepsignal1 with tensorflow 1.13.1 CPU and Deepsignal2; Tombo uses snakemake==5.3.0, python 3.6, wiggelen, h5py 2.10.0). Convert multi-read fast5 to single-read fast5 with ont_fast5_api's multi_to_single_fast5, and for Tombo add basecalls with tombo preprocess annotate_raw_with_fastqs. Place the reference genome in data/ and redefine the reference filename inside the Snakefiles (Nanopolish, Deepsignal1, Tombo, Guppy); the fast5 folder name (e.g. 'example') selects target outputs. Run each Snakefile with snakemake targeting the per-tool freq-perCG outputs (e.g. snakemake -s Nanopolish ...freq-perCG.tsv --cores all); Guppy modified basecalling uses the dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg config plus the gcf52ref-based Snakefile, Megalodon v2.5.0 runs with Guppy v6.1.1 and Remora v1.0.0 on GPU, and DeepMod runs bin/DeepMod.py detect followed by Rscript script/run_deepmod.R. Outputs are per-site files in augmented BED format (chromosome, start, end, coverage, methylation frequency, strandedness), with an optional combined-strand file averaging methylation frequencies and summing coverage across CpG sites.

## Related tools

- **Snakemake** (Workflow engine executing per-tool Snakefiles for reproducible methylation calling) — https://github.com/comprna/METEORE
- **ont_fast5_api** (multi_to_single_fast5 preprocessing of input fast5 files) — https://github.com/nanoporetech/ont_fast5_api
- **Nanopolish** (Methylation calling via its Snakemake Snakefile producing per-CpG frequencies) — https://github.com/comprna/METEORE
- **DeepSignal** (Methylation calling run across two Conda environments via the Deepsignal1/Deepsignal2 Snakefiles) — https://github.com/bioinfomaticsCSU/deepsignal
- **Tombo** (Modified base prediction via its Snakefile after fast5 basecall annotation) — https://github.com/comprna/METEORE
- **Guppy** (Modified basecalling with dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg, post-processed via gcf52ref Snakefile) — https://github.com/kpalin/gcf52ref
- **Megalodon** (Modified base detection with Guppy v6.1.1 and Remora v1.0.0 backend in a single command) — https://github.com/nanoporetech/megalodon
- **Remora** (Remora modified-base model backend for Megalodon (--remora-modified-bases dna_r9.4.1_e8 hac 0.0.0 5mc CG 0)) — https://github.com/nanoporetech/remora
- **DeepMod** (Deep-learning methylation detection (bin/DeepMod.py detect with conmodC model, merged per strand by run_deepmod.R)) — https://github.com/WGLab/DeepMod

## Examples

```
snakemake -s Nanopolish --cores all
```

## Evaluation signals

- Each Snakefile produces its expected freq-perCG/augmented-BED output named with the fast5 folder name (e.g. 'example'); rerunning with --cores all is deterministic.
- Output columns match the documented augmented BED schema: chromosome, start, end, coverage, methylation frequency, strandedness (6 fields), and 5 fields for the combined-strand file.
- Combined-strand file CpG coverage equals the sum of per-strand coverages and methylation frequency equals the per-strand average.
- On the example dataset (50 single-read fast5 E.coli positive-control files, Simpson et al. 2017), all pipelines complete without errors against data/ecoli_k12_mg1655.fasta.
- Per-read score files contain the required ID, Chr, Pos, Strand, Score columns accepted by METEORE's combined models.

## Limitations

- Megalodon provided the best performance but relies on GPUs to be time efficient; combining multiple tools may be impractical for routine analyses due to running times of methods without GPU support.
- Tombo and Guppy showed local discrepancies with the other tools; Guppy correctly identified unmethylated sites but failed at fully methylated sites.
- The fast5 folder name must match the target output name in each Snakefile, and the reference genome must be redefined inside Nanopolish, Deepsignal1, Tombo and Guppy Snakefiles when using a new genome.

## Evidence

- [abstract] Snakemake pipelines are also provided for reproducibility and to enable the systematic application of our analyses to other datasets.: "Snakemake pipelines are also provided for reproducibility and to enable the systematic application of our analyses to other datasets."
- [methods] Please use the `multi_to_single_fast5` command from the ont_fast5_api package to convert the fast5 files to single-read fast5 format: [evidence span withheld — anchor status: not_found]
- [methods] A Snakefile named `Nanopolish` contains all rules for the Snakemake workflow. Run the snakemake to create the output files: "A Snakefile named `Nanopolish` contains all rules for the Snakemake workflow. Run the snakemake to create the output files"
- [methods] You can perform modified basecalling from the signal data using the `dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg` guppy config: "You can perform modified basecalling from the signal data using the `dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg` guppy config"
- [readme] We provide an example dataset `data/example` along with a genome reference `data/ecoli_k12_mg1655.fasta` for you to try the pipelines with.: "We provide an example dataset `data/example` along with a genome reference `data/ecoli_k12_mg1655.fasta` for you to try the pipelines with."
- [readme] In the second output file, we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage.: "In the second output file, we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage."
