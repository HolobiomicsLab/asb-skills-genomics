---
name: nanopore-read-simulation
description: 'Use when evaluating SV-caller sensitivity/specificity (NanoSV, Lumpy,
  Sniffles) without wet-lab validation: simulated nanopore reads with a realistic
  error profile and known ground-truth breakpoints must be generated.'
when_to_use_negative:
- You already have real nanopore data with validated breakpoint junctions (as for
  Patient 1's known de novo junctions) — benchmark directly rather than simulating
- Your goal is SNV/short-read variant calling — this skill is specific to long-read
  SV breakpoint detection with split/gapped alignment evidence
- Coverage or read-length distribution of the simulation should mimic a platform other
  than Oxford Nanopore (e.g. Pacific Biosciences SMRT) without retraining the read
  model on that platform's data
tools:
- name: NanoSim
  role: generates simulated nanopore reads from the rearranged reference using an
    error model trained on real Patient 2 MinION reads
- name: LAST
  role: mapper of choice for SV calling with NanoSV; reference indexed with lastdb,
    parameters trained with last-train, alignments produced with lastal and last-split
- name: sambamba
  role: converts MAF-derived SAM to BAM (view -h -S --format=bam) and coordinate-sorts
    it for NanoSV input
  repo: https://github.com/biod/sambamba
- name: NanoSV
  role: primary SV caller under evaluation; identifies split- and gapped-aligned reads
    and clusters them by orientation and position to define breakpoint junctions
  repo: https://github.com/mroosmalen/nanosv
- name: Lumpy
  role: comparison SV caller run on every downsampled BAM
- name: Sniffles
  role: comparison SV caller run on every downsampled BAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:7e418e58201ed9c43d2aa4ccaf40efadf408ae00e91eb2e31d5d5adea0f7ed9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biod/sambamba
  tools:
  - LAST
  - Lumpy
  - NanoSV
  - NanoSim
  - Sniffles
  - sambamba
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nanopore-read-simulation

## Summary

Simulate error-profile-matched nanopore long reads from a designed (rearranged) reference to benchmark SV callers: a NanoSim read model trained on real MinION reads is used to generate synthetic reads over 501 simulated chromosome 1 breakpoints, which are then downsampled (1x–44x) and evaluated by recall and false-positive rate.

## When to use

Use when evaluating SV-caller sensitivity/specificity (NanoSV, Lumpy, Sniffles) without wet-lab validation: simulated nanopore reads with a realistic error profile and known ground-truth breakpoints must be generated.

## When NOT to use

- You already have real nanopore data with validated breakpoint junctions (as for Patient 1's known de novo junctions) — benchmark directly rather than simulating
- Your goal is SNV/short-read variant calling — this skill is specific to long-read SV breakpoint detection with split/gapped alignment evidence
- Coverage or read-length distribution of the simulation should mimic a platform other than Oxford Nanopore (e.g. Pacific Biosciences SMRT) without retraining the read model on that platform's data

## Inputs

- GRCh37 chromosome 1 reference FASTA (unshuffled and in-silico-rearranged versions)
- Real nanopore FASTQ reads (Patient 2 MinION reads) for NanoSim read-model training
- Sample of >10,000 reads for LAST last-train parameter estimation
- Ground-truth breakpoint coordinate list (501 simulated SV junctions)
- Picard sequence dictionary (.dict) for the reference, for MAF-to-SAM conversion

## Outputs

- Simulated nanopore FASTQ read sets at discrete coverage points (1x–44x)
- LAST-aligned MAF files converted to sorted, indexed BAM files
- Per-caller, per-coverage SV VCF call sets (NanoSV, Lumpy, Sniffles)
- sv_caller_recall_fpr.csv with recall and false-positive rate per caller per coverage

## How to apply

First reshuffle GRCh37 chromosome 1 in silico to introduce 501 rearrangement breakpoints and record their coordinates as ground truth. Train a NanoSim read model on real Patient 2 MinION reads so that simulated reads carry the same error profile, then simulate long reads from the rearranged reference. Downsample the simulated FASTQ from 1x to 44x coverage in discrete increments (the paper's own patient data were 11–16x, and all known de novo breakpoint junctions for Patient 1 were recovered at 16x, making 1x–44x a sensible sweep). Align each downsampled set with LAST — which the NanoSV authors found to give the most accurate SV-calling input after last-train parameter training on >10,000 reads — convert MAF to SAM via maf-convert (using a picard sequence dictionary) and to sorted indexed BAM via sambamba. Run NanoSV (plus comparison callers Lumpy and Sniffles) on every BAM, then match calls against the 501 truth breakpoints; compute recall against the rearranged reference and false-positive rate against the unshuffled reference at every coverage level, writing sv_caller_recall_fpr.csv.

## Related tools

- **NanoSim** (generates simulated nanopore reads from the rearranged reference using an error model trained on real Patient 2 MinION reads)
- **LAST** (mapper of choice for SV calling with NanoSV; reference indexed with lastdb, parameters trained with last-train, alignments produced with lastal and last-split)
- **sambamba** (converts MAF-derived SAM to BAM (view -h -S --format=bam) and coordinate-sorts it for NanoSV input) — https://github.com/biod/sambamba
- **NanoSV** (primary SV caller under evaluation; identifies split- and gapped-aligned reads and clusters them by orientation and position to define breakpoint junctions) — https://github.com/mroosmalen/nanosv
- **Lumpy** (comparison SV caller run on every downsampled BAM)
- **Sniffles** (comparison SV caller run on every downsampled BAM)

## Examples

```
last-train -Q1 [referencedb] [reads_sample.fastq] > [my-params] && lastal -Q1 -p [my-params] [referencedb] [reads.fastq] | last-split > [reads.maf]
```

## Evaluation signals

- NanoSV should reproduce 99.2% recall at 27x coverage with a maximum false-positive rate of 1.2% (at 44x) on the 501-breakpoint simulated set
- Comparison callers should land near Lumpy 92.4% recall / 78.8% FPR and Sniffles 92.6% recall / 97% FPR as external sanity anchors
- Every simulated call set must be matched against the same 501 ground-truth breakpoints; FPR is computed against calls on the unshuffled reference
- BAM inputs must be coordinate-sorted and indexed before NanoSV (which requires this), and per-chromosome BAM splitting is permissible for speed on human genomes
- Coverage sweep values must appear as discrete points in sv_caller_recall_fpr.csv spanning 1x to 44x

## Limitations

- Medium coverage and intrinsic nanopore sequencing biases limited sensitivity in the source work; simulated performance may overestimate or underestimate real-patient performance
- LAST mapping 'requires more compute resources' than alternative mappers, per the NanoSV README
- The paper itself did not attempt de novo assembly from MinION reads due to insufficient coverage, so simulated reads support SV calling, not assembly-based validation
- NanoSV defaults target hg19/human (default BED: human_hg19.bed); other genomes need a custom BED for coverage-depth calculations

## Evidence

- [readme] NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data: "NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data, such as data produced by Oxford Nanopore"
- [readme] LAST alignments give the most accurate results for SV calling: "We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources."
- [readme] Train LAST on a read subset before mapping: "Train LAST to get the best scoring parameters for your particular alignment. We typically use a subset of > 10,000 reads for this step"
- [discussion] All known de novo breakpoint junctions for Patient 1 were recovered at 16x coverage: "We were able to extract all known de novo breakpoint junctions for Patient 1 (Fig. 1), even at relatively low coverage (16×)"
- [discussion] SV pipeline precision and sensitivity estimates: [evidence span withheld — anchor status: not_found]
- [readme] This BAM file needs to be coordinate-sorted and indexed: "This BAM file needs to be coordinate-sorted and indexed. Note that if you are performing SV calling on a large genome (e.g. human)"
