---
name: genome-wide-coverage-quantification
description: Use when nanopore (or other long-read) alignment (BAM from minimap2 vs
  GRCh38) must be converted into per-sample, genome-wide coverage to genotype SV/TE
  insertion and deletion genotypes.
when_to_use_negative:
- Input is raw unaligned FASTQ with no BAM available — coverage quantification requires
  an aligned, sorted/indexed alignment file first
- The goal is de novo TE insertion detection or annotation rather than assigning genotypes
  (0/1, 1/1, 0/0) to already-called variants
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: mosdepth
  role: Computes the per-sample genome-wide coverage that feeds the genotyping stage
  repo: https://github.com/brentp/mosdepth
- name: RetroInspector
  role: Genotypes insertions and deletions from the coverage profiles and read support
    using its Sniffles2 reimplementation
  repo: https://github.com/javiercguard/retroinspector
- name: minimap2
  role: Aligns basecalled nanopore reads to GRCh38 to produce the alignments coverage
    is computed from
  repo: https://github.com/lh3/minimap2
- name: Sniffles2
  role: Reference algorithm whose genotyping logic RetroInspector reimplements
  repo: https://github.com/fritzsedlazeck/Sniffles
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:3c1e6b62fca545d0368838486cf1e56fc29d0451a23180a34807dca23264f301
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/brentp/mosdepth
  tools:
  - RetroInspector
  - Sniffles2
  - minimap2
  - mosdepth
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# genome-wide-coverage-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-sample genome-wide read-depth profiles with mosdepth as the substrate for RetroInspector's TE insertion/deletion genotyping, which reimplements the Sniffles2 genotyping algorithm on top of coverage plus read-support information.

## When to use

Use when nanopore (or other long-read) alignment (BAM from minimap2 vs GRCh38) must be converted into per-sample, genome-wide coverage to genotype SV/TE insertion and deletion genotypes.

## When NOT to use

- Input is raw unaligned FASTQ with no BAM available — coverage quantification requires an aligned, sorted/indexed alignment file first
- The goal is de novo TE insertion detection or annotation rather than assigning genotypes (0/1, 1/1, 0/0) to already-called variants

## Inputs

- Sorted, indexed BAM/CRAM alignments of nanopore reads against GRCh38 (produced with minimap2)
- Per-sample FASTQ/FASTA files (one per sample) as used by the RetroInspector Snakemake pipeline
- Variant callsets from cuteSV and/or Sniffles2 with at least the default 3 supporting reads

## Outputs

- Per-sample genome-wide coverage profiles (mosdepth output)
- Genotypes for TE insertions and deletions per sample (0/0 filtered), e.g. genotyping_benchmark_metrics.tsv when benchmarking

## How to apply

After aligning the basecalled nanopore reads to GRCh38 with minimap2 and sorting/indexing the BAM, run mosdepth per sample to obtain genome-wide coverage; this is the dominant cost of the genotyping stage (98.3% of the 37.83 ± 3.59 min/sample genotyping time). RetroInspector then uses these coverage profiles together with read support to genotype insertions and deletions via its reimplementation of the Sniffles2 algorithm, applying the pipeline's default minimum read support of 3. Variants genotyped 0/0 (homozygous reference) across samples are filtered out. Genotype concordance against a curated truth set (e.g., HGSV2) can be scored per allele, counting 0/1 as one positive and 1/1 as two.

## Related tools

- **mosdepth** (Computes the per-sample genome-wide coverage that feeds the genotyping stage) — https://github.com/brentp/mosdepth
- **RetroInspector** (Genotypes insertions and deletions from the coverage profiles and read support using its Sniffles2 reimplementation) — https://github.com/javiercguard/retroinspector
- **minimap2** (Aligns basecalled nanopore reads to GRCh38 to produce the alignments coverage is computed from) — https://github.com/lh3/minimap2
- **Sniffles2** (Reference algorithm whose genotyping logic RetroInspector reimplements) — https://github.com/fritzsedlazeck/Sniffles

## Examples

```
snakemake --use-conda -c 8 --config referenceGenome=/home/yourname/hg38.fa outputPath=/home/yourname/experiment_results fastq_directory=/home/yourname/data/samples
```

## Evaluation signals

- Genotyping stage runtime should be dominated by coverage calculation (~98.3% of ~38 min per sample in the paper)
- Variants genotyped 0/0 across samples are removed from the output callset
- Per-allele genotype concordance against the HGSV2 curated truth set should be high, with precision exceeding 95% (best 96.99% with cuteSV+Sniffles2)
- Metrics are written to genotyping_benchmark_metrics.tsv when the benchmark workflow is run
- Only autosomal-chromosome variants present in the curated truth set should enter the concordance scoring

## Limitations

- Publicly available HGSVC data was basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers
- Reassembly excludes supporting reads whose insertion length differs by more than 15% of the reported insertion length, which can affect the reads feeding genotyping
- Coverage computation is the runtime bottleneck (98.3% of genotyping time), so throughput is bounded by mosdepth performance on whole genomes

## Evidence

- [methods] mosdepth is the coverage tool: [evidence span withheld — anchor status: not_found]
