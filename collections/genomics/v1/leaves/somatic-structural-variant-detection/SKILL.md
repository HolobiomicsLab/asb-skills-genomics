---
name: somatic-structural-variant-detection
description: Use when you have paired tumor/control long-read sequencing (ONT FASTQ
  or BAM) from a cancer genome and need somatic SV calls at single-nucleotide resolution,
  including events hidden in repetitive regions.
when_to_use_negative:
- You lack a matched control sample — the joint tumor/control comparison (control
  filtering, control panel subtraction) is the core of the method and separate detection-then-subtraction
  is known to produce false positives from germline SVs.
- The variant of interest is a somatic VNTR/microsatellite repeat expansion — the
  authors state the reference-genome-based framework 'may not be appropriate to capture
  long repeat expansion events'.
- Both breakpoints of the SV lie in poorly characterized highly repetitive regions
  (e.g. some centromeric sequences) — the Single breakend SV module cannot detect
  SVs where both breakpoints are in areas where the reference genome is not well characterized.
- You only have short-read data — this skill is for ONT/PacBio long-read tumor/control
  pairs.
edam_operation: http://edamontology.org/operation_3229
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: nanomonsv
  role: primary somatic SV caller (Canonical SV and Single breakend SV modules) run
    via 'parse' and 'get' subcommands
  repo: https://github.com/friend1ws/nanomonsv
- name: minimap2
  role: aligns ONT FASTQ reads to GRCh38 with the '-ax map-ont -t 8 -p 0.1' preset
    before SV calling
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: sorts and indexes tumor/control BAMs required as input to nanomonsv
- name: htslib
  role: tabix/bgzip required in PATH for nanomonsv parse bed.gz/bedpe.gz outputs
- name: racon
  role: read correction/polishing in the nanomonsv workflow
  repo: https://github.com/isovic/racon
- name: Sniffles2
  role: subtraction baseline (--minsupport 1 --non-germline) for precision/recall
    comparison
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: CAMPHORsomatic
  role: somatic SV caller baseline run with default settings for benchmarking
  repo: https://github.com/afujimoto/CAMPHORsomatic
- name: SURVIVOR
  role: simSV used to inject germline/somatic SVs into GRCh38 when building the simulation
    truth set
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: NanoSim
  role: simulates ONT-like tumor/control reads (v2.6.0, trained with read_analysis.py)
    for the benchmark
  repo: https://github.com/BirolLab/NanoSim
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:b0ab4a5d4b986ed6e797c8ad3501f2eb7b74d17beda8ea726f34c4cd6e91d808
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - CAMPHORsomatic
  - Delly
  - NanoSim
  - SURVIVOR
  - SVIM
  - Sniffles
  - Sniffles2
  - cuteSV
  - htslib
  - minimap2
  - nanomonsv
  - racon
  - samtools
  edam_operation: http://edamontology.org/operation_3229
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# somatic-structural-variant-detection

## Summary

Detect somatic structural variants (SVs) with single-base resolution by jointly analyzing tumor and matched-control long-read (ONT/PacBio) data with nanomonsv, using its Canonical SV and Single breakend SV modules rather than separate detection-then-subtraction.

## When to use

Use when you have paired tumor/control long-read sequencing (ONT FASTQ or BAM) from a cancer genome and need somatic SV calls at single-nucleotide resolution, including events hidden in repetitive regions.

## When NOT to use

- You lack a matched control sample — the joint tumor/control comparison (control filtering, control panel subtraction) is the core of the method and separate detection-then-subtraction is known to produce false positives from germline SVs.
- The variant of interest is a somatic VNTR/microsatellite repeat expansion — the authors state the reference-genome-based framework 'may not be appropriate to capture long repeat expansion events'.
- Both breakpoints of the SV lie in poorly characterized highly repetitive regions (e.g. some centromeric sequences) — the Single breakend SV module cannot detect SVs where both breakpoints are in areas where the reference genome is not well characterized.
- You only have short-read data — this skill is for ONT/PacBio long-read tumor/control pairs.

## Inputs

- Tumor ONT FASTQ reads (e.g. COLO829, H2009, HCC1954 cell lines; ENA PRJEB27698 for COLO829BL control)
- Matched-control ONT FASTQ reads for the same genome
- GRCh38.d1.vd1 reference genome (FASTA) plus GRCh38 simple-repeat BED file
- Indexed tumor and matched-control BAM files sorted with samtools
- Optional non-matched control panel prefix (e.g. HPRC Nanopore panel)

## Outputs

- deletion, insertion, rearrangement and bp_info sorted bed.gz/bedpe.gz intermediate files from 'nanomonsv parse'
- {tumor_prefix}.nanomonsv.result.txt with an Is_Filter column marking PASS somatic SV calls (e.g. 49, 724 and 748 canonical SVs for COLO829, H2009 and HCC1954)
- Single breakend SV calls, with rescue of canonical SVs (39 rescued across the three cell lines)
- Mobile element insertion classification with 5' truncations, internal inversions, and source sites for 3' transductions

## How to apply

