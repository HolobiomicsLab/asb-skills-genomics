---
name: hbv-rearrangement-structure-characterization
description: Use when ONT PromethION reads from an HBV-positive tumor are paired with
  a matched or dummy control BAM and the goal is to enumerate viral integration sites
  and resolve complex virus-mediated rearrangement structures at single-breakend resolution.
when_to_use_negative:
- When the target is standard diploid somatic SVs with both breakpoints resolvable
  in the reference genome — the nanomonsv Canonical SV module, not the single breakend
  viral-integration workflow, is the right tool.
- When both breakpoints of an event lie in poorly characterized highly repetitive
  regions — the article notes the Single breakend SV module 'cannot detect SVs where
  both of the breakpoints are located in areas where reference genomes are not well-characterized'.
- When no matched (or dummy) control long-read data are available, since nanomonsv
  is designed for paired tumor/control detection and 'nanomonsv get' requires --control_prefix
  and --control_bam.
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0654
tools:
- name: nanomonsv
  role: 'Core caller: ''parse'' extracts SV-supporting reads from tumor/control BAMs;
    ''get'' with the Single breakend SV module (default on; omit --no_single_bnd)
    emits .sbnd.result.txt/.vcf used to find HBV integrations'
  repo: https://github.com/friend1ws/nanomonsv
- name: minimap2
  role: Aligns ONT reads to GDC GRCh38.d1.vd1 with '-ax map-ont -t 8 -p 0.1' to produce
    tumor/control BAMs for nanomonsv
  repo: https://github.com/lh3/minimap2
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read platform whose reads resolve virus-mediated rearrangements that
    short reads miss
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:89804ce1d31fec6a57addc7ac7fd5420432adc2714baeb4d0ec35da90c38f787
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - Oxford Nanopore Technologies (ONT)
  - minimap2
  - nanomonsv
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hbv-rearrangement-structure-characterization

## Summary

Detects HBV integration breakpoints and characterizes their rearrangement structures in tumor long-read (ONT) data using nanomonsv's Single breakend SV module against a matched (or dummy) control, capturing virus-mediated complex events that canonical paired-breakpoint callers miss.

## When to use

Use when ONT PromethION reads from an HBV-positive tumor are paired with a matched or dummy control BAM and the goal is to enumerate viral integration sites and resolve complex virus-mediated rearrangement structures at single-breakend resolution.

## When NOT to use

- When the target is standard diploid somatic SVs with both breakpoints resolvable in the reference genome — the nanomonsv Canonical SV module, not the single breakend viral-integration workflow, is the right tool.
- When both breakpoints of an event lie in poorly characterized highly repetitive regions — the article notes the Single breakend SV module 'cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized'.
- When no matched (or dummy) control long-read data are available, since nanomonsv is designed for paired tumor/control detection and 'nanomonsv get' requires --control_prefix and --control_bam.

## Inputs

- Tumor ONT PromethION FASTQ (R9.4.1 flowcell, Guppy 3.4.5 base-called)
- Matched or dummy-control long-read BAM (e.g. BL2009) with its parsed nanomonsv prefix
- GDC GRCh38.d1.vd1 reference genome (minimap2 index)
- Hepatitis B virus reference sequence for contig/sequence matching of single breakend calls
- CHM13 reference genome for mapping repetitive satellite-sequence origins
- Tumor BAM processed by 'nanomonsv parse' (sorted bed/bedpe.gz outputs)

## Outputs

- Single breakend result table {tumor_prefix}.nanomonsv.sbnd.result.txt
- Single breakend VCF {tumor_prefix}.nanomonsv.sbnd.result.vcf (hmm: article names .sbnd.result.txt and .vcf)
- Set of HBV integration calls whose contig sequence matches HBV (12 integrations in PRC/PRF/5 with BL2009 control)
- Structurally annotated complex events (e.g. chr5 TERT-promoter to chr13 inverted duplication; chr8 to Human Satellite 2 with chr1 origin)

## How to apply

Align ONT FASTQ reads (e.g. PromethION R9.4.1, Guppy 3.4.5 base-called) to the GDC GRCh38.d1.vd1 reference with minimap2 using '-ax map-ont -t 8 -p 0.1'. Run 'nanomonsv parse' separately on the tumor BAM and the control BAM (e.g. BL2009 as a dummy matched control) to extract SV-supporting reads into sorted bed/bedpe.gz files, then run 'nanomonsv get' with --control_prefix and --control_bam pointing at the control parsed output, keeping the Single breakend SV module enabled (do NOT pass --no_single_bnd) because HBV integrations are often accompanied by large deletions and translocations that present as single breakends. Scan the single breakend output ({tumor_prefix}.nanomonsv.sbnd.result.txt and .vcf) for calls whose contig sequence matches hepatitis B virus to obtain the integration set (the reported case yields 12 integrations, including all previously Illumina-identified sites plus one novel one). For highlighted complex events, characterize the inserted human/viral segment structure manually: e.g. the HBV-mediated rearrangement connecting the TERT promoter on chromosome 5 to a chromosome 13 locus forming an inverted duplication, and the chromosome 8 rearrangement joined to a Human Satellite 2 sequence whose origin was resolved to chromosome 1 by aligning to the CHM13 reference genome.

## Related tools

- **nanomonsv** (Core caller: 'parse' extracts SV-supporting reads from tumor/control BAMs; 'get' with the Single breakend SV module (default on; omit --no_single_bnd) emits .sbnd.result.txt/.vcf used to find HBV integrations) — https://github.com/friend1ws/nanomonsv
- **minimap2** (Aligns ONT reads to GDC GRCh38.d1.vd1 with '-ax map-ont -t 8 -p 0.1' to produce tumor/control BAMs for nanomonsv) — https://github.com/lh3/minimap2
- **Oxford Nanopore Technologies (ONT)** (Long-read platform whose reads resolve virus-mediated rearrangements that short reads miss)

## Examples

```
nanomonsv get --control_prefix BL2009 --control_bam BL2009.bam PRC_PRF5 PRC_PRF5.out
```

## Evaluation signals

- The single breakend scan yields exactly the reported 12 HBV integrations for PRC/PRF/5 with BL2009 as dummy control.
- The detected integration set contains all integrations previously identified by Illumina short-read analysis plus one novel integration.
- The chr5 TERT-promoter to chr13 inverted duplication event and the chr8 to Human Satellite 2 event (origin resolved to chromosome 1 via CHM13) are both present.
- The single breakend module was left enabled (no --no_single_bnd passed to 'nanomonsv get'), confirmed by presence of the .sbnd.result.txt/.vcf outputs.
- Alignment was performed with minimap2 '-ax map-ont -t 8 -p 0.1' against GDC GRCh38.d1.vd1, and both tumor and control were parsed separately before 'nanomonsv get'.

## Limitations

- The Single breakend SV module cannot detect SVs where both breakpoints are in poorly characterized, highly repetitive reference regions.
- Interpretation of detailed structure and properties of complex SVs is not fully automated; the TERT and satellite-sequence events required manual characterization and CHM13 alignment.
- No established format yet exists for describing complex SVs, making downstream reporting of these events non-standardized.

## Evidence

- [abstract] Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads"
- [intro] Single breakend SV module of nanomonsv can identify single breakend SVs where only one breakpoint is identified: "Single breakend SV module of nanomonsv can identify single breakend SVs where only one breakpoint is identified"
- [readme] nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data: "nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data"
- [discussion] it cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions: "it cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions"
