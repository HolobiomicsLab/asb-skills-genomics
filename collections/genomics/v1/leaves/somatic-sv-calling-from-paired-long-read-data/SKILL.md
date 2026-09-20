---
name: somatic-sv-calling-from-paired-long-read-data
description: Use when you have paired tumor/normal long-read data (Oxford Nanopore
  or PacBio FASTQ/BAM) and need somatic SV calls with single-nucleotide breakpoint
  resolution rather than germline-only SV calls from a single sample.
when_to_use_negative:
- The goal is detecting somatic VNTR/microsatellite repeat expansion events — the
  reference-genome-based framework may not reliably capture long repeat expansion
  events
- Both breakpoints of an SV of interest lie in poorly characterized highly repetitive
  regions (e.g. centromeric sequences) — the Single breakend SV module cannot detect
  such SVs
- Only a single sample (no matched control) is available and the aim is germline SV
  calling — use a germline caller such as Sniffles2 instead
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: nanomonsv
  role: Primary somatic SV caller; run as 'parse' then 'get' on paired tumor/control
    long-read BAMs to produce nanomonsv.result.txt/.vcf
  repo: https://github.com/friend1ws/nanomonsv
- name: minimap2
  role: Aligns ONT and PacBio tumor/control reads to the GRCh38 reference before SV
    calling
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2-aligned BAM files
- name: liftOver
  role: Converts truth-set coordinates (e.g. Valle-Inclan truthset_somaticSVs_COLO829.vcf)
    to GRCh38 for benchmark comparison
- name: Sniffles2
  role: Alternative/comparison long-read SV caller for PacBio and ONT data
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: CAMPHORsomatic
  role: Alternative somatic SV caller for long-reads used for comparison
  repo: https://github.com/afujimoto/CAMPHORsomatic
- name: SURVIVOR
  role: Merging and comparing SV call sets when benchmarking against truth sets
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:58cc26f2f96646ae2d598069a0f06310d6e27314cf4d230870ee65c06af6e36d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - CAMPHORsomatic
  - PacBio
  - SURVIVOR
  - Sniffles2
  - liftOver
  - minimap2
  - nanomonsv
  - samtools
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# somatic-sv-calling-from-paired-long-read-data

## Summary

Detect somatic structural variations at single-base resolution by jointly analyzing tumor and matched control long-read sequencing data (ONT and PacBio) with nanomonsv, avoiding the false positives of separate detect-and-subtract approaches.

## When to use

Use when you have paired tumor/normal long-read data (Oxford Nanopore or PacBio FASTQ/BAM) and need somatic SV calls with single-nucleotide breakpoint resolution rather than germline-only SV calls from a single sample.

## When NOT to use

- The goal is detecting somatic VNTR/microsatellite repeat expansion events — the reference-genome-based framework may not reliably capture long repeat expansion events
- Both breakpoints of an SV of interest lie in poorly characterized highly repetitive regions (e.g. centromeric sequences) — the Single breakend SV module cannot detect such SVs
- Only a single sample (no matched control) is available and the aim is germline SV calling — use a germline caller such as Sniffles2 instead

## Inputs

- Tumor long-read FASTQ (ONT or PacBio), e.g. ENA ERR2752451
- Matched control long-read FASTQ, e.g. ENA ERR2752452
- GRCh38 reference genome
- minimap2-aligned, sorted/indexed tumor and control BAM files
- High-confidence somatic SV truth set (bedpe or VCF) for benchmarking

## Outputs

