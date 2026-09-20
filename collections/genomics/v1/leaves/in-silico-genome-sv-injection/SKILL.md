---
name: in-silico-genome-sv-injection
description: Use when benchmarking somatic SV calling methods (e.g. nanomonsv vs detection-subtraction
  or CAMPHORsomatic) requires ground-truth breakpoints and controlled tumor purity/depth
  scenarios.
when_to_use_negative:
- Evaluating callers on real tumor/control long-read data where a validated experimental
  truth set (e.g. CCLE/hybrid-capture-validated SVs) is available
- Benchmarking germline-only SV callers, since the injected truth set here is designed
  around somatic tumor/control contrast
- Simulating platforms other than ONT (PacBio reads require different simulator profiles)
edam_operation: http://edamontology.org/operation_3202
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: SURVIVOR
  role: simSV module used to inject germline and somatic SVs into GRCh38 to build
    the in-silico control and tumor genomes (v1.0.6)
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: NanoSim
  role: read simulator (v2.6.0) trained on real ONT reads with read_analysis.py, generating
    Nanopore-like tumor/control reads with simulator.py
  repo: https://github.com/BirolLab/NanoSim
- name: minimap2
  role: aligns simulated FASTQ reads to GRCh38 before sub-sampling to target coverage/purity
    points
  repo: https://github.com/lh3/minimap2
- name: nanomonsv
  role: somatic SV caller under benchmark on every sub-sampled tumor/control BAM pair
  repo: https://github.com/friend1ws/nanomonsv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:7c2a6d73fcc8316a3fc42f97618d004a82b90bc1873c0a4a63708440ad2a0ba5
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
  - cuteSV
  - minimap2
  - nanomonsv
  edam_operation: http://edamontology.org/operation_3202
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# in-silico-genome-sv-injection

## Summary

Constructs an in-silico diploid matched-control genome from GRCh38 by injecting germline SVs with SURVIVOR simSV, then injects a somatic SV truth set to create a tumor genome, enabling benchmarking of somatic SV callers on simulated Nanopore data.

## When to use

Use when benchmarking somatic SV calling methods (e.g. nanomonsv vs detection-subtraction or CAMPHORsomatic) requires ground-truth breakpoints and controlled tumor purity/depth scenarios.

## When NOT to use

- Evaluating callers on real tumor/control long-read data where a validated experimental truth set (e.g. CCLE/hybrid-capture-validated SVs) is available
- Benchmarking germline-only SV callers, since the injected truth set here is designed around somatic tumor/control contrast
- Simulating platforms other than ONT (PacBio reads require different simulator profiles)

## Inputs

- GRCh38 reference genome (22 autosomes + chromosome X)
- Real ONT FASTQ/BAM reads for NanoSim model training (e.g. COLO829BL Nanopore reads, ENA PRJEB27698)
- Specified germline and somatic SV counts per class (duplications, indels, inversions, translocations, inversion-deletions, inversion-duplications)

## Outputs

- In-silico diploid matched-control genome with injected germline SVs
- In-silico tumor genome with injected somatic SVs and a GRCh38-coordinate somatic SV truth set
- Simulated Nanopore FASTQ reads for tumor and control (~180 Gb yields)
- Minimap2-aligned BAMs sub-sampled to 10x/20x/30x/40x/50x coverage at 0/20/40/60/80/100% tumor purity

## How to apply

Build a diploid matched-control genome from GRCh38 (22 autosomes + chromosome X) by injecting germline SVs using SURVIVOR v1.0.6 simSV (2500 duplications, 5000 indels, 100 inversions, 50 inversion-deletions, 50 inversion-duplications). Then inject somatic SVs (100 duplications, 200 indels, 100 translocations, 100 inversions) to create the in-silico tumor genome, converting somatic SV coordinates back to GRCh38 to define the truth set. Train NanoSim 2.6.0 parameters on real ONT reads (e.g. COLO829BL Nanopore reads from ENA PRJEB27698 aligned to chromosome 22 with read_analysis.py) and generate Nanopore-like tumor and control reads (~180 Gb yields) with simulator.py. Align simulated FASTQ to GRCh38 with minimap2 and sub-sample tumor/control BAMs to 10x–50x coverage and 0%–100% tumor purity so precision/recall of each caller can be tabulated against the injected truth set.

## Related tools

- **SURVIVOR** (simSV module used to inject germline and somatic SVs into GRCh38 to build the in-silico control and tumor genomes (v1.0.6)) — https://github.com/fritzsedlazeck/SURVIVOR
- **NanoSim** (read simulator (v2.6.0) trained on real ONT reads with read_analysis.py, generating Nanopore-like tumor/control reads with simulator.py) — https://github.com/BirolLab/NanoSim
- **minimap2** (aligns simulated FASTQ reads to GRCh38 before sub-sampling to target coverage/purity points) — https://github.com/lh3/minimap2
- **nanomonsv** (somatic SV caller under benchmark on every sub-sampled tumor/control BAM pair) — https://github.com/friend1ws/nanomonsv

## Examples

```
SURVIVOR simSV 2500 5000 100 50 50 GRCh38.bed diploid_genome.fa; NanoSim read_analysis.py classifier -i COLO829BL.chr22.bam; NanoSim simulator.py genome -r GRCh38.fa -b training_profile
```

## Evaluation signals

- Somatic SV truth set coordinates must map back correctly to GRCh38 after tumor-genome injection
- Simulated yields should approximate ~180 Gb for tumor and control before sub-sampling
- Precision and recall are computed against the injected somatic truth set at every coverage (10x–50x) and purity (0%–100%) grid point
- Validation: nanomonsv precision and recall must be superior to the separate detection-subtraction approach and CAMPHORsomatic, with precision remaining relatively stable while recall decreases at low tumor purity and low sequence yield

## Limitations

- Subclonal SVs present in a minority of cells are still challenging to detect at modest sequencing depth, so recall drops at low tumor purity
- Reference-genome-based simulation cannot capture long repeat expansion events because the reference is unreliable at loci susceptible to VNTR/microsatellite expansions
- Simulation trained on a single chromosome (chromosome 22) of real reads may not fully capture genome-wide ONT error profiles

## Evidence

- [supplementary] simulation workflow: [evidence span withheld — anchor status: not_found]
- [supplementary] NanoSim training: [evidence span withheld — anchor status: not_found]
- [supplementary] sub-sampling grid: [evidence span withheld — anchor status: not_found]
- [results] simulation result: [evidence span withheld — anchor status: not_found]
