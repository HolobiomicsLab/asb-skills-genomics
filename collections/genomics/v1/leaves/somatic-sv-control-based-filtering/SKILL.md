---
name: somatic-sv-control-based-filtering
description: Use when calling somatic structural variations from paired tumor and
  matched-control long-read sequencing data and you need to suppress germline SV false
  positives at single-nucleotide breakpoint resolution.
when_to_use_negative:
- Only one sample (tumor without a matched control) is available — the joint tumor/control
  comparison that suppresses germline false positives cannot be performed
- Both breakpoints of the SV of interest lie in poorly characterized, highly repetitive
  regions — the Single breakend SV module cannot detect SVs where both breakpoints
  are in such areas
- The goal is detecting long repeat expansion events (VNTR/microsatellite expansions)
  — the reference-genome-based framework may not capture these
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: nanomonsv
  role: Joint tumor/control somatic SV detection with single-base resolution; 'parse'
    extracts supporting reads per sample, 'get' performs the paired-control filtering
  repo: github.com/friend1ws/nanomonsv
- name: minimap2
  role: Aligns ONT reads of tumor and control to the reference genome with the '-ax
    map-ont' preset
  repo: lh3/minimap2
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform providing the tumor/control read data
- name: Sniffles
  role: Long-read SV caller used as a comparison method against nanomonsv's joint
    approach
  repo: fritzsedlazeck/Sniffles
- name: CAMPHORsomatic
  role: Somatic SV caller for long reads used as an existing-method comparator
  repo: afujimoto/CAMPHORsomatic
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:3cd54e6f7f000112a52a84c5e09fffb1b6178293a945e16963d4a69effaba37c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMPHORsomatic
  - Oxford Nanopore Technologies (ONT)
  - Sniffles
  - minimap2
  - nanomonsv
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# somatic-sv-control-based-filtering

## Summary

Jointly analyzing tumor and matched-control long-read (ONT/PacBio) sequencing data so that somatic SV calls are distinguished from germline SVs, avoiding the false positives produced by detecting SVs in the tumor alone and subtracting control calls afterwards.

## When to use

Use when calling somatic structural variations from paired tumor and matched-control long-read sequencing data and you need to suppress germline SV false positives at single-nucleotide breakpoint resolution.

## When NOT to use

- Only one sample (tumor without a matched control) is available — the joint tumor/control comparison that suppresses germline false positives cannot be performed
- Both breakpoints of the SV of interest lie in poorly characterized, highly repetitive regions — the Single breakend SV module cannot detect SVs where both breakpoints are in such areas
- The goal is detecting long repeat expansion events (VNTR/microsatellite expansions) — the reference-genome-based framework may not capture these

## Inputs

- Tumor long-read sequencing data (ONT or PacBio; e.g. FASTQ/BAM from BioProject PRJDB10898)
- Matched-control long-read sequencing data for the same patient/pair
- Reference genome (GRCh38.d1.vd1) alignment BAMs for tumor and control
- GRCh38 simple-repeat BED file (passed via --simple_repeat_bed)

## Outputs

- Somatic SV calls with single-base resolution: deletion, insertion, rearrangement, bp_info bed.gz outputs from 'nanomonsv parse'
- {tumor_prefix}.nanomonsv.sbnd.result.txt — single breakend SV results per cell line
- Somatic SV counts per tumor/control pair validated against benchmark cell lines

## How to apply

Align both tumor and matched-control reads to the reference (e.g. GRCh38.d1.vd1 with minimap2 using the ONT preset '-ax map-ont'), then run 'nanomonsv parse' separately on each tumor BAM and each matched-control BAM to extract supporting reads for putative SVs (deletion, insertion, rearrangement, bp_info bed.gz outputs). Next run 'nanomonsv get' with --control_prefix/--control_bam set to the matched control so candidate SVs are jointly compared rather than called in the tumor and subtracted afterwards; the rationale is that the naive separate-detection-and-subtraction approach generates many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control. Keep single breakend detection enabled (omit --no_single_bnd) if single breakend SVs are of interest, producing {tumor_prefix}.nanomonsv.sbnd.result.txt, and pass --simple_repeat_bed with the GRCh38 simple-repeat BED for repeat annotation. Validate output counts against known benchmark sets (e.g. the COLO829, H2009 and HCC1954 pairs from BioProject PRJDB10898) and compare precision/recall against callers such as Sniffles or CAMPHORsomatic.

## Related tools

- **nanomonsv** (Joint tumor/control somatic SV detection with single-base resolution; 'parse' extracts supporting reads per sample, 'get' performs the paired-control filtering) — github.com/friend1ws/nanomonsv
- **minimap2** (Aligns ONT reads of tumor and control to the reference genome with the '-ax map-ont' preset) — lh3/minimap2
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform providing the tumor/control read data)
- **Sniffles** (Long-read SV caller used as a comparison method against nanomonsv's joint approach) — fritzsedlazeck/Sniffles
- **CAMPHORsomatic** (Somatic SV caller for long reads used as an existing-method comparator) — afujimoto/CAMPHORsomatic

## Examples

```
nanomonsv get --control_prefix control --control_bam control.bam --simple_repeat_bed simpleRepeat.bed --no_control_validated_insertion seq_error_tested tumor_prefix; minimap2 -ax map-ont GRCh38.d1.vd1.fa reads.fq
```

## Evaluation signals

- On the COLO829/H2009/HCC1954 pairs, single breakend counts should reproduce 3, 38 and 50 (91 total) with 23 alpha-satellite-, 5 human-satellite- and 4 simple-repeat-bound events
- The Canonical SV module should show higher precision and recall than existing methods (e.g. Sniffles, CAMPHORsomatic) on simulated and benchmark data
- Germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control should not appear in the somatic callset (no separate-detection-and-subtraction artifacts)
- Output files must include the full set expected from 'nanomonsv parse' (deletion, insertion, rearrangement, bp_info bed.gz) and {tumor_prefix}.nanomonsv.sbnd.result.txt when single breakend detection is enabled
- Somatic breakpoints should be reported at single-nucleotide resolution, verifiable against validated SV sequences

## Limitations

- The separate-detection-and-subtraction strategy used by some approaches generates many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control — one reason joint calling is required
- Detection of subclonal SVs/MEIs present in a minority of cells is still challenging with a modest sequencing depth
- The framework cannot detect SVs where both breakpoints are located in poorly characterized, highly repetitive regions of the reference genome
- The reference-genome-based framework may not be appropriate to capture long repeat expansion events (somatic VNTR/microsatellite expansions)

## Evidence

- [readme] nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data.: "nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data."
- [intro] separate-detection-and-subthesis false positives: "this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control"
- [intro] single-nucleotide resolution objective: "identify somatic SVs with single-nucleotide resolution jointly using both tumor and control long-read sequencing data"
- [abstract] Single breakend SV module scope: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated"
- [discussion] repeat-region detection limit: "it cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions"
