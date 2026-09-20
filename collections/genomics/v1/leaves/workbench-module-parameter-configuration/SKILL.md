---
name: workbench-module-parameter-configuration
description: 'Use when setting up or retuning a miRPursuit run: before launching miRPursuit.sh,
  when memory requires a split genome for miRCat, or when re-running the pipeline
  from a specific step with different filter or miRCat thresholds.'
when_to_use_negative:
- No reference genome is available for the organism — PatMaN mapping, miRCat precursor
  search and PAREsnip all require a supplied genome, so parameter configuration alone
  cannot make the pipeline run
- Input is already a processed expression/count table — the skill configures upstream
  read-processing modules, not downstream differential-expression analysis
- Quality control of raw reads is the goal — fastqc is not yet implemented in miRPursuit,
  so the cfg files offer no QC parameters
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: miRPursuit
  role: Shell-script pipeline whose config-directory .cfg files hold all module parameters;
    parameters can be changed without editing sRNAworkFlow.sh
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: miRCat
  role: UEA sRNA workbench novel-miRNA predictor whose thresholds (min_energy, min_paired,
    max_genome_hits, min_length, max_length, max_unpaired, pval) are set in wbench_mircat.cfg
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: PatMaN
  role: Short-read pattern matcher used for 0-mismatch genome mapping; its genome-filtering
    parameters live in patman_genome.cfg
- name: PAREsnip
  role: Degradome target-prediction module whose parameters are stored in wbench_paresnip.cfg
- name: FASTX-Toolkit
  role: fastq_to_fasta and fastx_clipper used in Step 1 adaptor trimming and format
    conversion, controlled by the filter cfg / ADAPTOR variable in workdirs.cfg
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:e73278b9b73c99fdf3c7b8ae3bf857eda448d7098f4eeda8821ae1b329557e57
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/forestbiotech-lab/miRPursuit
  tools:
  - FASTX-Toolkit
  - Java
  - PAREsnip
  - PatMaN
  - miRCat
  - miRPursuit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# workbench-module-parameter-configuration

## Summary

Configure the UEA sRNA workbench modules used inside miRPursuit via the config-directory .cfg files (workdirs.cfg, software_dirs.cfg, patman_genome.cfg, wbench_mircat.cfg, wbench_tasi.cfg), so each pipeline stage (Wbench Filter, PatMaN mapping, TaSi, miRCat, PAREsnip) runs with explicit, reproducible parameters.

## When to use

Use when setting up or retuning a miRPursuit run: before launching miRPursuit.sh, when memory requires a split genome for miRCat, or when re-running the pipeline from a specific step with different filter or miRCat thresholds.

## When NOT to use

- No reference genome is available for the organism — PatMaN mapping, miRCat precursor search and PAREsnip all require a supplied genome, so parameter configuration alone cannot make the pipeline run
- Input is already a processed expression/count table — the skill configures upstream read-processing modules, not downstream differential-expression analysis
- Quality control of raw reads is the goal — fastqc is not yet implemented in miRPursuit, so the cfg files offer no QC parameters

## Inputs

- miRPursuit config/ directory .cfg files (workdirs.cfg, software_dirs.cfg, patman_genome.cfg, wbench_mircat.cfg, wbench_tasi.cfg, wbench_paresnip.cfg)
- Reference genome FASTA (e.g. Arabidopsis thaliana TAIR10 chromosome 4 from testDataset/Genome)
- miRBase database path (MIRBASE)
- sRNA library insert directory (INSERTS_DIR) with FASTQ/FASTA files

## Outputs

- Configured pipeline execution producing mirbase hits, predicted targets, predicted mRNA outputs
- [workdir]/logs files recording all used parameters (%y%m%d:%h%m%s-type.log or *.log.ok)
- [workdir]/counts space-separated read-count statistics files
- mircat_novel_mirnas.csv with surviving novel miRNA precursors and loci

## How to apply

