---
name: wgs-read-downsampling-coverage-normalization
description: Use when input BAM/CRAM files are sequenced far above the coverage needed
  for short-read variant calling (e.g. GIAB HG002/HG003/HG004 trio at ~300x) and the
  caller workflow expects ~30x WGS coverage.
when_to_use_negative:
- Input BAM/CRAM is already at or near the target coverage (e.g. the 1000 Genomes
  NA12878 WGS 30x sample) — subsampling would discard data needlessly
- The downstream caller is designed for the original high depth, e.g. when benchmarking
  on the full ~300x GIAB data rather than reproducing the ~30x call set
- Input is long-read PacBio HiFi data at typical HiFi depths — this normalization
  was defined for high-depth Illumina short-read trio data
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: samtools
  role: Performs the deterministic read subsampling (--subsample with seed 69) that
    normalizes the ~300x GIAB trio to ~30x coverage
- name: Tortoise
  role: Consumes the downsampled ~30x trio BAM/CRAMs in the open-source GATK HaplotypeCaller/DeepVariant
    + GLnexus DNV calling workflow
  repo: https://github.com/TNTurnerLab/Tortoise
- name: HAT
  role: Downstream DNV caller whose ~30x call set (1105 DNVs, 16.5% at CpG) is the
    validation target of the downsampling
  repo: https://github.com/TNTurnerLab/HAT
- name: genome-in-a-bottle/giab_data_indexes
  role: Source of the AshkenazimTrio (HG002/HG003/HG004) sequence and alignment indexes
    that are downsampled
  repo: https://github.com/genome-in-a-bottle/giab_data_indexes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:81610a33e966ef00b905ceb2b81e3a235949af3e93ddc5eb7b1b1a7847a1a9be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/Tortoise
  tools:
  - DeepVariant
  - GATK
  - GLnexus
  - HAT
  - Tortoise
  - acorn
  - genome-in-a-bottle/giab_data_indexes
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# wgs-read-downsampling-coverage-normalization

## Summary

Normalizes high-coverage Illumina WGS trio data (e.g. GIAB AshkenazimTrio ~300x) down to a target coverage such as ~30x with deterministic samtools subsampling, so that downstream de novo variant calling (HAT/Hare/Tortoise) operates at a comparable coverage to standard WGS benchmarks.

## When to use

Use when input BAM/CRAM files are sequenced far above the coverage needed for short-read variant calling (e.g. GIAB HG002/HG003/HG004 trio at ~300x) and the caller workflow expects ~30x WGS coverage.

## When NOT to use

- Input BAM/CRAM is already at or near the target coverage (e.g. the 1000 Genomes NA12878 WGS 30x sample) — subsampling would discard data needlessly
- The downstream caller is designed for the original high depth, e.g. when benchmarking on the full ~300x GIAB data rather than reproducing the ~30x call set
- Input is long-read PacBio HiFi data at typical HiFi depths — this normalization was defined for high-depth Illumina short-read trio data

## Inputs

- Aligned Illumina WGS read files (BAM or CRAM) for a parent-child trio, e.g. GIAB AshkenazimTrio HG002/HG003/HG004 at ~300x coverage
- GRCh38 reference genome used for the alignments
- Target coverage level (e.g. ~30x) and subsample fraction derived from original coverage

## Outputs

- Per-sample downsampled BAM/CRAM files at ~30x coverage with a fixed subsampling seed
- Coverage summaries confirming the ~30x target for each trio member

## How to apply

Download the trio's aligned reads (CRAM/BAM) from genome-in-a-bottle/giab_data_indexes (HG002 son, HG003 father, HG004 mother) along with the GRCh38 reference genome. Estimate the downsample fraction needed to bring each sample from ~300x to ~30x and run samtools view with the --subsample option, fixing the random seed at 69 so subsampling is deterministic and reproducible. Keep the same downsampled per-sample files as the shared input for both the GPU-accelerated Hare-style pipeline and the open-source Tortoise pipeline (GATK HaplotypeCaller and DeepVariant followed by GLnexus joint genotyping) so coverage differences cannot confound the comparison of DNV call sets. Verify the resulting coverage per sample before proceeding to variant calling, since the target of the reproduction (e.g. 1105 DNVs, 16.5% at CpG sites) is defined at the ~30x depth.

## Related tools

- **samtools** (Performs the deterministic read subsampling (--subsample with seed 69) that normalizes the ~300x GIAB trio to ~30x coverage)
- **Tortoise** (Consumes the downsampled ~30x trio BAM/CRAMs in the open-source GATK HaplotypeCaller/DeepVariant + GLnexus DNV calling workflow) — https://github.com/TNTurnerLab/Tortoise
- **HAT** (Downstream DNV caller whose ~30x call set (1105 DNVs, 16.5% at CpG) is the validation target of the downsampling) — https://github.com/TNTurnerLab/HAT
- **genome-in-a-bottle/giab_data_indexes** (Source of the AshkenazimTrio (HG002/HG003/HG004) sequence and alignment indexes that are downsampled) — https://github.com/genome-in-a-bottle/giab_data_indexes

## Examples

```
samtools view --subsample -s 69.1 HG002_300x.cram -b -o HG002_30x.cram
```

## Evaluation signals

- Post-downsample mean coverage per sample is approximately 30x (i.e. roughly one-tenth of the original ~300x), for each of HG002, HG003, HG004
- Rerunning samtools view with seed 69 reproduces the identical downsampled file (byte-identical or identical DNV call set), confirming determinism
- Downstream HAT/Tortoise run on the downsampled trio yields approximately 1105 DNVs for HG002 with approximately 16.5% at CpG sites
- All three trio members are downsampled with the same procedure/seed so family-level GLnexus joint genotyping remains valid
- Downsampled files remain valid CRAM/BAM indexed inputs that GATK HaplotypeCaller and DeepVariant can process without errors

## Limitations

- Subsampling reduces effective depth, so de novo sensitivity at true ~300x depth is not preserved — the target result (~1105 DNVs) is specific to the ~30x setting
- The seed (69) and fraction must be recorded; using a different seed changes the downsampled read set and can shift the DNV call set
- Coverage normalization was defined for Illumina short-read trio data; PacBio HiFi long-read data in the HAT workflow is handled at its native depth

## Evidence

- [other] Downsampling ~300x trio to ~30x with samtools --subsample seed 69: [evidence span withheld — anchor status: not_found]
- [readme] GIAB trio is the data source: [evidence span withheld — anchor status: external_doc]
- [intro] Tortoise consumes the open-source callers at the normalized coverage: "Tortoise uses freely available, open source versions of the these variant callers"
- [intro] GLnexus joint genotyping follows calling on the downsampled files: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
