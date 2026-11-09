---
name: sv-recall-fpr-evaluation
description: Use when comparing SV caller sensitivity/FPR (e.g., NanoSV vs Lumpy vs
  Sniffles) from simulated nanopore reads with a known set of rearrangement breakpoints
  as ground truth.
when_to_use_negative:
- Input is real patient data without a known ground truth — recall/FPR cannot be computed
  without simulated or validated breakpoint truth sets
- Short-read (Illumina) SV calling — the benchmark design, read simulation (NanoSim),
  and NanoSV's split-read clustering are specific to long nanopore reads
- Estimating novel-variant discovery rates in real genomes, where the article instead
  reports conservative filtering with ~75% estimated sensitivity and >95% precision
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: NanoSV
  role: Primary long-read SV caller being benchmarked; identifies split- and gapped-aligned
    reads and clusters read segments by orientation and genomic position to define
    breakpoint junctions
  repo: https://github.com/mroosmalen/nanosv
- name: Lumpy
  role: Comparator SV caller run on the same downsampled BAMs to compute competing
    recall/FPR figures
- name: Sniffles
  role: Comparator SV caller run on the same downsampled BAMs to compute competing
    recall/FPR figures
- name: LAST
  role: Long-read aligner used to map each downsampled simulated read set to the rearranged
    reference before SV calling
- name: NanoSim
  role: Trained on Patient 2 nanopore reads to simulate error-profile-matched long
    reads from the rearranged reference
- name: sambamba
  role: Converts LAST MAF output to SAM and produces coordinate-sorted BAMs as SV
    caller input
  repo: https://github.com/biod/sambamba
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:90d6977dd61c7e63a760117a01976ec17e57f8e405a84c76f195d98591ead872
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - LAST
  - Lumpy
  - NanoSV
  - NanoSim
  - Sniffles
  - sambamba
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# sv-recall-fpr-evaluation

## Summary

Benchmark structural variant callers on simulated nanopore long reads with known ground-truth breakpoints, computing per-caller recall and false positive rate across downsampled coverage levels (1x to 44x).

## When to use

Use when comparing SV caller sensitivity/FPR (e.g., NanoSV vs Lumpy vs Sniffles) from simulated nanopore reads with a known set of rearrangement breakpoints as ground truth.

## When NOT to use

- Input is real patient data without a known ground truth — recall/FPR cannot be computed without simulated or validated breakpoint truth sets
- Short-read (Illumina) SV calling — the benchmark design, read simulation (NanoSim), and NanoSV's split-read clustering are specific to long nanopore reads
- Estimating novel-variant discovery rates in real genomes, where the article instead reports conservative filtering with ~75% estimated sensitivity and >95% precision

## Inputs

- GRCh37 chromosome 1 reference FASTA
- Nanopore reads (e.g., Patient 2 MinION reads) for training a NanoSim read model
- Simulated FASTQ read sets downsampled from 1x to 44x coverage
- Sorted, indexed BAM alignments produced with LAST
- Per-caller VCF call sets from NanoSV, Lumpy, and Sniffles
- Ground-truth breakpoint coordinate list (501 reshuffled breakpoints)

## Outputs

- sv_caller_recall_fpr.csv with per-caller, per-coverage recall and FPR values
- Per-caller, per-coverage VCF call sets
- Ground-truth breakpoint coordinate file

## How to apply

Reshuffle GRCh37 chromosome 1 in silico to create 501 simulated SV breakpoints and record their coordinates as ground truth. Train a NanoSim read model on Patient 2 nanopore reads and simulate long reads from the rearranged reference using that error profile, then downsample from 1x to 44x coverage in discrete increments. Align each read set with LAST (the article authors found LAST alignments give the most accurate results for SV calling with NanoSV, via lastdb/last-train/lastal/last-split, then maf-convert to SAM and sambamba to sorted BAM). Run NanoSV, Lumpy, and Sniffles on every per-coverage BAM to produce per-caller VCF call sets. Match each caller's calls against the 501 ground-truth breakpoints for recall, and against the unshuffled reference for false positive rate, at every coverage level; write the results to sv_caller_recall_fpr.csv.

## Related tools

- **NanoSV** (Primary long-read SV caller being benchmarked; identifies split- and gapped-aligned reads and clusters read segments by orientation and genomic position to define breakpoint junctions) — https://github.com/mroosmalen/nanosv
- **Lumpy** (Comparator SV caller run on the same downsampled BAMs to compute competing recall/FPR figures)
- **Sniffles** (Comparator SV caller run on the same downsampled BAMs to compute competing recall/FPR figures)
- **LAST** (Long-read aligner used to map each downsampled simulated read set to the rearranged reference before SV calling)
- **NanoSim** (Trained on Patient 2 nanopore reads to simulate error-profile-matched long reads from the rearranged reference)
- **sambamba** (Converts LAST MAF output to SAM and produces coordinate-sorted BAMs as SV caller input) — https://github.com/biod/sambamba

## Examples

```
NanoSV -t 8 -s sambamba -b human_hg19.bed -o calls.vcf reads.sorted.bam
```

## Evaluation signals

- NanoSV recall reaches ~99.2% at 27x coverage with a maximum FPR of 1.2% (at 44x)
- Lumpy reproduces ~92.4% recall / 78.8% FPR and Sniffles ~92.6% recall / 97% FPR at comparable conditions
- Ground truth contains exactly 501 simulated breakpoints on reshuffled chromosome 1; every call must be matched within tolerance to these coordinates
- All BAMs are coordinate-sorted and indexed (required for NanoSV, which can be run per chromosome for speed)
- Recall and FPR are computed at each discrete coverage level from 1x to 44x and tabulated in sv_caller_recall_fpr.csv

## Limitations

- The article's real-data pipeline uses extremely stringent post-calling filtering that lowers estimated sensitivity to ~75% despite >95% precision; benchmark numbers assume different, per-caller settings
- Sensitivity is limited by coverage level — known de novo junctions were recovered down to ~16x, so very low coverage points (e.g., 1x) are expected to yield poor recall
- LAST alignment is more compute-intensive than alternative mappers tested for NanoSV (BWA MEM, minimap2, NGMLR)
- Fraction of novel variants is lower than previously reported Pacific BioSciences data (89% vs 14%), partly due to conservative SV calling and post-calling filtering steps

## Evidence

- [readme] LAST gives most accurate results for SV calling with NanoSV: "We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources."
- [readme] NanoSV split-read clustering algorithm: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments to define breakpoint-junctions"
- [discussion] Precision/sensitivity of the SV pipeline: [evidence span withheld — anchor status: not_found]
- [discussion] All de novo junctions recovered at 16x coverage: "We were able to extract all known de novo breakpoint junctions for Patient 1 (Fig. 1), even at relatively low coverage (16×)"
