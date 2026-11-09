---
name: read-depth-and-purity-subsampling
description: Use when evaluating a paired tumor/control long-read SV caller's robustness
  across variable sequencing yield and tumor purity, requiring a simulated truth set
  of somatic SVs.
when_to_use_negative:
- Input is real (non-simulated) tumor/control sequencing data with no truth set —
  precision/recall cannot be computed, so the depth/purity grid has no ground truth
  to score against.
- Evaluating germline SV calling where tumor purity sub-sampling is irrelevant to
  the caller's design.
- Short-read-based SV calling, where ONT-specific NanoSim read simulation and the
  sub-sampling grid do not model the platform error profile.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: SURVIVOR
  role: simSV module injects germline and somatic SVs into the in-silico diploid control
    and tumor genomes
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: NanoSim
  role: read_analysis.py trains ONT read parameters; simulator.py generates Nanopore-like
    tumor/control reads
  repo: https://github.com/BirolLab/NanoSim
- name: minimap2
  role: aligns simulated FASTQ reads to GRCh38 before sub-sampling
  repo: https://github.com/lh3/minimap2
- name: nanomonsv
  role: SV caller under evaluation, run on every subsampled tumor/control BAM pair
  repo: https://github.com/friend1ws/nanomonsv
- name: Sniffles2
  role: baseline 'separate detection and subtraction' caller, run with --minsupport
    1 --non-germline on tumor and control
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: cuteSV
  role: subtraction baseline (min_support 1, max_cluster_bias_INS/DEL 100, diff_ratio_merging
    0.3)
- name: Delly
  role: subtraction baseline run in 'lr' mode on tumor and control separately
- name: SVIM
  role: subtraction baseline run in 'alignment' mode with --skip_genotyping
- name: CAMPHORsomatic
  role: somatic SV baseline run with default settings
  repo: https://github.com/afujimoto/CAMPHORsomatic
- name: biobambam
  role: BAM processing tool used in the alignment/sub-sampling workflow
  repo: https://github.com/gt1/biobambam
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:090051517ef00aa88b084bdb40e06bde56fd56efbe60144b728daabd817c3c33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - CAMPHORsomatic
  - Delly
  - NanoSim
  - SURVIVOR
  - SVIM
  - Sniffles
  - Sniffles2
  - biobambam
  - cuteSV
  - minimap2
  - nanomonsv
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# read-depth-and-purity-subsampling

## Summary

Benchmarking skill that stress-tests somatic SV callers by sub-sampling simulated tumor/control long-read BAM alignments to a grid of sequencing depths and tumor purities, then scoring precision/recall against a simulated truth set.

## When to use

Use when evaluating a paired tumor/control long-read SV caller's robustness across variable sequencing yield and tumor purity, requiring a simulated truth set of somatic SVs.

## When NOT to use

- Input is real (non-simulated) tumor/control sequencing data with no truth set — precision/recall cannot be computed, so the depth/purity grid has no ground truth to score against.
- Evaluating germline SV calling where tumor purity sub-sampling is irrelevant to the caller's design.
- Short-read-based SV calling, where ONT-specific NanoSim read simulation and the sub-sampling grid do not model the platform error profile.

## Inputs

- Simulated tumor and control Nanopore FASTQ read sets (~180 Gb yield) from NanoSim
- BAM alignments of simulated reads to GRCh38 (minimap2)
- Simulated somatic SV truth set (GRCh38 coordinates)
- GRCh38 reference (22 autosomes + chromosome X)

## Outputs

- Grid of sub-sampled tumor/control BAM pairs (10x–50x depth; 0–100% purity)
- Per-method SV call sets (VCF) at each grid point
- Precision and recall tables/curves per method per coverage/purity point (cf. Supplementary Figures S10–S12)

## How to apply

