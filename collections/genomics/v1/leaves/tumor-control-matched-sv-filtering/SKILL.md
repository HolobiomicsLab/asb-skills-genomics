---
name: tumor-control-matched-sv-filtering
description: Use when detecting somatic SVs from paired tumor/matched-control long-read
  ONT or PacBio data (e.g. COLO829, H2009, HCC1954 pairs) where germline SVs and control-panel
  events must be filtered before counting canonical somatic SVs.
when_to_use_negative:
- When no matched-control sequencing data exist for the tumor sample — the joint tumor/control
  subtraction strategy (control_prefix/control_bam) requires a matched control
- When the target is long repeat expansion events (VNTR/microsatellite expansions)
  — the reference-genome-based framework may not capture them reliably
- When both SV breakpoints lie in poorly characterized highly repetitive regions (e.g.
  centromeres) — the Single breakend SV module cannot detect SVs there
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: nanomonsv
  role: 'Primary caller: parse tumor/control BAMs and run get with control-panel and
    simple-repeat filtering to produce somatic SV calls'
  repo: github.com/friend1ws/nanomonsv
- name: minimap2
  role: Align ONT FASTQ reads to GRCh38.d1.vd1 using the '-ax map-ont -t 8 -p 0.1'
    preset
  repo: github.com/lh3/minimap2
- name: samtools
  role: Sort and index tumor and control BAMs before nanomonsv parse
  repo: github.com/samtools/samtools
- name: htslib
  role: Supplies tabix/bgzip required in PATH to produce sorted bed.gz/bedpe.gz files
- name: racon
  role: Long-read consensus/polishing support referenced in the nanomonsv workflow
  repo: github.com/isovic/racon
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:dd427c05b2f0ca434f09182799fbf0678dcf2423271db07fdb27f3d447177459
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - htslib
  - minimap2
  - nanomonsv
  - racon
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tumor-control-matched-sv-filtering

## Summary

Jointly analyze tumor and matched-control long-read (ONT/PacBio) alignments with nanomonsv to subtract germline and panel-of-control SVs and call somatic structural variations at single-nucleotide resolution. This is the core tumor/control subtraction strategy behind nanomonsv's Canonical SV and Single breakend SV modules.

## When to use

Use when detecting somatic SVs from paired tumor/matched-control long-read ONT or PacBio data (e.g. COLO829, H2009, HCC1954 pairs) where germline SVs and control-panel events must be filtered before counting canonical somatic SVs.

## When NOT to use

- When no matched-control sequencing data exist for the tumor sample — the joint tumor/control subtraction strategy (control_prefix/control_bam) requires a matched control
- When the target is long repeat expansion events (VNTR/microsatellite expansions) — the reference-genome-based framework may not capture them reliably
- When both SV breakpoints lie in poorly characterized highly repetitive regions (e.g. centromeres) — the Single breakend SV module cannot detect SVs there

## Inputs

- Tumor ONT FASTQ reads (e.g. COLO829, H2009, HCC1954 tumor cell-line sequencing)
- Matched-control ONT FASTQ reads for each tumor/control pair
- GRCh38.d1.vd1 reference genome (minimap2-indexed)
- Non-matched control panel prefix (e.g. HPRC Nanopore panel)
- GRCh38 simple-repeat BED file
- Sorted, indexed tumor and matched-control BAM files

## Outputs

- Deletion, insertion, rearrangement and bp_info sorted bed.gz/bedpe.gz files from 'nanomonsv parse'
- {tumor_prefix}.nanomonsv.result.txt with Is_Filter column and PASS somatic SV calls
- Per-sample canonical SV counts (49, 724, 748 for COLO829, H2009, HCC1954 respectively, including single-breakend-rescued events)

## How to apply

Align tumor and control ONT FASTQ reads to the GRCh38.d1.vd1 reference with minimap2 using the '-ax map-ont -t 8 -p 0.1' preset, then sort/index the BAMs with samtools. Run 'nanomonsv parse' on each indexed tumor and matched-control BAM to produce the deletion, insertion, rearrangement and bp_info sorted bed.gz/bedpe.gz files (requires htslib tabix/bgzip in PATH). Run 'nanomonsv get' per tumor prefix with --control_prefix and --control_bam from the matched control plus a non-matched control panel prefix (e.g. the HPRC Nanopore panel) and the GRCh38 simple-repeat BED, so deletions/insertions confined within simple repeats and events seen in controls are filtered; use the --qv10 preset when data were basecalled with Guppy 3.4.5. Keep the Single breakend SV module (default) enabled and reclassify single-breakend calls whose opposite breakpoint is unambiguously identified into the canonical SV set. Collect the final {tumor_prefix}.nanomonsv.result.txt files and count PASS calls from the Is_Filter column. The rationale is that separate detection-then-subtraction pipelines generate false positives from germline SVs that pass thresholds in tumor but narrowly miss them in matched control; joint analysis with matched-control and panel filtering suppresses these.

## Related tools

- **nanomonsv** (Primary caller: parse tumor/control BAMs and run get with control-panel and simple-repeat filtering to produce somatic SV calls) — github.com/friend1ws/nanomonsv
- **minimap2** (Align ONT FASTQ reads to GRCh38.d1.vd1 using the '-ax map-ont -t 8 -p 0.1' preset) — github.com/lh3/minimap2
- **samtools** (Sort and index tumor and control BAMs before nanomonsv parse) — github.com/samtools/samtools
- **htslib** (Supplies tabix/bgzip required in PATH to produce sorted bed.gz/bedpe.gz files)
- **racon** (Long-read consensus/polishing support referenced in the nanomonsv workflow) — github.com/isovic/racon

## Examples

```
minimap2 -ax map-ont -t 8 -p 0.1 GRCh38.d1.vd1.fa tumor.fastq | samtools sort -o tumor.bam; samtools index tumor.bam; nanomonsv get tumor_prefix --control_prefix ctrl_prefix --control_bam ctrl.bam --qv10
```

## Evaluation signals

- Each sample yields a {tumor_prefix}.nanomonsv.result.txt containing the Is_Filter column
- PASS canonical SV counts match the reported 49, 724 and 748 for COLO829, H2009 and HCC1954 within reasonable tolerance
- Single-breakend-rescued contributions (39 total rescued canonical SVs across the three lines) are included in the PASS counts
- The deletion, insertion, rearrangement and bp_info sorted bed.gz/bedpe.gz files were produced by 'nanomonsv parse' before 'nanomonsv get'
- Control-panel events and deletions/insertions confined within the GRCh38 simple-repeat BED are absent from the PASS set

## Limitations

- Detection of subclonal SVs/MEIs present in a minority of cells is challenging at modest sequencing depth
- The Single breakend SV module cannot detect SVs where both breakpoints are in poorly characterized highly repetitive regions
- Reference-genome-based framework may not capture long repeat expansion events; somatic VNTR/microsatellite expansions are not addressed
- Interpretation of detailed structure of complex SVs is not fully automated and much work is done manually

## Evidence

- [intro] joint matched-control subtraction rationale: "this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control"
- [abstract] nanomonsv purpose: "We present our novel software, nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution."
- [abstract] two detection modules: "The current version of nanomonsv includes two detection modules, Canonical SV module, and Single breakend SV module."
- [results] benchmark counts: [evidence span withheld — anchor status: not_found]
- [discussion] single breakend scope limit: "it cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions"
