---
name: breakpoint-junction-recall-benchmarking
description: Use when a curated set of validated SV breakpoint junctions exists (e.g.
  40 junctions in Patient 1, 32 in Patient 2) and you need to compute per-caller recall
  for one or more SV callers on the same sequencing data.
when_to_use_negative:
- No validated ground-truth breakpoint junctions exist for the sample, so recall cannot
  be computed and de novo SV catalogs should be reported instead
- The question is caller precision or novel-variant discovery rather than sensitivity
  to a known junction set (the NanoSV pipeline's own precision benchmark is >95% with
  ~75% estimated sensitivity, a different metric)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: NanoSV
  role: Long-read SV caller applied to nanopore BAMs; achieved 40/40 and 29/32 validated
    junction recovery
  repo: https://github.com/mroosmalen/nanosv
- name: LAST
  role: Long-read mapper producing the BAM alignments fed to NanoSV; 'LAST alignments
    give the most accurate results for SV calling with NanoSV'
- name: Lumpy
  role: Alternative long-read SV caller run on the same nanopore BAMs for recall comparison
- name: Sniffles
  role: Alternative long-read SV caller run on the same nanopore BAMs for recall comparison
- name: Manta
  role: Short-read SV caller run on matched Illumina data for comparison
- name: Delly
  role: Short-read SV caller run on matched Illumina data for comparison
- name: sambamba
  role: SAM-to-BAM conversion and coordinate sorting of LAST mapping output
  repo: https://github.com/biod/sambamba
- name: MinION
  role: Nanopore sequencer producing the long-read data at 11-16x coverage used in
    the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:eed709a5680b1737661280612d5b3e3b0391737df19ca02b4ed6a3b339b2a12e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Delly
  - LAST
  - Lumpy
  - Manta
  - MinION
  - NanoSV
  - Sniffles
  - sambamba
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# breakpoint-junction-recall-benchmarking

## Summary

Benchmark the sensitivity (recall) of structural variant callers by intersecting each caller's VCF output against a validated set of de novo breakpoint junctions, as done when comparing NanoSV, Lumpy, and Sniffles on MinION nanopore data against Manta/Delly on matched Illumina data for chromothripsis patients.

## When to use

Use when a curated set of validated SV breakpoint junctions exists (e.g. 40 junctions in Patient 1, 32 in Patient 2) and you need to compute per-caller recall for one or more SV callers on the same sequencing data.

## When NOT to use

- No validated ground-truth breakpoint junctions exist for the sample, so recall cannot be computed and de novo SV catalogs should be reported instead
- The question is caller precision or novel-variant discovery rather than sensitivity to a known junction set (the NanoSV pipeline's own precision benchmark is >95% with ~75% estimated sensitivity, a different metric)

## Inputs

- Sorted, indexed nanopore BAM files (LAST-mapped MinION reads for Patient 1 and Patient 2, EGA study EGAS00001002333)
- Matched Illumina short-read BAM files per patient
- Validated de novo breakpoint-junction sets per patient (Patient 1: 40 junctions; Patient 2: 32 junctions)
- SV caller output VCFs from NanoSV, Lumpy, Sniffles, Manta, and Delly

## Outputs

- Recall table per caller and patient (sv_caller_recall_benchmark.tsv)
- Per-caller recovered-junction counts (e.g. NanoSV 40/40 on Patient 1) and percentage recall (83% Lumpy, 78% Sniffles)

## How to apply

Run each SV caller on the same alignment inputs: NanoSV on MinION nanopore BAMs mapped with LAST (last-split maf-convert pipeline), plus Lumpy and Sniffles on the identical nanopore BAMs, and Manta/Delly on matched Illumina short-read BAMs for comparison. Intersect each caller's SV output with the validated de novo breakpoint-junction sets per patient and compute recall as recovered junctions / total validated junctions (e.g. Patient 1: NanoSV 40/40, Lumpy 33/40, Sniffles 31/40). Write the recall table (e.g. sv_caller_recall_benchmark.tsv) and verify it reproduces the expected per-caller, per-patient values (Patient 2: NanoSV 29/32, Lumpy 9/32, Sniffles 16/32, Manta 19/32, Delly 22/32). The rationale is that split/gapped-aligned long reads cluster the most sensitive breakpoint signal, so caller choice materially changes recall on complex rearrangements like chromothripsis.

## Related tools

- **NanoSV** (Long-read SV caller applied to nanopore BAMs; achieved 40/40 and 29/32 validated junction recovery) — https://github.com/mroosmalen/nanosv
- **LAST** (Long-read mapper producing the BAM alignments fed to NanoSV; 'LAST alignments give the most accurate results for SV calling with NanoSV')
- **Lumpy** (Alternative long-read SV caller run on the same nanopore BAMs for recall comparison)
- **Sniffles** (Alternative long-read SV caller run on the same nanopore BAMs for recall comparison)
- **Manta** (Short-read SV caller run on matched Illumina data for comparison)
- **Delly** (Short-read SV caller run on matched Illumina data for comparison)
- **sambamba** (SAM-to-BAM conversion and coordinate sorting of LAST mapping output) — https://github.com/biod/sambamba
- **MinION** (Nanopore sequencer producing the long-read data at 11-16x coverage used in the benchmark)

## Examples

```
lastal -Q1 -p my-params refdb reads.fastq | last-split | maf-convert -f reference.dict sam /dev/stdin | sambamba view -h -S --format=bam /dev/stdin | sambamba sort /dev/stdin -o reads.sorted.bam && NanoSV -o nanosv_calls.vcf reads.sorted.bam
```

## Evaluation signals

- Recall table matches reported values exactly: Patient 1 NanoSV 40/40, Lumpy 33/40 (83%), Sniffles 31/40 (78%); Patient 2 NanoSV 29/32, Lumpy 9/32, Sniffles 16/32, Manta 19/32, Delly 22/32
- Each caller's VCF is intersected against the full validated junction set (Patient 1: 40; Patient 2: 32) so recall denominators are consistent
- All callers are run on identical BAM inputs (same LAST-mapped nanopore alignments) so the comparison isolates caller, not mapping, effects
- BAM inputs are coordinate-sorted and indexed as NanoSV requires

## Limitations

- Sensitivity is limited by medium coverage (11-16x) and intrinsic nanopore sequencing biases
- Extremely stringent post-calling filtering in the NanoSV pipeline lowers estimated sensitivity to ~75%, so unfiltered comparisons may differ
- Ground truth is limited to validated de novo chromothripsis junctions; genome-wide inherited SV recall is not assessed by this benchmark

## Evidence

- [results] NanoSV identified 100% of the 40 validated de novo breakpoint junctions, whereas Lumpy and Sniffles recovered only 33 (83%) and 31 (78%): [evidence span withheld — anchor status: not_found]
- [abstract] nanopore long reads are superior to short reads with regard to detection of de novo chromothripsis rearrangements: "nanopore long reads are superior to short reads with regard to detection of de novo chromothripsis rearrangements"
- [readme] We found that LAST alignments give the most accurate results for SV calling with NanoSV.: "We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources."
- [readme] This BAM file needs to be coordinate-sorted and indexed.: "This BAM file needs to be coordinate-sorted and indexed."
- [other] interleave workflow and benchmark intersection step: [evidence span withheld — anchor status: not_found]