Build an in-silico diploid matched-control genome from GRCh38 (22 autosomes + chromosome X) by injecting germline SVs with SURVIVOR v1.0.6 simSV (2500 duplications, 5000 indels, 100 inversions, 50 inversion-deletions, 50 inversion-duplications), then inject somatic SVs (100 duplications, 200 indels, 100 translocations, 100 inversions) and convert coordinates back to GRCh38. Train NanoSim 2.6.0 parameters with read_analysis.py on COLO829BL Nanopore reads (ENA PRJEB27698) aligned to chromosome 22, then use simulator.py to generate Nanopore-like tumor and control reads at ~180 Gb yields. Align the simulated FASTQ files to GRCh38 with minimap2 and sub-sample the tumor/control BAMs to a grid of 10x, 20x, 30x, 40x and 50x depth and 0%, 20%, 40%, 60%, 80% and 100% tumor purity. Run the SV caller on every tumor/control BAM pair in the grid and compute precision and recall against the simulated somatic SV truth set at each coverage/purity point. The rationale is to reveal the caller's failure modes: expect precision to remain relatively stable while recall drops at low tumor purity and low sequence yield.

## Related tools

- **SURVIVOR** (simSV module injects germline and somatic SVs into the in-silico diploid control and tumor genomes) — https://github.com/fritzsedlazeck/SURVIVOR
- **NanoSim** (read_analysis.py trains ONT read parameters; simulator.py generates Nanopore-like tumor/control reads) — https://github.com/BirolLab/NanoSim
- **minimap2** (aligns simulated FASTQ reads to GRCh38 before sub-sampling) — https://github.com/lh3/minimap2
- **nanomonsv** (SV caller under evaluation, run on every subsampled tumor/control BAM pair) — https://github.com/friend1ws/nanomonsv
- **Sniffles2** (baseline 'separate detection and subtraction' caller, run with --minsupport 1 --non-germline on tumor and control) — https://github.com/fritzsedlazeck/Sniffles
- **cuteSV** (subtraction baseline (min_support 1, max_cluster_bias_INS/DEL 100, diff_ratio_merging 0.3))
- **Delly** (subtraction baseline run in 'lr' mode on tumor and control separately)
- **SVIM** (subtraction baseline run in 'alignment' mode with --skip_genotyping)
- **CAMPHORsomatic** (somatic SV baseline run with default settings) — https://github.com/afujimoto/CAMPHORsomatic
- **biobambam** (BAM processing tool used in the alignment/sub-sampling workflow) — https://github.com/gt1/biobambam

## Examples

```
nanosim/src/read_analysis.py read -i COLO829BL_chr22.bam -o nanosim_model; nanosim/src/simulator.py read nanosim_model -c 180 -o sim_reads
```

## Evaluation signals

- nanomonsv's precision and recall must be superior to the separate detection-subtraction baselines (Sniffles2, cuteSV, Delly lr, SVIM) and to CAMPHORsomatic at every grid point
- Precision should remain relatively stable even at low tumor purities and low sequence yields
- Recall is expected to drop at low tumor purity and low sequence yield — a flat recall at 20x/0% purity suggests sub-sampling or truth-set conversion errors
- Somatic SV coordinate count matches the injected truth (100 duplications, 200 indels, 100 translocations, 100 inversions) after conversion back to GRCh38
- Results reproduce the structure of Supplementary Figures S10–S12 when tabulated per coverage/purity point

## Limitations

- Recall drops at low tumor purities and low sequence yields even for the best caller, so subclonal SVs present in a minority of cells remain hard to detect at modest sequencing depth
- The simulation uses NanoSim ONT profiles trained on a single chromosome-22 subset (COLO829BL), so simulated error profiles may not fully represent all ONT or PacBio data
- Results depend on the subtraction baseline's ad hoc parameters (e.g. 200 bp overlap margins for removing tumor calls overlapping control calls)

## Evidence

- [supplementary] On simulation data with varying tumor purity and sequence yields, nanomonsv showed superior precision and recall compared to other approaches: [evidence span withheld — anchor status: not_found]
- [supplementary] precision remained stable even at low tumor purities and low sequence yields, where recall dropped: [evidence span withheld — anchor status: not_found]
- [supplementary] Train NanoSim 2.6.0 parameters on COLO829BL Nanopore reads (ENA PRJEB27698) aligned to chromosome 22 with read_analysis.py: [evidence span withheld — anchor status: not_found]
- [supplementary] sub-sample the tumor/control BAMs to sequencing amounts of 10x, 20x, 30x, 40x and 50x and tumor purities of 0%, 20%, 40%, 60%, 80% and 100%: [evidence span withheld — anchor status: not_found]
- [intro] separate-detection-and-subtraction approaches for somatic SV calling from tumor/control data generate many false positives: "this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control"
