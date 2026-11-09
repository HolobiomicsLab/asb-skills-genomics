---
name: dnv-quality-metrics-benchmark-reproducibility
description: Use when reproducing or benchmarking a HAT (Hare/Tortoise) DNV callset
  on GIAB trio WGS data, or when validating DNV call quality via DNVs per individual,
  percent at CpG sites, and percent phased to the paternal chromosome of origin.
when_to_use_negative:
- You only have existing VCFs and no BAM/CRAM access — HAT-FLEX (caller-agnostic,
  drop-in trio DNV detection operating directly on VCFs) is the right tool instead
  of re-running the Hare/Tortoise workflows
- Data is long-read PacBio HiFi WGS — Hare v1.2 is effective for WGS and WES data
  only; use the Tortoise workflow, which is effective for WGS, WES and PacBio data
- Input is a single sample (no parent-child sequenced trio) — HAT begins with aligned
  read data from a parent-child trio and cannot derive parent-of-origin/phasing metrics
  without all three family members
edam_operation: http://edamontology.org/operation_3227
tools:
- name: HAT
  role: end-to-end DNV detection workflow (custom de novo variant filter) whose output
    quality metrics this skill benchmarks
  repo: https://github.com/TNTurnerLab/HAT
- name: Tortoise
  role: open-source, CPU-only version of the DNV calling pipeline (DeepVariant + GATK
    HaplotypeCaller) used to call variants on the downsampled trio
  repo: https://github.com/TNTurnerLab/Tortoise
- name: GATK 4.2.0 HaplotypeCaller
  role: per-sample variant caller run on the downsampled ~30x trio as the open-source
    analogue of the Parabricks-based Hare pipeline
  repo: https://github.com/TNTurnerLab/Tortoise
- name: DeepVariant 1.4.0
  role: second per-sample variant caller on the downsampled trio, combined with HaplotypeCaller
    before joint genotyping
  repo: https://github.com/TNTurnerLab/Tortoise
- name: GLnexus
  role: forms family-level joint-genotyped files that are run through HAT's custom
    de novo variant filter
  repo: https://github.com/TNTurnerLab/HAT
- name: samtools
  role: deterministically downsamples ~300x trio reads to ~30x via view --subsample
    with seed 69
- name: squirrel.py
  role: converts HAT output to acorn input for downstream quality statistics
  repo: https://github.com/TNTurnerLab/HAT
- name: acorn
  role: R package used to summarize DNV characteristics, including annotateCpG to
    compute percent of DNV SNVs at CpG sites using the cpg_b38.rda annotation
  repo: https://github.com/TNTurnerLab/acorn
- name: genome-in-a-bottle/giab_data_indexes
  role: source of the HG002/HG003/HG004 AshkenazimTrio sequencing and alignment index
    files
  repo: https://github.com/genome-in-a-bottle/giab_data_indexes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:2eee25cc6c09a62e67bb93fb04ee9ee39f7161c604a093845037b20f6802dc04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/HAT
  tools:
  - DeepVariant
  - DeepVariant 1.4.0
  - GATK
  - GATK 4.2.0 HaplotypeCaller
  - GLnexus
  - HAT
  - Tortoise
  - acorn
  - genome-in-a-bottle/giab_data_indexes
  - samtools
  - squirrel.py
  edam_operation: http://edamontology.org/operation_3227
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# dnv-quality-metrics-benchmark-reproducibility

## Summary

