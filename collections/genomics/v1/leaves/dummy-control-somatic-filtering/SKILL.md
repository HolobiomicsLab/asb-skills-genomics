---
name: dummy-control-somatic-filtering
description: Use when calling somatic SVs or viral (HBV) integrations from tumor long-read
  data of a cell line or sample lacking a true matched normal, and a suitably sequenced
  unrelated control must be substituted.
when_to_use_negative:
- When a true patient-matched control long-read dataset exists — the paper's design
  expects tumor/control paired data, and a dummy control cannot subtract patient-specific
  germline SVs
- When both breakpoints of an SV lie in poorly characterized repetitive regions —
  the Single breakend SV module cannot detect SVs where both breakpoints are in areas
  where reference genomes are not well-characterized
- When the goal is detecting long repeat expansions (VNTR/microsatellite) — the reference-genome-based
  framework may not capture long repeat expansion events
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: nanomonsv
  role: 'Primary caller: ''parse'' extracts SV-supporting reads from tumor and dummy-control
    BAMs; ''get'' with --control_prefix/--control_bam performs somatic filtering and
    single breakend detection'
  repo: https://github.com/friend1ws/nanomonsv
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform providing the tumor and dummy-control input
    reads used for somatic SV/viral integration detection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:0a80cddc22fbd3a5e555b8945ba770c03cec033affb758a70ada85ef10619bdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - Oxford Nanopore Technologies (ONT)
  - nanomonsv
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dummy-control-somatic-filtering

## Summary

Use a surrogate ('dummy') matched control sample in nanomonsv to filter non-somatic SVs and viral integrations when a true patient-matched control is unavailable, reproducing the paper's detection of 12 HBV integrations in the PRC/PRF/5 liver cancer cell line using BL2009 as the dummy control.

## When to use

Use when calling somatic SVs or viral (HBV) integrations from tumor long-read data of a cell line or sample lacking a true matched normal, and a suitably sequenced unrelated control must be substituted.

## When NOT to use

- When a true patient-matched control long-read dataset exists — the paper's design expects tumor/control paired data, and a dummy control cannot subtract patient-specific germline SVs
- When both breakpoints of an SV lie in poorly characterized repetitive regions — the Single breakend SV module cannot detect SVs where both breakpoints are in areas where reference genomes are not well-characterized
- When the goal is detecting long repeat expansions (VNTR/microsatellite) — the reference-genome-based framework may not capture long repeat expansion events

## Inputs

- Tumor ONT FASTQ (e.g. PromethION R9.4.1, Guppy 3.4.5 base-called)
- Dummy matched-control long-read BAM/FASTQ from an unrelated sample (e.g. BL2009)
- Reference genome FASTQ/FASTA (GDC GRCh38.d1.vd1; CHM13 for repetitive-locus validation)
- Parsed SV-supporting read files: sorted bed/bedpe.gz from 'nanomonsv parse' for both samples

## Outputs

- nanomonsv somatic SV call sets (Canonical and Single breakend results, .result.txt and .vcf)
- Single breakend output files {tumor_prefix}.nanomonsv.sbnd.result.txt and .vcf containing viral integration calls
- Curated list of HBV integration events (12 in PRC/PRF/5), including complex rearrangements with breakpoints at single-base resolution

## How to apply

Align tumor ONT reads (e.g. PRC/PRF/5 PromethION R9.4.1, Guppy 3.4.5 base-called) to the GDC GRCh38.d1.vd1 reference with minimap2 '-ax map-ont -t 8 -p 0.1'. Run 'nanomonsv parse' separately on tumor and dummy-control BAMs to extract SV-supporting reads into sorted bed/bedpe.gz files. Run 'nanomonsv get' with --control_prefix and --control_bam pointing to the dummy control's parsed output, keeping the Single breakend SV module enabled (no --no_single_bnd) so viral integrations accompanied by large deletions or translocations are captured as single breakend calls. Scan {tumor_prefix}.nanomonsv.sbnd.result.txt/.vcf for calls whose contig matches hepatitis B virus; in the paper this yielded 12 HBV integrations in PRC/PRF/5, including the TERT-promoter (chr5–chr13) rearrangement and a chr8 event connected to Human Satellite 2 mapping to chromosome 1 via CHM13. The rationale is that a dummy control still absorbs germline/shared artifacts, though it cannot model patient-specific germline SVs.

## Related tools

- **nanomonsv** (Primary caller: 'parse' extracts SV-supporting reads from tumor and dummy-control BAMs; 'get' with --control_prefix/--control_bam performs somatic filtering and single breakend detection) — https://github.com/friend1ws/nanomonsv
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform providing the tumor and dummy-control input reads used for somatic SV/viral integration detection)

## Examples

```
nanomonsv get --tumor_prefix PRC_PRF5 --control_prefix BL2009 --control_bam BL2009.bam --output_dir out/  # keep Single breakend module enabled (omit --no_single_bnd)
```

## Evaluation signals

- The detected HBV integration count matches the reported 12 integrations in PRC/PRF/5 when BL2009 is used as dummy control
- The integration set contains the previously Illumina-identified integrations plus one novel one
- The single breakend output includes the TERT-promoter (chr5 to chr13 inverted duplication) and the chr8 / Human Satellite 2 (chr1-derived) events
- Each called event is supported by parsed SV-supporting reads present in tumor but filtered against the dummy-control bed/bedpe.gz output
- Calls whose contig sequence aligns to hepatitis B virus appear in {tumor_prefix}.nanomonsv.sbnd.result.txt/.vcf rather than the Canonical SV module output

## Limitations

- Most HBV integrations were only recovered via the Single breakend SV module, which reports events where only one breakpoint is identified — the other side may be unresolved
- Single breakend detection fails when both breakpoints are in poorly characterized repetitive regions of the reference genome
- A dummy control cannot perfectly model the absent true matched control, risking residual germline false positives or over-filtering
- Detailed interpretation of complex SV structures is not fully automated; substantial manual characterization (e.g. CHM13 re-alignment) is required

## Evidence

- [results] nanomonsv identified 12 HBV integrations in PRC/PRF/5, and most of these integrations were identified by the Single breakend SV module.: [evidence span withheld — anchor status: not_found]
- [abstract] Single breakend module detects complex SVs identifiable only by long-reads: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated"
- [abstract] nanomonsv detects somatic SVs using tumor and matched control long-read sequencing data: "We present our novel software, nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution."
- [discussion] Single breakend SV module limitation on repetitive regions: "it cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions"
- [intro] Single breakend SVs where only one breakpoint is identified, e.g. viral integrations: "Single breakend SV module of nanomonsv can identify single breakend SVs where only one breakpoint is identified"