Align ONT reads to the GRCh38.d1.vd1 reference with minimap2 using the '-ax map-ont -t 8 -p 0.1' preset, then sort and index the tumor and control BAMs with samtools. Run 'nanomonsv parse' on each indexed tumor/control BAM pair to produce the deletion, insertion, rearrangement and bp_info sorted bed.gz/bedpe.gz files, then run 'nanomonsv get' with --control_prefix and --control_bam, optionally a non-matched control panel prefix (e.g. an HPRC Nanopore panel) and the GRCh38 simple-repeat BED so deletions/insertions confined within simple repeats and events seen in controls are filtered; select the basecaller-matched quality preset (e.g. --qv10 for Guppy 3.4.5 data). Keep the Single breakend SV module enabled (default) and reclassify single-breakend calls whose opposite breakpoint is unambiguously identified into the canonical SV set. Judge success by Is_Filter=PASS counts per sample and, on simulated truth (NanoSim-generated reads at varying tumor purity/depth), by precision/recall superiority over the separate-detection-and-subtraction baselines (Sniffles2 --minsupport 1 --non-germline, cuteSV --min_support=1, Delly lr, SVIM alignment --skip_genotyping) and CAMPHORsomatic.

## Related tools

- **nanomonsv** (primary somatic SV caller (Canonical SV and Single breakend SV modules) run via 'parse' and 'get' subcommands) — https://github.com/friend1ws/nanomonsv
- **minimap2** (aligns ONT FASTQ reads to GRCh38 with the '-ax map-ont -t 8 -p 0.1' preset before SV calling) — https://github.com/lh3/minimap2
- **samtools** (sorts and indexes tumor/control BAMs required as input to nanomonsv)
- **htslib** (tabix/bgzip required in PATH for nanomonsv parse bed.gz/bedpe.gz outputs)
- **racon** (read correction/polishing in the nanomonsv workflow) — https://github.com/isovic/racon
- **Sniffles2** (subtraction baseline (--minsupport 1 --non-germline) for precision/recall comparison) — https://github.com/fritzsedlazeck/Sniffles
- **CAMPHORsomatic** (somatic SV caller baseline run with default settings for benchmarking) — https://github.com/afujimoto/CAMPHORsomatic
- **SURVIVOR** (simSV used to inject germline/somatic SVs into GRCh38 when building the simulation truth set) — https://github.com/fritzsedlazeck/SURVIVOR
- **NanoSim** (simulates ONT-like tumor/control reads (v2.6.0, trained with read_analysis.py) for the benchmark) — https://github.com/BirolLab/NanoSim

## Examples

```
minimap2 -ax map-ont -t 8 -p 0.1 GRCh38.d1.vd1.fa tumor.fastq | samtools sort -o tumor.sorted.bam && samtools index tumor.sorted.bam && nanomonsv parse tumor.sorted.bam tumor_prefix
```

## Evaluation signals

- Each sample yields a {tumor_prefix}.nanomonsv.result.txt containing an Is_Filter column, with PASS canonical SV counts matching reported values (COLO829=49, H2009=724, HCC1954=748) within reasonable tolerance, including the 39 single-breakend-rescued canonical SVs.
- On NanoSim simulated data (10x–50x coverage, 0%–100% tumor purity), nanomonsv precision and recall must exceed the separate detection-subtraction baselines and CAMPHORsomatic (cf. Supplementary Figures S10–S12).
- Precision should remain relatively stable at low tumor purity and low sequence yield, while recall decreases — a characteristic signature of correct application.
- Filtering invariant: no PASS deletion/insertion is confined entirely within the GRCh38 simple-repeat BED, and no PASS event overlaps the matched-control or control-panel calls within the configured margins (200 bp for baselines).
- Intermediate check: 'nanomonsv parse' produces the four sorted bed.gz/bedpe.gz files (deletion, insertion, rearrangement, bp_info) before 'nanomonsv get' is run.

## Limitations

- Detection of subclonal SVs/MEIs present in a minority of cells is still challenging at modest sequencing depth.
- The reference-genome-based framework may not capture long repeat expansion events (VNTR/microsatellite expansions) because the reference genome is unreliable at those loci.
- Single breakend SV module cannot detect SVs where both breakpoints fall in poorly characterized highly repetitive regions of the reference genome.
- Interpretation of detailed structure/properties of complex SVs is not fully automated and much work is done manually; visualization methods and a standard format for describing complex SVs remain to be established.
- Quality preset must match the basecaller: paper data were basecalled with Guppy 3.4.5 and used the --qv10 preset.

## Evidence

- [abstract] nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution.: "We present our novel software, nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution."
- [abstract] two detection modules: Canonical SV and Single breakend SV: "The current version of nanomonsv includes two detection modules, Canonical SV module, and Single breakend SV module."
- [abstract] superior precision and recall over existing methods: "we demonstrate that Canonical SV module can identify somatic SVs that can be captured by short-read technologies with higher precision and recall than existing methods"
- [intro] motivation against separate detection-and-subtraction: "this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control"
- [discussion] repeat expansions not captured: "the current framework based on the reference genome may not be appropriate to capture long repeat expansion events because the reference genome is not reliable at the location susceptible to these"
- [readme] README scope statement: "nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data."
