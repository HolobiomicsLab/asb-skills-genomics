---
name: sv-call-count-reproduction
description: Use when validating a somatic SV caller against published cell-line benchmarks
  (COLO829, H2009, HCC1954 tumor/control ONT pairs) and you need exact PASS call counts
  comparable to a reported result.
when_to_use_negative:
- Goal is detecting single breakend SVs or complex SVs where both breakpoints fall
  in poorly characterized repetitive regions (e.g. centromeres) — the canonical count
  only covers canonical SVs and the module cannot detect both-breakpoint-repeat events
- Detecting somatic VNTR/microsatellite repeat expansion events — the reference-genome-based
  framework may not capture long repeat expansion events
- The caller under evaluation is a separate-detect-then-subtract design — the paper
  notes such approaches generate false positives like germline SVs passing threshold
  in tumor but narrowly missing it in the matched control
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3175
tools:
- name: nanomonsv
  role: Core somatic SV caller; 'parse' builds candidate bed.gz/bedpe.gz files and
    'get' filters tumor calls against matched and panel controls to produce the result.txt
    with PASS canonical SVs
  repo: github.com/friend1ws/nanomonsv
- name: minimap2
  role: Aligns ONT FASTQ reads to GRCh38.d1.vd1 using the '-ax map-ont -t 8 -p 0.1'
    preset
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2 BAMs prior to nanomonsv parse/get
- name: htslib
  role: tabix/bgzip must be in PATH; nanomonsv parse requires them to write the sorted
    bed.gz/bedpe.gz intermediates
- name: racon
  role: Read-correction tool used in the nanomonsv ecosystem for polishing inserted
    sequences
  repo: https://github.com/isovic/racon
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:8afb0b9b05162f489cb55634d210bc51a896ac71d5683440b577ebdbb1907414
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - htslib
  - minimap2
  - nanomonsv
  - racon
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sv-call-count-reproduction

## Summary

Reproduce the canonical somatic SV call counts (49/724/748 for COLO829/H2009/HCC1954) reported by the nanomonsv paper by running the full tumor/control paired ONT detection pipeline, including the 39 single-breakend-rescued canonical SVs.

## When to use

Use when validating a somatic SV caller against published cell-line benchmarks (COLO829, H2009, HCC1954 tumor/control ONT pairs) and you need exact PASS call counts comparable to a reported result.

## When NOT to use

- Goal is detecting single breakend SVs or complex SVs where both breakpoints fall in poorly characterized repetitive regions (e.g. centromeres) — the canonical count only covers canonical SVs and the module cannot detect both-breakpoint-repeat events
- Detecting somatic VNTR/microsatellite repeat expansion events — the reference-genome-based framework may not capture long repeat expansion events
- The caller under evaluation is a separate-detect-then-subtract design — the paper notes such approaches generate false positives like germline SVs passing threshold in tumor but narrowly missing it in the matched control

## Inputs

- ONT FASTQ reads for tumor and matched control of each cell-line pair (COLO829, H2009, HCC1954)
- GRCh38.d1.vd1 reference genome
- Indexed tumor BAM and matched-control BAM per pair
- Non-matched control panel prefix (e.g. HPRC Nanopore panel) BAM files
- GRCh38 simple-repeat BED file

## Outputs

- {tumor_prefix}.nanomonsv.result.txt with Is_Filter column and PASS canonical SV rows
- deletion/insertion/rearrangement/bp_info sorted bed.gz/bedpe.gz intermediate files from 'nanomonsv parse'
- Per-sample PASS canonical SV counts (expected: 49 COLO829, 724 H2009, 748 HCC1954)

## How to apply

Align each tumor/control ONT FASTQ pair to GRCh38.d1.vd1 with minimap2 using the '-ax map-ont -t 8 -p 0.1' preset, sort/index with samtools, then run 'nanomonsv parse' on each pair (requires htslib tabix/bgzip in PATH) to produce deletion/insertion/rearrangement/bp_info sorted bed.gz/bedpe.gz files. Run 'nanomonsv get' per tumor prefix with --control_prefix, --control_bam, a non-matched control panel prefix (e.g. HPRC Nanopore panel), and the GRCh38 simple-repeat BED so deletions/insertions confined to simple repeats and control-panel events are filtered; use the --qv10 preset since the paper data were basecalled with Guppy 3.4.5. Keep the Single breakend SV module enabled (default) and reclassify single-breakend calls whose opposite breakpoint is unambiguously identified into the canonical SV set, recovering 39 rescued canonical SVs across the three lines. Count PASS rows in each {tumor_prefix}.nanomonsv.result.txt (Is_Filter column present) and compare to 49/724/748 within reasonable tolerance.

## Related tools

- **nanomonsv** (Core somatic SV caller; 'parse' builds candidate bed.gz/bedpe.gz files and 'get' filters tumor calls against matched and panel controls to produce the result.txt with PASS canonical SVs) — github.com/friend1ws/nanomonsv
- **minimap2** (Aligns ONT FASTQ reads to GRCh38.d1.vd1 using the '-ax map-ont -t 8 -p 0.1' preset) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2 BAMs prior to nanomonsv parse/get)
- **htslib** (tabix/bgzip must be in PATH; nanomonsv parse requires them to write the sorted bed.gz/bedpe.gz intermediates)
- **racon** (Read-correction tool used in the nanomonsv ecosystem for polishing inserted sequences) — https://github.com/isovic/racon

## Examples

```
nanomonsv get --tumor_prefix tumor --control_prefix control --control_bam control.sorted.bam --panel_prefix hprc_ont --repeat_bed GRCh38_simpleRepeat.bed --qv10
```

## Evaluation signals

- Each sample yields a {tumor_prefix}.nanomonsv.result.txt containing an Is_Filter column
- PASS canonical SV counts match 49 (COLO829), 724 (H2009) and 748 (HCC1954) within reasonable tolerance
- The 39 single-breakend-rescued canonical SVs are included in the totals across the three lines
- The four 'nanomonsv parse' intermediate files (deletion, insertion, rearrangement, bp_info as sorted bed.gz/bedpe.gz) exist per pair
- Simple-repeat-confined deletions/insertions and events present in the matched or panel controls are absent from the PASS set

## Limitations

- Detection of subclonal SVs/MEIs present in a minority of cells is challenging at modest sequencing depth
- Single breakend SV module cannot detect SVs where both breakpoints are in poorly characterized repetitive regions such as centromeres
- The reference-genome-based framework may not capture long repeat expansion events (VNTR/microsatellite expansions)
- Detailed interpretation of complex SV structure is not fully automated and much work is done manually; visualization and standard formats remain open problems

## Evidence

- [results] minimap2 preset and workflow: [evidence span withheld — anchor status: not_found]
- [methods] nanomonsv parse outputs: [evidence span withheld — anchor status: not_found]
- [methods] control filtering: [evidence span withheld — anchor status: not_found]
- [results] single breakend rescue: [evidence span withheld — anchor status: not_found]
- [abstract] module description: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences"
- [intro] false-positive rationale: "this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control"
