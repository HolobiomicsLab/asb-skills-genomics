---
name: long-read-simulation-parameterization
description: Use when building a truth-set benchmark of non-reference TE insertions
  for long-read callers, i.e. when you need simulated PacBio/ONT BAM files with known
  insertion positions, types, and coverage.
when_to_use_negative:
- The goal is to call TE insertions in real (non-simulated) long-read data — sTELLeR
  should be run directly on the BAM/CRAM instead of simulating reads
- Only reference TEs already present in the reference genome are of interest — no
  modified-reference insertion step is needed
- The downstream caller requires short-read data — the 20x PacBio RS II-style long-read
  simulation does not match short-read calling pipelines
tools:
- name: PBSIM3
  role: Simulates long reads from the TE-modified reference at 20x coverage using
    the PacBio RS II error model
- name: minimap2
  role: Aligns the simulated reads back to the unmodified reference genome
- name: samtools
  role: Sorts and indexes the minimap2 alignments into a valid BAM usable as caller
    input
- name: sTELLeR
  role: Downstream TE insertion caller evaluated on the simulated dataset
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: pysam
  role: Dependency of sTELLeR used for BAM handling in the benchmark workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:6e7d3103a0ffe05271e64f914250cca82011a773e4ded064a67a9f194b9b1a06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - PBSIM3
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# long-read-simulation-parameterization

## Summary

Construct a simulated long-read benchmark dataset for evaluating TE insertion callers: insert annotated transposable element sequences into a modified reference, then simulate PacBio reads with PBSIM3 at 20x coverage using an error model matching PacBio RS II reads.

## When to use

Use when building a truth-set benchmark of non-reference TE insertions for long-read callers, i.e. when you need simulated PacBio/ONT BAM files with known insertion positions, types, and coverage.

## When NOT to use

- The goal is to call TE insertions in real (non-simulated) long-read data — sTELLeR should be run directly on the BAM/CRAM instead of simulating reads
- Only reference TEs already present in the reference genome are of interest — no modified-reference insertion step is needed
- The downstream caller requires short-read data — the 20x PacBio RS II-style long-read simulation does not match short-read calling pipelines

## Inputs

- GRCh38 reference genome (GCF_000001405.26) with RepeatMasker annotations
- TE fasta file (e.g. fasta/TEfastasequences.fa) containing Alu, L1, SVA, and HERV sequences
- PBSIM3 with PacBio RS II error model parameters
- minimap2 and samtools for alignment and BAM sorting/indexing

## Outputs

- Modified reference genome with TE sequences inserted at annotated positions
- Truth table of inserted elements (chromosome, position, TE type): 886 Alu, 888 L1, 444 SVA, 452 HERV
- Simulated BAM file at 20x coverage with PacBio RS II-like error rate, sorted and indexed

## How to apply

Extract Alu, L1, and SVA sequences at their annotated RepeatMasker positions from GRCh38 (GCF_000001405.26), together with the HERV sequence AF020092, and insert them into a copy of the reference at the annotated positions while masking the recipient loci. Record every insertion (chromosome, position, TE type) in a truth table sized 886 Alu, 888 L1, 444 SVA, and 452 HERV insertions. Simulate long reads from the modified reference with PBSIM3 at 20x coverage using the PacBio RS II error model. Align the simulated reads back to the unmodified reference with minimap2 and sort/index the alignments into BAM format with samtools; the BAM plus truth table then serve as caller input and benchmark ground truth.

## Related tools

- **PBSIM3** (Simulates long reads from the TE-modified reference at 20x coverage using the PacBio RS II error model)
- **minimap2** (Aligns the simulated reads back to the unmodified reference genome)
- **samtools** (Sorts and indexes the minimap2 alignments into a valid BAM usable as caller input)
- **sTELLeR** (Downstream TE insertion caller evaluated on the simulated dataset) — https://github.com/kristinebilgrav/sTELLeR
- **pysam** (Dependency of sTELLeR used for BAM handling in the benchmark workflow)

## Examples

```
pbsim3 --depth 20 --accuracy pacbio-rsii modified_reference.fa && minimap2 -ax map-pb GRCh38.fa simulated.fastq | samtools sort -o simulated.bam && samtools index simulated.bam
```

## Evaluation signals

- The truth table contains exactly the inserted elements: 886 Alu, 888 L1, 444 SVA, 452 HERV insertions
- The simulated reads yield a valid, sorted BAM usable as caller input
- Simulated coverage matches the target 20x
- Read error rate is consistent with the PacBio RS II error model used in PBSIM3
- A caller run on the simulated BAM recovers insertion positions recorded in the truth table (as with the supplementary testbamTRUTH.txt verification)

## Limitations

- Truth-table positions are inherited from RepeatMasker annotations, so the benchmark reflects annotated, reference-derived TE loci rather than novel insertions
- The simulation uses the PacBio RS II error model; datasets produced by other platforms (e.g. HiFi or ONT) would require different --style settings in downstream calling
- The supplementary simulated test reference is a masked chr22 with removed masked parts, packaged as a zip that must be unzipped (tar -xvzf) to use the correct reference bases

## Evidence

- [other] The TE fasta file was used with PBSIM3 to create a simulated dataset, producing a simulated BAM file with 20× coverage and reads with an error rate similar to those generated using the PacBio RS II.: [evidence span withheld — anchor status: not_found]
- [other] Simulate long reads from the modified reference with PBSIM3 at 20x coverage using the PacBio RS II error model.: [evidence span withheld — anchor status: not_found]
- [other] record each insertion (chromosome, position, TE type) in a truth table comprising 886 Alu, 888 L1, 444 SVA, and 452 HERV insertions: [evidence span withheld — anchor status: not_found]
- [readme] Additionally there is a small simulated bam file containing TE insertions and its reference file (masked chr22 with removed masked parts - zip file.: [evidence span withheld — anchor status: external_doc]
- [readme] correct results can be verified by comparing to the provided file testbamTRUTH.txt: "correct results can be verified by comparing to the provided file testbamTRUTH.txt"
