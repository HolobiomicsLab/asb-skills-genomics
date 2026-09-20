---
name: long-read-alignment-to-reference-genome
description: Use when paired tumor/control ONT or PacBio FASTQ data (e.g., COLO829/COLO829BL
  from ENA PRJEB27698) must be aligned to GRCh38 before somatic SV detection with
  nanomonsv.
when_to_use_negative:
- Input is short-read sequencing data — minimap2 long-read presets and the nanomonsv
  workflow target ONT/PacBio reads, not Illumina alignments
- Reads are already aligned and position-sorted/indexed BAMs — re-alignment is unnecessary
  and would waste compute
- The target variation is a long repeat expansion event — the article notes the reference-genome-based
  framework may not capture long repeat expansion events because the reference is
  unreliable at those loci
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns ONT and PacBio long reads to the GRCh38 reference genome before SV
    calling
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2 alignment output BAMs for downstream nanomonsv
    input
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:0cbbae5648439148a2d58efe41013a58dd468e0112a22769611b40e254e601d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - PacBio
  - minimap2
  - nanomonsv
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# long-read-alignment-to-reference-genome

## Summary

Aligns tumor and matched-control long-read sequencing data (ONT and PacBio) to a reference genome such as GRCh38 with minimap2, producing sorted, indexed BAMs that are the prerequisite input for somatic SV calling with nanomonsv.

## When to use

Use when paired tumor/control ONT or PacBio FASTQ data (e.g., COLO829/COLO829BL from ENA PRJEB27698) must be aligned to GRCh38 before somatic SV detection with nanomonsv.

## When NOT to use

- Input is short-read sequencing data — minimap2 long-read presets and the nanomonsv workflow target ONT/PacBio reads, not Illumina alignments
- Reads are already aligned and position-sorted/indexed BAMs — re-alignment is unnecessary and would waste compute
- The target variation is a long repeat expansion event — the article notes the reference-genome-based framework may not capture long repeat expansion events because the reference is unreliable at those loci

## Inputs

- ONT FASTQ reads for tumor and matched control (e.g., ERR2752451, ERR2752452)
- PacBio FASTQ reads for tumor and matched control (e.g., ERR2808247, ERR2808248)
- GRCh38 reference genome FASTA

## Outputs

- Sorted, indexed BAM files for each tumor sample
- Sorted, indexed BAM files for each matched control sample
- Aligned tumor/control long-read BAM pairs ready as nanomonsv parse/get input

## How to apply

Download tumor and matched-control long-read FASTQ files (here COLO829 and COLO829BL; ONT runs ERR2752451/ERR2752452 and PacBio runs ERR2808247/ERR2808248 from ENA study PRJEB27698). Align each FASTQ to the GRCh38 reference genome with minimap2, using the ONT preset for Nanopore reads and the PacBio preset for PacBio reads. Sort and index each resulting BAM with samtools so that both the tumor and matched-control BAMs are positionally sorted and queryable. Run alignment separately for each platform's tumor/control pair, because nanomonsv executes its parse and get stages independently on the ONT pair and the PacBio pair. Rationale: nanomonsv requires aligned tumor/control long-read BAMs to call somatic SVs at single-nucleotide resolution, and downstream benchmarking precision (e.g., 87.8%/91.8% on COLO829) depends on this alignment step.

## Related tools

- **minimap2** (Aligns ONT and PacBio long reads to the GRCh38 reference genome before SV calling) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2 alignment output BAMs for downstream nanomonsv input) — https://github.com/samtools/samtools

## Examples

```
minimap2 -ax map-ont GRCh38.fa ERR2752451.fq.gz | samtools sort -o tumor_ont.bam && samtools index tumor_ont.bam
```

## Evaluation signals

- Both tumor and matched-control BAMs exist for each platform (ONT and PacBio) and are position-sorted with accompanying .bai index files
- Alignment quality: the bulk of long reads map to GRCh38 (check samtools flagstat mapped read fraction before proceeding)
- nanomonsv parse/get run without error on the aligned tumor/control pair and produce nanomonsv.result.txt/.vcf call sets
- On COLO829 ONT data, the downstream SV benchmark against the Arora (75 SVs) and Valle-Inclan (58 SVs) truth sets reproduces precision of 87.8% (43/49) and 91.8% (45/49) and recall of 57.3% (43/75) and 77.6% (46/58)

## Limitations

- Breakpoint resolution on noisy long reads has seen little evaluation, so alignment-driven breakpoint precision is imperfect
- The reference-genome-based framework is unreliable for long repeat expansion events (VNTR/microsatellite expansions)
- SVs where both breakpoints fall in poorly characterized highly repetitive regions (e.g., centromeres) cannot be reliably resolved from alignments to the reference

## Evidence

- [abstract] nanomonsv for detecting somatic SVs with paired long-read tumor/control data: "We present our novel software, nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution."
- [intro] ONT and PacBio platforms: "with Oxford Nanopore Technologies (ONT) and PacBio platform"
- [readme] Sniffles README: long-read alignment input requirement: [evidence span withheld — anchor status: external_doc]