- Somatic SV call set as nanomonsv.result.txt
- Somatic SV call set in VCF format (nanomonsv .vcf)
- Full-length inserted sequences for insertion characterization (5' truncations, internal inversions, target site duplications)
- Per-benchmark precision and recall values (e.g. benchmark_precision_recall.csv)

## How to apply

Download tumor and matched-control long-read FASTQs (e.g. COLO829/COLO829BL from ENA PRJEB27698, ONT runs ERR2752451/ERR2752452 and PacBio runs ERR2808247/ERR2808248), align each to the GRCh38 reference with minimap2, and sort/index the BAMs with samtools. Run nanomonsv in two stages — 'parse' followed by 'get' — separately on each tumor/control pair to emit somatic SV call sets as nanomonsv.result.txt / .vcf files. Joint tumor/control comparison is the rationale: separate detection-and-subtraction pipelines generate false positives from germline SVs that pass thresholds in tumor but narrowly miss them in control. Nanomonsv provides a Canonical SV module for SVs capturable by short reads and a Single breakend SV module for SVs where one breakpoint falls in repetitive regions (LINE1-mediated rearrangements, centromeres, viral integrations). To benchmark, intersect calls with high-confidence truth sets (e.g. Arora bedpe and Valle-Inclan truthset_somaticSVs_COLO829.vcf lifted to GRCh38 with liftOver) by breakpoint overlap, remove insertions/deletions ≤100 bp from truth sets, count novel calls as false positives, and compute precision/recall.

## Related tools

- **nanomonsv** (Primary somatic SV caller; run as 'parse' then 'get' on paired tumor/control long-read BAMs to produce nanomonsv.result.txt/.vcf) — https://github.com/friend1ws/nanomonsv
- **minimap2** (Aligns ONT and PacBio tumor/control reads to the GRCh38 reference before SV calling) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2-aligned BAM files)
- **liftOver** (Converts truth-set coordinates (e.g. Valle-Inclan truthset_somaticSVs_COLO829.vcf) to GRCh38 for benchmark comparison)
- **Sniffles2** (Alternative/comparison long-read SV caller for PacBio and ONT data) — https://github.com/fritzsedlazeck/Sniffles
- **CAMPHORsomatic** (Alternative somatic SV caller for long-reads used for comparison) — https://github.com/afujimoto/CAMPHORsomatic
- **SURVIVOR** (Merging and comparing SV call sets when benchmarking against truth sets) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
minimap2 -ax map-ont GRCh38.fa tumor.fq | samtools sort -o tumor.bam && nanomonsv parse --control_bam control.bam tumor.bam prefix && nanomonsv get prefix
```

## Evaluation signals

- On COLO829 ONT data, reproduced precision against the Arora and Valle-Inclan benchmarks should be 87.8% (43/49) and 91.8% (45/49)
- On COLO829 ONT data, reproduced recall should be 57.3% (43/75) and 77.6% (46/58), matching Figure 3E
- Truth sets should be filtered to remove insertions and deletions of ≤100 bp before intersection
- Novel nanomonsv SVs not overlapping truth sets (6 for Arora, 4 for Valle-Inclan) must be counted as false positives in precision calculation
- Output files nanomonsv.result.txt and .vcf must exist for each tumor/control pair after the 'parse' and 'get' stages

## Limitations

- Detection of subclonal SVs/MEIs present in a minority of cells is still challenging with a modest sequencing depth
- Single breakend SV module cannot detect SVs where both breakpoints lie in poorly characterized highly repetitive regions
- The reference-genome-based framework may not be appropriate to capture long repeat expansion events (VNTR/microsatellite expansions)
- Interpretation of detailed structure and properties of complex SVs is not fully automated and much work is done manually; visualization methods still need to be developed

## Evidence

- [abstract] nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution.: "We present our novel software, nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution."
- [intro] this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control: "this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control"
- [abstract] The current version of nanomonsv includes two detection modules, Canonical SV module, and Single breakend SV module.: "The current version of nanomonsv includes two detection modules, Canonical SV module, and Single breakend SV module."
- [abstract] Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated"
- [discussion] the current framework based on the reference genome may not be appropriate to capture long repeat expansion events because the reference genome is not reliable at the location susceptible to these: "the current framework based on the reference genome may not be appropriate to capture long repeat expansion events because the reference genome is not reliable at the location susceptible to these"
- [discussion] it cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions: "it cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions"
