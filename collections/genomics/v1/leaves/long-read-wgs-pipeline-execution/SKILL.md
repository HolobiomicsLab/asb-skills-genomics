---
name: long-read-wgs-pipeline-execution
description: Use when raw long-read whole-genome reads (PacBio HiFi or targeted ONT
  FASTQ/BAM) from human rare-disease samples must be turned into aligned BAMs, phased
  variant calls, de novo assemblies and methylation profiles via a containerized Nextflow
  run.
when_to_use_negative:
- 'Input data are short-read or single-cell: Nallo is built for long reads from PacBio
  and (targeted) ONT data focused on rare-disease'
- 'The goal is only a single variant-calling subtask on a limited budget: epi2me-labs/wf-human-variation
  or PacificBiosciences/HiFi-human-WGS-WDL may suffice instead of the full Nallo pipeline
  including assembly (~344 core hours/sample)'
- 'Coverage is far below ~30x (e.g. targeted panels below 20x): alternative workflows
  may terminate or yield unexpected outcomes at low depth'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nallo
  role: The Nextflow pipeline being executed; orchestrates alignment, variant calling,
    phasing, assembly and methylation steps
  repo: https://github.com/genomic-medicine-sweden/nallo
- name: Nextflow
  role: Workflow engine used to launch Nallo with containerized execution (Docker/Singularity)
    and to emit trace reports of allocated cpus and runtimes
  repo: https://github.com/genomic-medicine-sweden/nallo
- name: Docker
  role: Container runtime isolating tool dependencies during local runs and tests
  repo: https://www.docker.com/products/docker-desktop/
- name: Minimap2
  role: Maps long reads to the GRCh38 reference genome
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts, merges and indexes aligned reads
  repo: https://github.com/samtools/samtools
- name: Hifiasm
  role: Assembles haplotypes from HiFi reads; assembled haplotypes are realigned with
    minimap2
  repo: https://github.com/chhylp123/hifiasm
- name: gfastats
  role: Converts the assembly to FASTA before alignment and outputs per-haplotype
    summary statistics
  repo: https://github.com/vgl-hub/gfastats
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag086
    title: 'Nallo: a Nextflow pipeline for comprehensive human long-read genome analysis'
  source_content_hash: sha256:ed781a5e1cfc998e86ec44bb436144537ce67d9105d02ff6c7cbb7fa67575b3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomic-medicine-sweden/nallo
  tools:
  - Docker
  - Hifiasm
  - Minimap2
  - Nallo
  - Nextflow
  - gfastats
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# long-read-wgs-pipeline-execution

## Summary

Executes Nallo, a Nextflow pipeline for comprehensive human long-read genome analysis (PacBio and Oxford Nanopore data), covering alignment, variant detection, phasing, de novo assembly and CpG methylation reporting. The skill includes benchmarked compute expectations (~344 core hours per sample, 5 h 52 min wall-clock for a ~32x PacBio HiFi sample on a 68-node dual Intel Xeon Gold 6248R cluster).

## When to use

Use when raw long-read whole-genome reads (PacBio HiFi or targeted ONT FASTQ/BAM) from human rare-disease samples must be turned into aligned BAMs, phased variant calls, de novo assemblies and methylation profiles via a containerized Nextflow run.

## When NOT to use

- Input data are short-read or single-cell: Nallo is built for long reads from PacBio and (targeted) ONT data focused on rare-disease
- The goal is only a single variant-calling subtask on a limited budget: epi2me-labs/wf-human-variation or PacificBiosciences/HiFi-human-WGS-WDL may suffice instead of the full Nallo pipeline including assembly (~344 core hours/sample)
- Coverage is far below ~30x (e.g. targeted panels below 20x): alternative workflows may terminate or yield unexpected outcomes at low depth

## Inputs

- ~32x PacBio HiFi whole-genome sequencing FASTQ files (e.g. GIAB HG002/HG003/HG004 trio)
- GRCh38 reference genome and reference resources
- PED/FAM pedigree file for trio/family analysis (rare-disease focus)
- Optional region/gene lists for methylation profiling (MethBat) and variant filtering (--filter_variants_hgnc_id)

## Outputs

- Sorted, merged, indexed and haplotagged BAM alignments (Minimap2/samtools)
- Phased, annotated VCFs covering SNVs/indels and structural variants with functional-consequence ranking
- Filtered family VCFs (filter_vep/bcftools) when filtering parameters are enabled
- Hifiasm haplotype assemblies in FASTA with per-haplotype summary statistics (gfastats)
- CpG methylation pileups (Modkit or pb-CpG-tools) and region methylation profiles (MethBat)
- Nextflow trace report and aggregated per-sample compute metrics (core-hours, wall-clock time)

