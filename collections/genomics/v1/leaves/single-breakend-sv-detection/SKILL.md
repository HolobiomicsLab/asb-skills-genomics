---
name: single-breakend-sv-detection
description: Use when paired tumor and matched-control ONT (or PacBio) long-read data
  must be scanned for somatic SVs with only one resolvable breakpoint, e.g., centromeric/satellite
  rearrangements, LINE1-mediated events, or viral integrations that short-read callers
  miss.
when_to_use_negative:
- Both breakpoints of the SV of interest lie in poorly characterized highly repetitive
  regions where the reference genome is unreliable — the Single breakend SV module
  cannot detect such events
- The goal is to capture long repeat expansion events (VNTR/microsatellite expansions)
  — a reference-genome-based framework is not appropriate for these
- Only short-read data are available — single breakend SVs of this complexity are
  identifiable only by long reads
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_3175
tools:
- name: nanomonsv
  role: 'Primary caller: ''nanomonsv parse'' extracts SV-supporting reads; ''nanomonsv
    get'' with the Single breakend SV module enabled produces the sbnd result files'
  repo: https://github.com/friend1ws/nanomonsv
- name: minimap2
  role: Aligns tumor/control ONT reads to GRCh38.d1.vd1 with the '-ax map-ont' preset
    prior to parsing
  repo: https://github.com/lh3/minimap2
- name: Oxford Nanopore Technologies (ONT)
  role: Source of the long-read sequencing data from which single breakend SVs are
    detected
- name: RepeatMasker
  role: Annotates contig sequences realigned past the breakpoint to classify events
    as alpha satellite, human satellite, or simple repeat bound
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:5a8ac3844d6cbf85723f0fe3df3aca18b4cbd720a95e9a56322db9d17b5d6ea5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - Oxford Nanopore Technologies (ONT)
  - RepeatMasker
  - minimap2
  - nanomonsv
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-breakend-sv-detection

## Summary

Use nanomonsv's Single breakend SV module to detect somatic SVs in which only one breakpoint can be identified — typically because the other end falls in poorly characterized repetitive regions such as centromeric satellite sequences, LINE1-mediated rearrangements, or viral (e.g., HBV) integration sites — from tumor/control paired long-read data.

## When to use

Use when paired tumor and matched-control ONT (or PacBio) long-read data must be scanned for somatic SVs with only one resolvable breakpoint, e.g., centromeric/satellite rearrangements, LINE1-mediated events, or viral integrations that short-read callers miss.

## When NOT to use

- Both breakpoints of the SV of interest lie in poorly characterized highly repetitive regions where the reference genome is unreliable — the Single breakend SV module cannot detect such events
- The goal is to capture long repeat expansion events (VNTR/microsatellite expansions) — a reference-genome-based framework is not appropriate for these
- Only short-read data are available — single breakend SVs of this complexity are identifiable only by long reads

## Inputs

- Tumor ONT FASTQ/BAM aligned to GRCh38.d1.vd1 with minimap2 '-ax map-ont'
- Matched-control (or dummy control, e.g., BL2009) ONT BAM and its 'nanomonsv parse' output
- 'nanomonsv parse' outputs for tumor and control (deletion.bed.gz, insertion.bed.gz, rearrangement.bedpe.gz, bp_info)
- GRCh38 simple-repeat BED file (passed via --simple_repeat_bed)
- Assembled contig sequences from the breakpoint onward (for post hoc realignment/annotation)

## Outputs

- {tumor_prefix}.nanomonsv.sbnd.result.txt per cell line (PASS single breakend SV calls)
- {tumor_prefix}.nanomonsv.sbnd.result.vcf
- Classification of events as bound to alpha satellite, human satellite, or simple repeat sequences (via RepeatMasker annotation)
- Identified viral integration calls (e.g., HBV contig-matching single breakends)

## How to apply

Align tumor and matched-control long reads to the GRCh38.d1.vd1 reference with minimap2 using the ONT preset '-ax map-ont', then run 'nanomonsv parse' separately on each BAM to extract supporting reads for putative SVs. Run 'nanomonsv get' with --control_prefix/--control_bam set to the matched control's parsed output, leaving single breakend detection enabled (do not pass --no_single_bnd); a dummy matched control (e.g., BL2009 for HBV-positive PRC/PRF/5) is acceptable when a true pair is unavailable. The module emits {tumor_prefix}.nanomonsv.sbnd.result.txt (and .vcf) listing PASS single breakend calls; classify each event by realigning the assembled contig sequence after the breakpoint to the reference and annotating with RepeatMasker against the GRCh38 simple-repeat BED (alpha satellite, human satellite, simple repeat classes). For virus-mediated events, scan single breakend output for contig sequences matching the viral genome, and align to the CHM13 reference to trace origins of satellite-connected rearrangements. Rationale: the paired tumor/control design filters germline SVs that would otherwise appear as false positives in tumor-only detection.

## Related tools

- **nanomonsv** (Primary caller: 'nanomonsv parse' extracts SV-supporting reads; 'nanomonsv get' with the Single breakend SV module enabled produces the sbnd result files) — https://github.com/friend1ws/nanomonsv
- **minimap2** (Aligns tumor/control ONT reads to GRCh38.d1.vd1 with the '-ax map-ont' preset prior to parsing) — https://github.com/lh3/minimap2
- **Oxford Nanopore Technologies (ONT)** (Source of the long-read sequencing data from which single breakend SVs are detected)
- **RepeatMasker** (Annotates contig sequences realigned past the breakpoint to classify events as alpha satellite, human satellite, or simple repeat bound)

## Examples

```
nanomonsv get --control_prefix BL2009 --control_bam BL2009.bam --simple_repeat_bed GRCh38.simple_repeat.bed --tumor_prefix PRC_PRF5
```

## Evaluation signals

- COLO829, H2009 and HCC1954 should yield 3, 38 and 50 somatic single breakend SVs respectively (91 total)
- Of the 91 events, 32 should be bound to satellite (23 alpha satellite, 5 human satellite) or simple repeat (4) sequences
- For PRC/PRF/5 with BL2009 as dummy control, expect 12 HBV integrations, mostly detected by the Single breakend SV module
- The HBV integration set must include the TERT-promoter (chr5–chr13 inverted duplication) and chr8–Human Satellite 2 (origin chr1) complex events
- Output files {tumor_prefix}.nanomonsv.sbnd.result.txt and .vcf must exist and contain PASS calls for each analyzed pair

## Limitations

- Cannot detect SVs where both breakpoints are located in poorly characterized, highly repetitive reference regions
- Contig sequence after the breakpoint often realigns poorly; annotation of repetitive bound ends is imperfect and some interpretation remains manual
- Detection of subclonal single breakend events present in a minority of cells is challenging at modest sequencing depth

## Evidence

- [abstract] Single breakend module detects complex long-read-only SVs: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated"
- [intro] Definition of single breakend SVs and their origin in repetitive regions: "Single breakend SV module of nanomonsv can identify single breakend SVs where only one breakpoint is identified"
- [discussion] Workflow includes dedicated single breakend detection: "we have developed a workflow for detecting and classifying single breakend SVs (Single breakend SV module)"
- [results] Expected counts for validation on the three cell lines: [evidence span withheld — anchor status: not_found]
- [results] HBV integrations detected via single breakend calls with a dummy control: [evidence span withheld — anchor status: not_found]
- [discussion] Both-breakpoints-in-repeats limitation: "it cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions"
