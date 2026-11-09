---
name: simple-repeat-region-filtering
description: Use when running nanomonsv on tumor/control paired ONT or PacBio long-read
  data and deletion/insertion calls must be filtered against a GRCh38 simple-repeat
  BED to remove events confined to low-complexity repeats.
when_to_use_negative:
- Investigating repeat expansion events (VNTR/microsatellite expansions) themselves
  — the paper states the reference-genome-based framework is not appropriate for capturing
  long repeat expansion events
- Somatic SVs with breakpoints inside highly repetitive regions like centromeres —
  these belong in the Single breakend SV module, not in the simple-repeat-filtered
  canonical set
- Germline SV calling from a single sample — the filter depends on paired tumor/control
  design
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: nanomonsv
  role: Performs the somatic SV detection and applies the simple-repeat mask, matched-control
    and control-panel filtering in its 'get' step
  repo: https://github.com/friend1ws/nanomonsv
- name: minimap2
  role: Aligns ONT reads to GRCh38.d1.vd1 with the -ax map-ont preset prior to parsing;
    reference-based alignment quality inside repeats drives the filtering rationale
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes tumor/control BAMs used as --control_bam input
- name: htslib
  role: Provides tabix/bgzip needed for the sorted bed.gz/bedpe.gz files consumed
    by nanomonsv get
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:e936781ec02765774e54e4b50c45abbaefac3047162cbe6dc3e08ac4436ff5dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - htslib
  - minimap2
  - nanomonsv
  - racon
  - samtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Simple repeat region filtering

## Summary

Filters somatic deletion and insertion calls from nanomonsv that are confined within simple repeat (VNTR/microsatellite) regions of the GRCh38 reference, using a simple-repeat BED mask in 'nanomonsv get' so that artifacts inside these error-prone loci do not inflate the somatic SV set.

## When to use

Use when running nanomonsv on tumor/control paired ONT or PacBio long-read data and deletion/insertion calls must be filtered against a GRCh38 simple-repeat BED to remove events confined to low-complexity repeats.

## When NOT to use

- Investigating repeat expansion events (VNTR/microsatellite expansions) themselves — the paper states the reference-genome-based framework is not appropriate for capturing long repeat expansion events
- Somatic SVs with breakpoints inside highly repetitive regions like centromeres — these belong in the Single breakend SV module, not in the simple-repeat-filtered canonical set
- Germline SV calling from a single sample — the filter depends on paired tumor/control design

## Inputs

- nanomonsv parse outputs (deletion, insertion, rearrangement sorted .bed.gz/.bedpe.gz and bp_info files)
- Indexed tumor BAM and matched-control BAM aligned to GRCh38.d1.vd1 with minimap2 -ax map-ont
- GRCh38 simple-repeat BED mask file
- Non-matched control panel prefix (e.g. HPRC Nanopore panel)

## Outputs

- {tumor_prefix}.nanomonsv.result.txt with Is_Filter column and PASS canonical SV calls
- Somatic SV set with repeat-confined deletions/insertions removed

## How to apply

During the 'nanomonsv get' step, supply the tumor prefix together with --control_prefix/--control_bam from the matched control and a non-matched control panel prefix (e.g. the HPRC Nanopore panel); also provide the GRCh38 simple-repeat BED file so that deletions and insertions confined entirely within simple-repeat intervals are removed. Rationale: simple repeats and VNTRs confound reference-based alignment of noisy long reads, so calls sitting wholly inside such intervals are prone to be germline artifacts or alignment errors rather than somatic events. Combine this filtering with control-panel subtraction and the --qv10 preset (appropriate because the paper's data were basecalled with Guppy 3.4.5), then read the filtered result from {tumor_prefix}.nanomonsv.result.txt using the Is_Filter column. Keep single-breakend rescued canonical SVs in the final PASS set, since 39 canonical SVs in the COLO829/H2009/HCC1954 runs were recovered that way.

## Related tools

- **nanomonsv** (Performs the somatic SV detection and applies the simple-repeat mask, matched-control and control-panel filtering in its 'get' step) — https://github.com/friend1ws/nanomonsv
- **minimap2** (Aligns ONT reads to GRCh38.d1.vd1 with the -ax map-ont preset prior to parsing; reference-based alignment quality inside repeats drives the filtering rationale) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes tumor/control BAMs used as --control_bam input)
- **htslib** (Provides tabix/bgzip needed for the sorted bed.gz/bedpe.gz files consumed by nanomonsv get)

## Examples

```
nanomonsv get --tumor_prefix tumor --control_prefix control --control_bam control.sorted.bam --panel_prefix hprc_panel --simple_repeat_mask_file simpleRepeat.bed --qv10
```

## Evaluation signals

- Each sample yields a {tumor_prefix}.nanomonsv.result.txt containing an Is_Filter column; only rows marked PASS count as canonical somatic SVs
- PASS canonical SV counts match the reported 49 (COLO829), 724 (H2009) and 748 (HCC1954) within reasonable tolerance
- Single-breakend-rescued canonical SVs (39 total across the three pairs) are included in the final counts
- No deletions or insertions confined within GRCh38 simple-repeat intervals remain in the PASS set
- Events overlapping the matched control BAM or the HPRC Nanopore control panel are absent from the PASS set

## Limitations

- The current reference-genome-based framework may not capture long repeat expansion events because the reference genome is unreliable at locations susceptible to these events
- Single breakend SV module cannot detect SVs where both breakpoints are in poorly characterized, highly repetitive regions
- Filtering is tuned to data basecalled with Guppy 3.4.5 (--qv10); different basecaller versions may need a different quality setting

## Evidence

- [abstract] We present our novel software, nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution.: "We present our novel software, nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution."
- [methods] Run 'nanomonsv get' with control panel and simple-repeat BED filters: [evidence span withheld — anchor status: not_found]
- [methods] so deletions/insertions confined within simple repeats and events seen in controls are filtered; use the --qv10 preset since the paper data were basecalled with Guppy 3.4.5: [evidence span withheld — anchor status: not_found]
