---
name: snakemake-pipeline-execution
description: Use when raw nanopore FASTQ/FASTA files from human samples need end-to-end
  transposable element insertion/deletion detection, annotation, and genotyping via
  a reproducible Snakemake workflow.
when_to_use_negative:
- Input reads are short-read (Illumina) data — RetroInspector is designed for nanopore
  long-read sequencing of human genomes
- The task is TE analysis in a non-human organism or against a reference where gene
  annotation is unavailable, since annotation steps are overreliant on a particular
  reference version
- You need a single-call tool comparison benchmark including xTea, which was not used
  in RetroInspector's comparative evaluation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: RetroInspector
  role: The Snakemake pipeline being executed; performs detection, annotation, enrichment,
    and genotyping of TEs
  repo: github.com/javiercguard/retroinspector
- name: Snakemake
  role: Workflow engine that executes the pipeline with --use-conda and --config parameters
- name: minimap2
  role: Aligns basecalled nanopore reads against the GRCh38 reference genome
  repo: github.com/lh3/minimap2
- name: cuteSV
  role: Variant caller on aligned reads; combined with Sniffles2 gives best agreement
    with the HGSV truth set
  repo: github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Variant caller whose algorithm RetroInspector reimplements for genotyping;
    best concordance with cuteSV (96.99% precision on HGSV2 genotyping)
  repo: github.com/fritzsedlazeck/Sniffles
- name: mosdepth
  role: Computes per-sample genome-wide coverage used in genotyping (98.3% of genotyping
    runtime)
  repo: github.com/brentp/mosdepth
- name: spoa
  role: Partial-order alignment used to reassemble the affected region from supporting
    reads
  repo: github.com/rvaser/spoa
- name: pysam
  role: Library used for BAM/VCF manipulation within the pipeline
  repo: github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:d5bd9f197f380941a3ce8ff267e111aa2f75df5bc71bb3cf8db11cd8258ed3c3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  tools:
  - RetroInspector
  - Snakemake
  - Sniffles2
  - cuteSV
  - minimap2
  - mosdepth
  - pysam
  - spoa
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# snakemake-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the RetroInspector Snakemake pipeline to detect, annotate, enrich, and genotype TE insertions and deletions from nanopore sequencing data aligned to GRCh38, with configurable parameters such as minimum supporting read threshold (default 3).

## When to use

Use when raw nanopore FASTQ/FASTA files from human samples need end-to-end transposable element insertion/deletion detection, annotation, and genotyping via a reproducible Snakemake workflow.

## When NOT to use

- Input reads are short-read (Illumina) data — RetroInspector is designed for nanopore long-read sequencing of human genomes
- The task is TE analysis in a non-human organism or against a reference where gene annotation is unavailable, since annotation steps are overreliant on a particular reference version
- You need a single-call tool comparison benchmark including xTea, which was not used in RetroInspector's comparative evaluation

## Inputs

- Nanopore FASTA/FASTQ files, one per sample (fastq_directory)
- Human reference genome FASTA (hg38/GRCh38)
- Optional YAML configuration file (copied from config.template.yaml)

## Outputs

- Detected and annotated TE insertion/deletion callset per sample
- Genotypes for insertions and deletions (0/0 homozygous-reference variants filtered out)
- Reassembled inserted-region sequences from supporting reads
- Sample-vs-sample comparison results (60 bp default merge limit)
- Enrichment and gene annotation of TE events

## How to apply

Clone the retroinspector repository, create a snakemake conda/mamba environment ('mamba create -n snakemake snakemake'), then run 'snakemake --use-conda' with --config supplying referenceGenome (hg38/GRCh38 FASTA), outputPath, and fastq_directory. First execute with the '-n' (dry-run) flag to verify the job DAG before launching, and set thread count with '-c <n>'. The pipeline aligns basecalled reads against GRCh38 with minimap2, calls variants with cuteSV and Sniffles2 (default and recommended: at least 3 supporting reads), reassembles inserted regions from supporting reads with spoa, computes genome-wide coverage with mosdepth, and genotypes insertions/deletions using a reimplementation of the Sniffles2 algorithm, filtering out variants genotyped 0/0. Optionally copy config.template.yaml to configure settings rather than passing everything on the command line; per-stage runtime expectations are alignment ~231.88 ± 32 min/sample, sorting/indexing ~12.78 ± 1.31 min, Sniffles2 ~7.29 ± 0.33 min, cuteSV ~28.01 ± 0.90 min, and genotyping ~37.83 ± 3.59 min per sample.

## Related tools

- **RetroInspector** (The Snakemake pipeline being executed; performs detection, annotation, enrichment, and genotyping of TEs) — github.com/javiercguard/retroinspector
- **Snakemake** (Workflow engine that executes the pipeline with --use-conda and --config parameters)
- **minimap2** (Aligns basecalled nanopore reads against the GRCh38 reference genome) — github.com/lh3/minimap2
- **cuteSV** (Variant caller on aligned reads; combined with Sniffles2 gives best agreement with the HGSV truth set) — github.com/tjiangHIT/cuteSV
- **Sniffles2** (Variant caller whose algorithm RetroInspector reimplements for genotyping; best concordance with cuteSV (96.99% precision on HGSV2 genotyping)) — github.com/fritzsedlazeck/Sniffles
- **mosdepth** (Computes per-sample genome-wide coverage used in genotyping (98.3% of genotyping runtime)) — github.com/brentp/mosdepth
- **spoa** (Partial-order alignment used to reassemble the affected region from supporting reads) — github.com/rvaser/spoa
- **pysam** (Library used for BAM/VCF manipulation within the pipeline) — github.com/pysam-developers/pysam

## Examples

```
mamba activate snakemake; snakemake --use-conda -c 8 --config referenceGenome=/home/you/hg38.fa outputPath=/home/you/results fastq_directory=/home/you/data/samples
```

## Evaluation signals

- Dry-run with '-n' produces a job list without errors before the real run is launched
- Genotyping stage output shows high per-allele genotype concordance with the HGSV2 truth set (>95% precision for all caller combinations; best 96.99% with cuteSV+Sniffles2)
- Insertion detection precision exceeds 80%: cuteSV & Sniffles2 strict gives 80.90% TE precision (85.63% all-SV) and 87.10% recall on the HGSV truth set
- Genotypes for variants called 0/0 are filtered out and absent from final outputs
- Per-stage runtimes fall near reported means (e.g., alignment ~232 min/sample; Sniffles2 ~7.3 min/sample) on comparable data

## Limitations

- Alignment is the most time-consuming stage (~231.88 ± 32 min per sample), so the pipeline is expensive on large cohorts
- HGSVC truth-set data was basecalled with an older basecaller; even after reassembly sequences may be imperfect, particularly for homopolymers
- SVA F1 elements are not in Dfam, so identification relies on RetroInspector's MAST2 thumbprint check
- Reassembly excludes supporting reads whose insertion length differs by more than 15% of the reported insertion length, which may drop true support reads
- Gene annotation may not be available when a new reference genome sequence is released, making steps overreliant on a particular reference version

## Evidence

    outputPath="/home/yourname/experiment_results" \
    fastq_directory="/home/yourname/data/samples""