## How to apply

Fetch Nallo from genomic-medicine-sweden/nallo and launch it with Nextflow under containerized execution (Docker or Singularity via the -profile flag). Supply a samplesheet of ~32x PacBio HiFi WGS FASTQ inputs (e.g. GIAB trio HG002/HG003/HG004) together with GRCh38 reference resources; the pipeline maps reads with Minimap2, sorts/merges/indexes with samtools, phases with LongPhase/WhatsHap/HiPhase (haplotagging the aligned reads), calls and annotates a wide range of variants, assembles genomes with Hifiasm, converts/summarizes assemblies with gfastats, and generates methylation pileups with Modkit or pb-CpG-tools (Revio data). Enable Nextflow trace/executed reporting so each process logs allocated cpus and real-time; parse the trace to compute per-sample core-hours (allocated cpus x runtime summed over processes) and validate against the published ~344 core-hours-per-sample, 5h52m benchmark. Optionally use --filter_variants_hgnc_id or --filter_snvs_expression to obtain family VCFs filtered with filter_vep and bcftools.

## Related tools

- **Nallo** (The Nextflow pipeline being executed; orchestrates alignment, variant calling, phasing, assembly and methylation steps) — https://github.com/genomic-medicine-sweden/nallo
- **Nextflow** (Workflow engine used to launch Nallo with containerized execution (Docker/Singularity) and to emit trace reports of allocated cpus and runtimes) — https://github.com/genomic-medicine-sweden/nallo
- **Docker** (Container runtime isolating tool dependencies during local runs and tests) — https://www.docker.com/products/docker-desktop/
- **Minimap2** (Maps long reads to the GRCh38 reference genome) — https://github.com/lh3/minimap2
- **samtools** (Sorts, merges and indexes aligned reads) — https://github.com/samtools/samtools
- **Hifiasm** (Assembles haplotypes from HiFi reads; assembled haplotypes are realigned with minimap2) — https://github.com/chhylp123/hifiasm
- **gfastats** (Converts the assembly to FASTA before alignment and outputs per-haplotype summary statistics) — https://github.com/vgl-hub/gfastats

## Examples

```
nextflow run genomic-medicine-sweden/nallo -r v0.5.1 -profile docker --input samplesheet.csv --ref GRCh38.fasta --with-trace
```

## Evaluation signals

- Per-sample core-hours parsed from the Nextflow trace (allocated cpus x runtime summed over all processes) are consistent with the published ~344 core hours per sample
- Total wall-clock completion per ~32x sample is approximately 5 hours 52 minutes on comparable hardware (68 dual Intel Xeon Gold 6248R nodes)
- Output BAMs are sorted, merged, indexed and, when phasing is enabled, carry haplotype tags from the active phasing tool
- Assemblies exist in FASTA form with per-haplotype summary statistics produced by gfastats
- Methylation pileups (Modkit or pb-CpG-tools) and, for PacBio data, MethBat region methylation profiles are produced for every sample

## Limitations

- Compute cost is high: a complete run (alignment, variant calling, de novo assembly) consumes ~344 core hours per ~32x sample, requiring a cluster-scale deployment
- Runtime figures are benchmark-specific (Nallo v0.5.1, 68-node dual Xeon Gold 6248R cluster, ~32x PacBio HiFi trio); other coverage levels, ONT data, or hardware will shift wall-clock and core-hour results
- Variant filtering outputs are conditional: family VCFs are filtered only if --filter_variants_hgnc_id or --filter_snvs_expression is used
- pb-CpG-tools methylation pileups are limited to Revio data; ONT methylation goes through Modkit instead

## Evidence

- [abstract] Nallo is a Nextflow pipeline for long-read analysis: "Here we present Nallo, a Nextflow pipeline for analysis of PacBio and Oxford Nanopore data"
- [intro] Mapping and indexing tooling: "[Minimap2](https://github.com/lh3/minimap2) is used to map the reads to a reference genome. The aligned reads are sorted, merged and indexed using [samtools]"
- [intro] Assembly tooling: "[Hifiasm](https://github.com/chhylp123/hifiasm) is used to assemble genomes. The assembled haplotypes are then aligned to the reference genome with [minimap2]"
- [readme] Coverage recommendation for long-read workflows: [evidence span withheld — anchor status: not_found]
- [other] Compute benchmark finding: [evidence span withheld — anchor status: not_found]
