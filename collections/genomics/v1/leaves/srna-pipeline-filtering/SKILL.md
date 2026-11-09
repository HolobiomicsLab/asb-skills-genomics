---
name: srna-pipeline-filtering
description: Use when processing plant sRNA-seq FASTQ/FASTA libraries and you need
  reproducible, parameterized read filtering plus genome-mapped reads before conserved
  miRNA (miRBase) or novel miRNA/tasiRNA analysis.
when_to_use_negative:
- When the goal is strict genome-based read removal — wbench_filter.cfg sets filter_genome_hits=false,
  so this stage does NOT discard reads based on genome hits
- When reads should be mapped to the genome with mismatches or gaps allowed — patman_genome.cfg
  fixes EDITS=0 and GAPS=0, so this skill only supports exact 0-mismatch mapping
- When the aim is quality control of raw reads — fastqc is not yet implemented in
  miRPursuit
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: patman
  role: Pattern matcher used to map the Wbench-filtered reads to the configured genome
    with 0 mismatches, producing genome_mapped_reads.gff
  repo: https://bioinf.eva.mpg.de/patman/
- name: PatMaN
  role: Short-sequence pattern matcher implementing the genome-mapping filter step
    of the miRPursuit Filtering stage
  repo: http://bioinf.eva.mpg.de/patman
- name: FASTX-Toolkit
  role: fastx_clipper performs the adaptor-trimming sub-step of Step 1 before Wbench
    filtering
  repo: http://hannonlab.cshl.edu/fastx_toolkit/
- name: miRPursuit
  role: Pipeline shell scripts that chain Wbench Filter and PatMaN genome mapping
    via configuration files (wbench_filter.cfg, patman_genome.cfg)
  repo: https://github.com/forestbiotech-lab/miRPursuit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:8d741c88b19f51bc8809da5fd35a4b22ca1486ced9cbd9b445a3d61a58039710
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FASTX-Toolkit
  - PatMaN
  - miRPursuit
  - patman
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# srna-pipeline-filtering

## Summary

Filters plant small-RNA (sRNA) sequencing reads in the miRPursuit Filtering stage: adaptor trimming, Wbench filtering (length 18–26, abundance ≥5, low-complexity, t/rRNA removal), then mapping reads to the reference genome with PatMaN at 0 mismatches/0 gaps.

## When to use

Use when processing plant sRNA-seq FASTQ/FASTA libraries and you need reproducible, parameterized read filtering plus genome-mapped reads before conserved miRNA (miRBase) or novel miRNA/tasiRNA analysis.

## When NOT to use

- When the goal is strict genome-based read removal — wbench_filter.cfg sets filter_genome_hits=false, so this stage does NOT discard reads based on genome hits
- When reads should be mapped to the genome with mismatches or gaps allowed — patman_genome.cfg fixes EDITS=0 and GAPS=0, so this skill only supports exact 0-mismatch mapping
- When the aim is quality control of raw reads — fastqc is not yet implemented in miRPursuit

## Inputs

- Raw sRNA-seq FASTQ or FASTA files (Lib_1.fq, Lib_1.fa, or fastq.gz archives) from one or more libraries
- Reference genome FASTA (e.g. Arabidopsis_thaliana.TAIR10.dna_rm.chromosome.4.fa, from Phytozome or Ensembl Plants)
- Config files: wbench_filter.cfg, patman_genome.cfg, workdirs.cfg (incl. ADAPTOR, FILTER_SUF, INSERTS_DIR)

## Outputs

- Adaptor-trimmed and Wbench-filtered sRNA read set (length-, abundance-, low-complexity-, t/rRNA-filtered FASTA)
- genome_mapped_reads.gff — read-to-genome alignments from PatMaN used as input to Step 2: Filter Genome & mirbase
- Read-count statistics in the counts directory (space-separated tsv-like files)

## How to apply

Run miRPursuit Step 1 (Adaptor trimming via fastx_clipper from FASTX-Toolkit, if flagged with --trim, followed by Wbench Filter) on the raw FASTQ/FASTA inputs, configured through wbench_filter.cfg with min_length=18, max_length=26, min_abundance=5, trrna=true, filter_low_comp=true, filter_invalid=true, and filter_genome_hits=false. Take the filtered sRNA read set and map it to the configured genome (e.g. the test genome Arabidopsis_thaliana.TAIR10.dna_rm.chromosome.4.fa) using patman as the short-sequence pattern matcher with EDITS=0, GAPS=0, SINGLESTRAND=FALSE, PREFETCH=3 as set in patman_genome.cfg. Because filter_genome_hits=false, no reads are discarded based on genome hits; the genome-mapped reads are written as genome_mapped_reads.gff and feed into Step 2 (Filter Genome & mirbase). The rationale is that genome mapping with 0 mismatches anchors the filtered reads for downstream miRProf alignment to miRBase and miRCat/tasi prediction, while retaining the filtering decision in explicit config files for reproducibility.

## Related tools

- **patman** (Pattern matcher used to map the Wbench-filtered reads to the configured genome with 0 mismatches, producing genome_mapped_reads.gff) — https://bioinf.eva.mpg.de/patman/
- **PatMaN** (Short-sequence pattern matcher implementing the genome-mapping filter step of the miRPursuit Filtering stage) — http://bioinf.eva.mpg.de/patman
- **FASTX-Toolkit** (fastx_clipper performs the adaptor-trimming sub-step of Step 1 before Wbench filtering) — http://hannonlab.cshl.edu/fastx_toolkit/
- **miRPursuit** (Pipeline shell scripts that chain Wbench Filter and PatMaN genome mapping via configuration files (wbench_filter.cfg, patman_genome.cfg)) — https://github.com/forestbiotech-lab/miRPursuit

## Examples

```
bash pathToMirPursuit/miRPursuit.sh -f 1 -l 2 --fasta test_dataset-
```

## Evaluation signals

- Confirm PatMaN ran with exactly EDITS=0 and GAPS=0 per patman_genome.cfg
- Verify genome_mapped_reads.gff exists and is parseable as a GFF of read-to-genome alignments
- Check wbench_filter.cfg values: min_length=18, max_length=26, min_abundance=5, trrna=true, filter_low_comp=true, filter_genome_hits=false
- Confirm the filtered read set feeds Step 2 (Filter Genome & mirbase) and that read-count statistics appear in the counts directory
- Confirm pipeline execution logs in [workdir]/logs end with *.log.ok for the filtering steps

## Limitations

- Genome-mapping step permits only exact matches (0 edits, 0 gaps), so reads with sequencing variation relative to the genome are not mapped
- Quality control with fastqc is not yet implemented in miRPursuit
- filter_genome_hits=false means genome hits alone do not remove reads; the filtering decisions live in config files that must be set per project (FILTER_SUF, ADAPTOR, INSERTS_DIR)
- Java version matters: the article specifies Java v.8 while the README check-list mentions ~1.7 — environment mismatch can break the pipeline

## Evidence

- [readme] patman_genome.cfg holds the genome filtering parameters: [evidence span withheld — anchor status: not_found]
