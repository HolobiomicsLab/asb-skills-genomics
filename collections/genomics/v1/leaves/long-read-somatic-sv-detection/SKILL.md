---
name: long-read-somatic-sv-detection
description: Use when paired tumor/matched-control long-read sequencing data (ONT
  PromethION FASTQ/BAM or PacBio) must be mined for somatic SVs at single-base resolution,
  especially single breakend calls such as HBV/viral integrations, LINE1-mediated
  rearrangements, or centromeric events.
when_to_use_negative:
- 'Detecting long repeat expansion events (VNTR/microsatellite expansions): the reference-genome-based
  framework is not reliable at susceptible loci, per the authors'' discussion.'
- Detecting SVs where both breakpoints are located in poorly characterized highly
  repetitive regions — the Single breakend SV module requires one breakpoint anchored
  to a characterized reference locus.
- Calling subclonal SVs or mobile element insertions present in a minority of cells
  from modest-depth long-read data — the authors state this remains challenging.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: nanomonsv
  role: Primary paired tumor/control long-read SV caller; provides the Canonical SV
    module and the Single breakend SV module used to capture viral integrations and
    other single-breakpoint events
  repo: https://github.com/friend1ws/nanomonsv
- name: minimap2
  role: Aligns ONT long reads to the reference genome with '-ax map-ont' settings
    before nanomonsv parse/get
  repo: https://github.com/lh3/minimap2
- name: Sniffles
  role: Existing long-read SV caller used as a comparison method for benchmarking
    nanomonsv's precision and recall
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: CAMPHORsomatic
  role: Existing somatic long-read SV caller used as a comparison method in the evaluation
  repo: https://github.com/afujimoto/CAMPHORsomatic
- name: biobambam
  role: BAM processing utilities used in the read-processing pipeline
  repo: https://github.com/gt1/biobambam
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:92b65b317e7ad654c02bfa7a697cb5761ed3547290caec41de581779d54c049b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - CAMPHORsomatic
  - Sniffles
  - biobambam
  - minimap2
  - nanomonsv
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# long-read-somatic-sv-detection

## Summary

Detects somatic structural variations at single-nucleotide resolution from paired tumor/control long-read (ONT/PacBio) cancer genome data using nanomonsv's Canonical SV and Single breakend SV modules, including complex events like viral integrations and centromeric rearrangements invisible to short reads.

## When to use

Use when paired tumor/matched-control long-read sequencing data (ONT PromethION FASTQ/BAM or PacBio) must be mined for somatic SVs at single-base resolution, especially single breakend calls such as HBV/viral integrations, LINE1-mediated rearrangements, or centromeric events.

## When NOT to use

- Detecting long repeat expansion events (VNTR/microsatellite expansions): the reference-genome-based framework is not reliable at susceptible loci, per the authors' discussion.
- Detecting SVs where both breakpoints are located in poorly characterized highly repetitive regions — the Single breakend SV module requires one breakpoint anchored to a characterized reference locus.
- Calling subclonal SVs or mobile element insertions present in a minority of cells from modest-depth long-read data — the authors state this remains challenging.

## Inputs

- Paired tumor and matched-control ONT or PacBio long-read FASTQ files (e.g. PromethION R9.4.1, Guppy 3.4.5 base-called)
- BAM alignments of tumor and control reads to a reference genome (e.g. GRCh38.d1.vd1) produced by minimap2
- Reference genome FASTA (optionally extended with viral/viral-integration source sequences, e.g. hepatitis B virus)

## Outputs

- Somatic SV calls at single-base resolution from the Canonical SV module (bedpe/VCF-style results)
- Single breakend SV results: {tumor_prefix}.nanomonsv.sbnd.result.txt and {tumor_prefix}.nanomonsv.sbnd.vcf
- Full-length inserted sequences supporting SV calls, usable to classify mobile element insertions and complex rearrangements (e.g. viral-human junctions)

## How to apply

Align long reads to the reference (e.g. GDC GRCh38.d1.vd1) with minimap2 using '-ax map-ont -t 8 -p 0.1' for ONT R9.4.1 data. Run 'nanomonsv parse' separately on the tumor BAM and matched-control BAM to extract SV-supporting reads into sorted bed/bedpe.gz files. Run 'nanomonsv get' with --control_prefix and --control_bam pointing at the parsed control output, keeping the Single breakend SV module enabled (default; do NOT pass --no_single_bnd) so single-breakpoint events — viral integrations accompanied by large deletions/translocations — are captured. Scan outputs ({tumor_prefix}.nanomonsv.sbnd.result.txt and .vcf) for calls whose contig sequence matches the virus of interest (e.g. hepatitis B virus). Rationale: nanomonsv jointly uses tumor and control data to avoid the false positives of separate-detect-then-subtract approaches, and single breakend calls capture rearrangements whose other breakpoint lies in repetitive regions.

## Related tools

- **nanomonsv** (Primary paired tumor/control long-read SV caller; provides the Canonical SV module and the Single breakend SV module used to capture viral integrations and other single-breakpoint events) — https://github.com/friend1ws/nanomonsv
- **minimap2** (Aligns ONT long reads to the reference genome with '-ax map-ont' settings before nanomonsv parse/get) — https://github.com/lh3/minimap2
- **Sniffles** (Existing long-read SV caller used as a comparison method for benchmarking nanomonsv's precision and recall) — https://github.com/fritzsedlazeck/Sniffles
- **CAMPHORsomatic** (Existing somatic long-read SV caller used as a comparison method in the evaluation) — https://github.com/afujimoto/CAMPHORsomatic
- **biobambam** (BAM processing utilities used in the read-processing pipeline) — https://github.com/gt1/biobambam

## Examples

```
nanomonsv get PRC_PRF5 --control_prefix BL2009 --control_bam BL2009.sorted.bam
```

## Evaluation signals

- Reproduction benchmark: applied to PRC/PRF/5 with BL2009 as a dummy matched control, the pipeline should recover exactly 12 HBV integrations, mostly via the Single breakend SV module.
- File-existence check: {tumor_prefix}.nanomonsv.sbnd.result.txt and .vcf must be produced after 'nanomonsv get' runs with the Single breakend module enabled (no --no_single_bnd).
- Known-event recovery: the call set must contain the TERT-promoter (chr5) to chr13 HBV-mediated rearrangement with inverted duplication and the chr8 rearrangement linked to a Human Satellite 2 sequence mapping to chromosome 1 on CHM13.
- Control subtraction sanity: germline SVs present in the matched control must be removed, since joint tumor/control analysis is designed to avoid false positives from threshold-passing germline events.
- Resolution check: called breakpoints should reach single-nucleotide resolution, consistent with the paper's stated precision of somatic SV detection.

## Limitations

- Detection of subclonal SVs/MEIs present in a minority of cells is still challenging with modest sequencing depth.
- The Single breakend SV module cannot detect SVs where both breakpoints are in poorly characterized, highly repetitive regions.
- The reference-genome-based framework may not capture long repeat expansion events (VNTR/microsatellites).
- Interpretation of detailed structure of complex SVs is not fully automated and much of the work is manual; visualization methods are still to be developed, and no established format exists for describing complex SVs.

## Evidence

- [abstract] Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated"
- [intro] identify somatic SVs with single-nucleotide resolution jointly using both tumor and control long-read sequencing data: "identify somatic SVs with single-nucleotide resolution jointly using both tumor and control long-read sequencing data"
- [discussion] we have developed a workflow for detecting and classifying single breakend SVs (Single breakend SV module): "we have developed a workflow for detecting and classifying single breakend SVs (Single breakend SV module)"
- [readme] nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data.: "nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data."
