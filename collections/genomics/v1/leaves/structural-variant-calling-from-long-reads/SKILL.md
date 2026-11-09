---
name: structural-variant-calling-from-long-reads
description: Use when detecting inversion structural variants from long noisy reads
  (PacBio or Oxford Nanopore) mapped to a reference genome, especially when implanted
  or known inversion truth sets (e.g., SURVIVOR VCF) must be evaluated per mapper.
when_to_use_negative:
- Input reads are short reads (Illumina); the article's method targets longer PacBio/Oxford
  Nanopore reads that frequently span SV breakpoints
- The analysis goal is SNV/small-indel calling rather than inversion structural variant
  detection
- No reference genome or truth VCF is available, so sensitivity/precision against
  implanted inversions cannot be evaluated
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: invMap
  role: Long-read mapper with inversion-aware chaining that produces SAM alignments
    feeding SVision; run as './invMap genome.fa reads.fa > invMap_aligned.sam'
  repo: https://github.com/zhang134/invMap.git
- name: SVision
  role: Structural variant caller run on each mapper's alignment file to call inversion
    SVs from the benchmark alignments
- name: SURVIVOR
  role: Used to implant 1000 inversions into the reference genome and to provide the
    ground-truth inversion set used to match SV calls
- name: samtools
  role: Converts each mapper's SAM output to sorted BAM prior to SV calling
- name: minimap2
  role: Competing long-read mapper whose alignments are processed through the same
    SVision pipeline for sensitivity comparison
- name: NGMLR
  role: Competing long-read mapper used to generate comparison alignments for the
    SVision benchmark
- name: Winnowmap2
  role: Competing long-read mapper used to generate comparison alignments for the
    SVision benchmark
- name: lra
  role: Competing long-read mapper included in the mapper accuracy benchmark (63.25%
    reference accuracy)
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:8dbdef19b96af4013177e8a83121daea086d5e4a854b0227fbd3bbd025cbbb3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - SURVIVOR
  - SVision
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# structural-variant-calling-from-long-reads

## Summary

Call inversion structural variants from PacBio/ONT long-read alignments using the invMap mapper followed by SVision, then benchmark sensitivity/precision against a SURVIVOR-derived truth set. This captures the invMap-SVision pipeline that the article shows recovers inversions better than minimap2-, NGMLR-, Winnowmap2- or lra-based pipelines.

## When to use

Use when detecting inversion structural variants from long noisy reads (PacBio or Oxford Nanopore) mapped to a reference genome, especially when implanted or known inversion truth sets (e.g., SURVIVOR VCF) must be evaluated per mapper.

## When NOT to use

- Input reads are short reads (Illumina); the article's method targets longer PacBio/Oxford Nanopore reads that frequently span SV breakpoints
- The analysis goal is SNV/small-indel calling rather than inversion structural variant detection
- No reference genome or truth VCF is available, so sensitivity/precision against implanted inversions cannot be evaluated

## Inputs

- Long-read sequencing data in FASTA or FASTQ format (e.g., HG002 PacBio long-read dataset)
- Reference genome in FASTA format (optionally with SURVIVOR-implanted inversions)
- Mapper alignments in SAM/BAM format per mapper (invMap, minimap2, NGMLR, Winnowmap2, lra)
- Ground-truth inversion VCF (SURVIVOR-derived truth set of implanted inversions)

## Outputs

- invMap_aligned.sam mapping output (SAM format)
- Sorted BAM per mapper after samtools conversion
- Per-mapper SV call sets as VCF files from SVision
- Comparison statistics table (e.g., table2_sv_evaluation.csv) with total calls, correct calls, sensitivity and precision per mapper

## How to apply

Map long reads to the (possibly inversion-altered) reference with invMap by running './invMap genome.fa reads.fa > invMap_aligned.sam', then convert and sort the SAM to BAM with samtools. Run SVision on each mapper's BAM to call inversion SVs, keeping the mapper identity attached to each call set, and export per-mapper call sets as VCF files. Match the SVision calls against the SURVIVOR-derived ground-truth inversion set (e.g., 1000 SURVIVOR-implanted inversions in GRCh37) to count total and correct calls per mapper. Compute sensitivity and precision per mapper; the invMap-SVision pipeline yields 125 inversion calls with 82 correct and a 2.5%–15.49% sensitivity improvement over Winnowmap2, minimap2, NGMLR and lra pipelines. Rationale: invMap's chaining-based aligned-region localization plus anchor checking for inversions produces alignments from which SVision calls inversions more completely than competing mappers.

## Related tools

- **invMap** (Long-read mapper with inversion-aware chaining that produces SAM alignments feeding SVision; run as './invMap genome.fa reads.fa > invMap_aligned.sam') — https://github.com/zhang134/invMap.git
- **SVision** (Structural variant caller run on each mapper's alignment file to call inversion SVs from the benchmark alignments)
- **SURVIVOR** (Used to implant 1000 inversions into the reference genome and to provide the ground-truth inversion set used to match SV calls)
- **samtools** (Converts each mapper's SAM output to sorted BAM prior to SV calling)
- **minimap2** (Competing long-read mapper whose alignments are processed through the same SVision pipeline for sensitivity comparison)
- **NGMLR** (Competing long-read mapper used to generate comparison alignments for the SVision benchmark)
- **Winnowmap2** (Competing long-read mapper used to generate comparison alignments for the SVision benchmark)
- **lra** (Competing long-read mapper included in the mapper accuracy benchmark (63.25% reference accuracy))

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap-SVision pipeline reproduces 125 total inversion calls with 82 correct calls against the ground truth
- Sensitivity improvement of invMap-SVision over Winnowmap2, minimap2, NGMLR and lra pipelines falls in the 2.5%–15.49% range
- On the HG002 PacBio real-read benchmark, invMap accuracy reaches 76.55% vs. minimap2 75.15%, NGMLR 73.4%, Winnowmap2 67.10% and lra 63.25%
- Each mapper's SAM converts to a sorted BAM and SVision emits a valid VCF call set per mapper
- Correct-call counts are computed by matching SVision VCF calls against the 1000-inversion SURVIVOR truth set

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- invMap version covered is 1.0.0 (2023/09/08); no changelog is available
- Findings are demonstrated on simulated datasets across different genomes/coverages and the NA12878 real dataset; HG002 benchmark values are reference values to reproduce, not guarantees
- Compiled binary runs on Linux (recommended) or Windows 7/10; approximately 800 M of available disk space is needed

## Evidence

- [abstract] invMap chaining + SVision discovery: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [readme] invMap SAM invocation: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [results] SVision sensitivity improvement: [evidence span withheld — anchor status: not_found]
- [intro] Memory requirement: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [results] SVision call counts: [evidence span withheld — anchor status: not_found]