Benchmark and reproduce a HAT de novo variant (DNV) callset on the GIAB HG002 Ashkenazim trio by downsampling ~300x Illumina WGS to ~30x, running the Hare/Tortoise-style workflow (GATK HaplotypeCaller, DeepVariant, GLnexus, HAT's custom de novo filter), and validating against quality metrics such as total DNVs per individual and percent of DNVs at CpG sites.

## When to use

Use when reproducing or benchmarking a HAT (Hare/Tortoise) DNV callset on GIAB trio WGS data, or when validating DNV call quality via DNVs per individual, percent at CpG sites, and percent phased to the paternal chromosome of origin.

## When NOT to use

- You only have existing VCFs and no BAM/CRAM access — HAT-FLEX (caller-agnostic, drop-in trio DNV detection operating directly on VCFs) is the right tool instead of re-running the Hare/Tortoise workflows
- Data is long-read PacBio HiFi WGS — Hare v1.2 is effective for WGS and WES data only; use the Tortoise workflow, which is effective for WGS, WES and PacBio data
- Input is a single sample (no parent-child sequenced trio) — HAT begins with aligned read data from a parent-child trio and cannot derive parent-of-origin/phasing metrics without all three family members

## Inputs

- GIAB HG002/HG003/HG004 parent-child trio aligned Illumina WGS reads (BAM or CRAM, ~300x, from genome-in-a-bottle/giab_data_indexes)
- GRCh38 human reference genome
- family joint-genotyped VCFs from GLnexus (for the HAT custom de novo variant filter)

## Outputs

- per-child de novo variant (DNV) callset in VCF/tab-delimited format for HG002 (~1105 DNVs at ~30x coverage)
- HAT output prepped by squirrel.py into acorn input format (sample, chromosome, genomic position, reference allele, alternate allele, optional columns)
- CpG summary from acorn annotateCpG reporting number of CpG DNVs and percent of DNV SNVs at CpG (~16.5%)

## How to apply

Download the GIAB AshkenazimTrio BAM/CRAM files (HG002 son, HG003 father, HG004 mother) from genome-in-a-bottle/giab_data_indexes along with the GRCh38 reference. Downsample each family member's reads to ~30x coverage using samtools view --subsample with seed 69 so the subsampling is deterministic and reproducible. Run per-sample variant calling on the downsampled trio with the open-source Tortoise workflow (GATK 4.2.0 HaplotypeCaller and DeepVariant 1.4.0 — the CPU analogues of the Parabricks-based Hare pipeline), then joint-genotype the family call sets with GLnexus. Apply HAT's custom de novo variant filter to the joint-genotyped files to extract candidate DNVs for the HG002 child. Summarize the output by prepping it with squirrel.py and running acorn's annotateCpG (with the pre-computed cpg_b38.rda annotation) to compute total DNV count and the fraction of DNVs at CpG sites. The callset is accepted if it reproduces approximately 1105 DNVs for the child with approximately 16.5% of DNVs at CpG sites, matching the paper's quality metrics.

## Related tools

- **HAT** (end-to-end DNV detection workflow (custom de novo variant filter) whose output quality metrics this skill benchmarks) — https://github.com/TNTurnerLab/HAT
- **Tortoise** (open-source, CPU-only version of the DNV calling pipeline (DeepVariant + GATK HaplotypeCaller) used to call variants on the downsampled trio) — https://github.com/TNTurnerLab/Tortoise
- **GATK 4.2.0 HaplotypeCaller** (per-sample variant caller run on the downsampled ~30x trio as the open-source analogue of the Parabricks-based Hare pipeline) — https://github.com/TNTurnerLab/Tortoise
- **DeepVariant 1.4.0** (second per-sample variant caller on the downsampled trio, combined with HaplotypeCaller before joint genotyping) — https://github.com/TNTurnerLab/Tortoise
- **GLnexus** (forms family-level joint-genotyped files that are run through HAT's custom de novo variant filter) — https://github.com/TNTurnerLab/HAT
- **samtools** (deterministically downsamples ~300x trio reads to ~30x via view --subsample with seed 69)
- **squirrel.py** (converts HAT output to acorn input for downstream quality statistics) — https://github.com/TNTurnerLab/HAT
- **acorn** (R package used to summarize DNV characteristics, including annotateCpG to compute percent of DNV SNVs at CpG sites using the cpg_b38.rda annotation) — https://github.com/TNTurnerLab/acorn
- **genome-in-a-bottle/giab_data_indexes** (source of the HG002/HG003/HG004 AshkenazimTrio sequencing and alignment index files) — https://github.com/genome-in-a-bottle/giab_data_indexes

## Examples

```
library('acorn'); load('cpg_b38.rda'); input <- readDNV('hat_dnvs.txt'); CpGresult <- annotateCpG(DNVobject = input)
```

## Evaluation signals

- Total DNV count for the HG002 child on the downsampled (~30x) Illumina WGS trio is approximately 1105, matching the paper's reported metric
- Approximately 16.5% of the child's DNVs occur at CpG sites as computed by acorn annotateCpG with the cpg_b38.rda annotation
- Downsampled BAMs/CRAMs are byte-reproducible across runs because samtools view --subsample is invoked with seed 69
- Joint-genotyped family files exist for all three trio members (HG002/HG003/HG004) and pass HAT's custom de novo variant filter to produce per-child DNV outputs
- Percent of DNVs phased to the paternal chromosome of origin is reported and consistent with the paper's quality metrics (via unfazed-style parent-of-origin assignment)

## Limitations

- Results depend on the downsampling procedure; the ~1105 DNVs / ~16.5% CpG benchmark was established with samtools view --subsample at seed 69, so different subsampling parameters may shift counts
- The Hare (Parabricks/GPU) workflow is effective for WGS and WES data only; PacBio HiFi requires the Tortoise workflow
- acorn's annotateCpG requires a pre-computed CpG rda file for the genome build of interest (available for b38); it is too large to package with acorn and takes at least one minute to run
- The benchmark reproduces the paper's metrics only within tolerance ('approximately 1105', 'approximately 16.5%'), not exactly

## Evidence

- [other] benchmark acceptance criteria: [evidence span withheld — anchor status: not_found]
- [other] downsampling protocol: [evidence span withheld — anchor status: not_found]
- [abstract] quality metrics framework: "quality metrics including number of DNVs per individual, percent of DNVs at CpG sites, and percent of DNVs phased to the paternal chromosome of"
- [intro] joint genotyping and filtering: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [methods] followup statistics tooling: "If you would like to do some followup statistics on the output from HAT, you can install the R package acorn"
- [readme] Tortoise input format: [evidence span withheld — anchor status: external_doc]
- [readme] CpG annotation requirement: [evidence span withheld — anchor status: external_doc]