Edit the files in the miRPursuit config/ directory rather than the shell scripts: workdirs.cfg sets workdir, genomes path, INSERTS_DIR, MIRBASE path, MEMORY (e.g. 10g/2000m), THREADS, FILTER_SUF, and GENOME_MIRCAT (set to ${GENOME/.fa/part-1.fa} only when RAM forces a split-genome miRCat run, otherwise to ${GENOME}). software_dirs.cfg holds paths to PatMaN, FASTX-Toolkit and the UEA sRNA workbench. wbench_mircat.cfg carries the miRCat thresholds (broad defaults: extend=100.0, min_energy=-25.0, min_paired=17, max_gaps=3, max_genome_hits=16, min_length=18, max_length=26, min_gc=20, max_unpaired=60, pval=0.05, min_abundance=1; plant-mode variant: min_length=20, max_length=22, max_unpaired=50, Thread_Count=20), while wbench_tasi.cfg, patman_genome.cfg and wbench_paresnip.cfg play the same role for their stages. Broad sRNA filter defaults (min_length=18, max_length=26, min_abundance=5, trrna=true, filter_low_comp=true) live in the filter cfg selected by FILTER_SUF. The rationale is high customizability with full transparency: parameters are stored per tool so re-analysis from any of the 5 steps (Step 1 Wbench Filter ... Step 5 PareSnip) starts with documented settings, and all used parameters are recorded in the run log files.

## Related tools

- **miRPursuit** (Shell-script pipeline whose config-directory .cfg files hold all module parameters; parameters can be changed without editing sRNAworkFlow.sh) — https://github.com/forestbiotech-lab/miRPursuit
- **miRCat** (UEA sRNA workbench novel-miRNA predictor whose thresholds (min_energy, min_paired, max_genome_hits, min_length, max_length, max_unpaired, pval) are set in wbench_mircat.cfg) — https://github.com/forestbiotech-lab/miRPursuit
- **PatMaN** (Short-read pattern matcher used for 0-mismatch genome mapping; its genome-filtering parameters live in patman_genome.cfg)
- **PAREsnip** (Degradome target-prediction module whose parameters are stored in wbench_paresnip.cfg)
- **FASTX-Toolkit** (fastq_to_fasta and fastx_clipper used in Step 1 adaptor trimming and format conversion, controlled by the filter cfg / ADAPTOR variable in workdirs.cfg)

## Examples

```
vim config/software_dirs.cfg  # then bash miRPursuit.sh -f 1 -l 2 --fasta test_dataset-
```

## Evaluation signals

- Each config file parses without unset variables: software paths in software_dirs.cfg point to installed programs (PatMaN, FASTX-Toolkit, UEA workbench ~3.2, Java ~1.7, perl 5.8) and workdirs.cfg paths exist
- GENOME_MIRCAT is set to ${GENOME} unless memory genuinely forces part files; split-genome runs must merge all part predictions and drop miRNAs with more than max_genome_hits=16 genome hits
- miRCat cfg values match the intended mode: broad defaults (min_length=18, max_length=26, max_unpaired=60) vs plant-mode variant (min_length=20, max_length=22, max_unpaired=50, Thread_Count=20)
- The test dataset runs end-to-end after configuration: bash miRPursuit.sh -f 1 -l 2 --fasta test_dataset- completes and fills [workdir]/counts and [workdir]/logs
- Log files (*.log.ok notation %y%m%d:%h%m%s-type.log) record all parameters actually used for the run

## Limitations

- Quality control with fastqc is not yet implemented, so no cfg parameter covers QC
- The count output is space-separated despite being treated as tsv-like, and report file generation / detailed reporting tables are still TODO
- The README notes GENOME_MIRCAT split parts may be necessary with short RAM, and RNAfold depends on /lib/ld-linux.so.2 (libc6-i386 on Ubuntu) — a common setup failure point

## Evidence

- [readme] wbench_mircat.cfg - General parameters for mircat: [evidence span withheld — anchor status: external_doc]
- [readme] GENOME_MIRCAT is the genome used by mircat and can be set to genome parts for low-RAM machines: [evidence span withheld — anchor status: external_doc]
