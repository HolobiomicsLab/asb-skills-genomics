---
name: viral-integration-breakpoint-identification
description: Use when tumor long-read (ONT/PacBio) sequencing data must be screened
  for viral genome insertions (e.g. HBV) and the other breakpoint of each event lies
  in repetitive or uncharacterized sequence so only single breakends are callable.
when_to_use_negative:
- Both breakpoints of the suspected rearrangement lie in poorly characterized repetitive
  regions — the Single breakend SV module cannot detect SVs there (per the article's
  discussion).
- Long repeat expansion events (VNTR/microsatellite) rather than viral insertions
  — the reference-genome-based framework is not reliable at such loci per the paper's
  limitations.
- Only short-read Illumina data are available — the complex events this skill targets
  are identifiable only by long reads.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0199
tools:
- name: nanomonsv
  role: 'Primary caller: ''nanomonsv parse'' extracts SV-supporting reads from tumor/control
    BAMs and ''nanomonsv get'' with the Single breakend SV module (default; no --no_single_bnd)
    yields viral integration breakpoints at single-base resolution'
  repo: https://github.com/friend1ws/nanomonsv
- name: minimap2
  role: Aligns ONT reads to the GRCh38.d1.vd1 reference using '-ax map-ont -t 8 -p
    0.1' before SV extraction
  repo: https://github.com/lh3/minimap2
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read platform whose reads capture complex virus-mediated rearrangements
    missed by short reads
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:4455e8672961b9cf36cf55ccd7d3435a6a9cd91ff3dbae54cc1c6e4b6856c4fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - Oxford Nanopore Technologies (ONT)
  - minimap2
  - nanomonsv
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# viral-integration-breakpoint-identification

## Summary

Detects and resolves viral integration breakpoints in cancer long-read data by running nanomonsv's Single breakend SV module on tumor ONT reads against a matched (or dummy) control BAM, then scanning single breakend calls for viral contig sequences. In the nanomonsv paper this identified 12 HBV integrations in PRC/PRF/5, including a TERT-promoter rearrangement and a chromosome 8 event tied to Human Satellite 2.

## When to use

Use when tumor long-read (ONT/PacBio) sequencing data must be screened for viral genome insertions (e.g. HBV) and the other breakpoint of each event lies in repetitive or uncharacterized sequence so only single breakends are callable.

## When NOT to use

- Both breakpoints of the suspected rearrangement lie in poorly characterized repetitive regions — the Single breakend SV module cannot detect SVs there (per the article's discussion).
- Long repeat expansion events (VNTR/microsatellite) rather than viral insertions — the reference-genome-based framework is not reliable at such loci per the paper's limitations.
- Only short-read Illumina data are available — the complex events this skill targets are identifiable only by long reads.

## Inputs

- Tumor ONT FASTQ (e.g. PRC/PRF/5 PromethION R9.4.1, Guppy 3.4.5 base-called)
- Matched or dummy control BAM/FASTQ (e.g. BL2009 cell-line)
- Reference genome FASTQ/index (GDC GRCh38.d1.vd1); CHM13 reference for repetitive-origin tracing
- Viral reference sequence (hepatitis B virus) for contig matching
- Tumor and control minimap2 alignment BAMs

## Outputs

- Sorted bed/bedpe.gz files of putative SV-supporting reads from 'nanomonsv parse' (per sample)
- {tumor_prefix}.nanomonsv.sbnd.result.txt single breakend call table
- {tumor_prefix}.nanomonsv.sbnd.vcf single breakend VCF
- List of viral integration breakpoints at single-base resolution (12 HBV integrations in the reference case)

## How to apply

Align ONT FASTQ reads (here PRC/PRF/5 PromethION R9.4.1, Guppy 3.4.5 base-called) to the GDC GRCh38.d1.vd1 reference with minimap2 '-ax map-ont -t 8 -p 0.1'. Run 'nanomonsv parse' separately on the tumor BAM and the control BAM (BL2009 as dummy matched control) to extract SV-supporting reads into sorted bed/bedpe.gz files. Run 'nanomonsv get' with --control_prefix and --control_bam pointing to the control parsed output, keeping the Single breakend SV module enabled (default; do NOT pass --no_single_bnd) so viral integrations accompanied by large deletions and translocations surface as single breakend calls. Scan {tumor_prefix}.nanomonsv.sbnd.result.txt and .vcf for calls whose contig matches the viral sequence (hepatitis B virus), then characterize complex events — e.g. the HBV-mediated chr5 TERT-promoter to chr13 inverted duplication, and a chr8 rearrangement connected to Human Satellite 2 whose origin maps to chromosome 1 by aligning to the CHM13 reference genome.

## Related tools

- **nanomonsv** (Primary caller: 'nanomonsv parse' extracts SV-supporting reads from tumor/control BAMs and 'nanomonsv get' with the Single breakend SV module (default; no --no_single_bnd) yields viral integration breakpoints at single-base resolution) — https://github.com/friend1ws/nanomonsv
- **minimap2** (Aligns ONT reads to the GRCh38.d1.vd1 reference using '-ax map-ont -t 8 -p 0.1' before SV extraction) — https://github.com/lh3/minimap2
- **Oxford Nanopore Technologies (ONT)** (Long-read platform whose reads capture complex virus-mediated rearrangements missed by short reads)

## Examples

```
nanomonsv parse PRC_PRF5.bam --prefix PRC_PRF5; nanomonsv get PRC_PRF5 --control_prefix BL2009 --control_bam BL2009.bam
```

## Evaluation signals

- Reproduces the reported count of 12 HBV integrations in PRC/PRF/5 when BL2009 is used as dummy matched control
- The integration set contains the TERT-promoter (chr5-to-chr13) HBV-mediated rearrangement and the chr8–Human Satellite 2 event described in the paper
- Single breakend outputs {tumor_prefix}.nanomonsv.sbnd.result.txt and .vcf exist, are populated, and include calls whose contig sequence matches hepatitis B virus
- Most reported integrations are represented as single breakend calls (Single breakend SV module), consistent with the paper's finding
- All integrations previously identified by Illumina short-read analysis are recovered, plus the one novel long-read-only integration

## Limitations

- The Single breakend SV module cannot detect SVs where both breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions
- Interpretation of the detailed structure and properties of complex SVs is not fully automated; much of the work (e.g. tracing the Human Satellite 2 origin to chromosome 1 via CHM13) is done manually
- Reference-genome-based framework may not capture long repeat expansion events because the reference genome is not reliable at susceptible locations

## Evidence

- [abstract] Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated"
- [intro] Single breakend SV module of nanomonsv can identify single breakend SVs where only one breakpoint is identified: "Single breakend SV module of nanomonsv can identify single breakend SVs where only one breakpoint is identified"
- [methods] minimap2 '-ax map-ont -t 8 -p 0.1' alignment settings: [evidence span withheld — anchor status: not_found]
- [results] 12 HBV integrations, mostly via the Single breakend SV module: [evidence span withheld — anchor status: not_found]
