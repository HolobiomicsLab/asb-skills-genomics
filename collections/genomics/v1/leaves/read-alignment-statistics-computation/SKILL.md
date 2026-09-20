---
name: read-alignment-statistics-computation
description: Use when you have corrected or raw long reads (FASTA/FASTQ) and a reference
  genome and need quantitative correction-quality metrics such as aligned fraction,
  alignment identity, and read N50.
when_to_use_negative:
- When the goal is assembly-level evaluation (contig coverage of the reference, unaligned
  contigs) rather than per-read alignment statistics — the paper notes CONSENT's own
  assembly covers a smaller reference proportion and that analysis is separate.
- When polishing assemblies rather than correcting reads; CONSENT-polish output should
  be evaluated with assembly-level metrics instead.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0797
tools:
- name: minimap2
  role: Aligns corrected long reads back to the human reference genome to enable computation
    of aligned fraction, alignment identity, and corrected-read N50; also used internally
    by CONSENT for overlap computation.
  repo: https://github.com/morispi/CONSENT
- name: CONSENT
  role: Produces the corrected reads whose alignment statistics are computed; run
    via CONSENT-correct with --type ONT for nanopore data.
  repo: https://github.com/morispi/CONSENT
- name: ONT
  role: Sequencing technology of the ultra-long human reads (up to 1.5 Mbp, PRJEB23027,
    113 Gbp) whose corrected alignments are evaluated.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:da651a405b9635f533dae7879132daba4ee7c5de3f38315110b636c402551855
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - ONT
  - minimap2
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# read-alignment-statistics-computation

## Summary

Compute alignment-based quality statistics (fraction of aligned reads, alignment identity, corrected-read N50) by mapping corrected long reads back to a reference genome with minimap2, as done to validate CONSENT-corrected ONT reads against Table 5 of the paper.

## When to use

Use when you have corrected or raw long reads (FASTA/FASTQ) and a reference genome and need quantitative correction-quality metrics such as aligned fraction, alignment identity, and read N50.

## When NOT to use

- When the goal is assembly-level evaluation (contig coverage of the reference, unaligned contigs) rather than per-read alignment statistics — the paper notes CONSENT's own assembly covers a smaller reference proportion and that analysis is separate.
- When polishing assemblies rather than correcting reads; CONSENT-polish output should be evaluated with assembly-level metrics instead.

## Inputs

- Corrected long reads in FASTA format (output of CONSENT-correct)
- Raw ONT/PacBio long reads in FASTA or FASTQ format (baseline comparison)
- Reference genome (e.g. human reference) in FASTA format

## Outputs

- Fraction of corrected reads aligned to the reference (~98.4% for the full human ONT dataset)
- Average alignment identity of corrected reads (~93.86%)
- Corrected-read N50 (12,880 bp on the full human dataset)
- minimap2 alignment file (e.g. PAF/SAM) of corrected reads vs. reference

## How to apply

After self-correction (e.g. with CONSENT-correct --type ONT), align the corrected reads to the human reference genome using minimap2. From the resulting alignments, compute three metrics: (1) the fraction of corrected reads that align to the reference, (2) the average alignment identity of the aligned reads, and (3) the N50 of the corrected reads. Compare these against the paper's reported full-human-dataset benchmarks — ~98.4% aligned fraction, ~93.86% alignment identity, and a corrected-read N50 of 12,880 bp — and against the raw reads' identity (~93.5% average alignment identity after correction per the source card) to judge whether correction improved read quality without excessive loss of alignable reads.

## Related tools

- **minimap2** (Aligns corrected long reads back to the human reference genome to enable computation of aligned fraction, alignment identity, and corrected-read N50; also used internally by CONSENT for overlap computation.) — https://github.com/morispi/CONSENT
- **CONSENT** (Produces the corrected reads whose alignment statistics are computed; run via CONSENT-correct with --type ONT for nanopore data.) — https://github.com/morispi/CONSENT
- **ONT** (Sequencing technology of the ultra-long human reads (up to 1.5 Mbp, PRJEB23027, 113 Gbp) whose corrected alignments are evaluated.)

## Examples

```
./CONSENT-correct --in longReads.fastq --out correctedReads.fasta --type ONT --tmpdir /mnt/external --nproc 16 && minimap2 -ax map-ont reference.fa correctedReads.fasta > corrected_vs_ref.sam
```

## Evaluation signals

- Aligned fraction of corrected reads should be near ~98.4% for the full human ONT dataset; a large drop indicates reads were corrupted during correction.
- Average alignment identity of corrected reads should reach ~93.86% against the reference; raw-read identity (~93.5% average after correction per the task card) serves as the improvement baseline.
- Corrected-read N50 should remain around 12,880 bp; a much lower N50 indicates excessive read splitting/truncation during correction.
- Runtime and memory should stay near ~10 days and <100 GB RAM (with ~5 TB overlaps file on external storage) when the full pipeline is measured end-to-end as in Table 5.
- Verify minimap2 is available on the PATH and that alignments are computed against the same reference used for the published benchmarks.

## Limitations

- Correction step runtime of CONSENT tends to be higher than other state-of-the-art methods, so end-to-end benchmarking is costly.
- The overlaps file for a full human dataset requires ~5 TB of disk, which must be placed on external storage via --tmpdir.
- The selection of only the N highest-identity overlaps may prove inefficient for long repeated regions, which can affect read quality metrics in repetitive regions.
- Optimal compromise between window size / minimum number of anchors parameters and runtime has not been established, so statistics may vary with parameter tuning.

## Evidence

- [other] Align the corrected reads back to the human reference with minimap2 and compute the fraction of aligned reads, alignment identity, and corrected-read N50.: [evidence span withheld — anchor status: not_found]
- [abstract] CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days.: "CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days."
- [readme] Minimap2 available through you path.: "Minimap2 available through you path."
- [discussion] our strategy of only selecting the N highest identity overlaps might prove inefficient, especially when the length of the repeated regions grows longer.: "our strategy of only selecting the N highest identity overlaps might prove inefficient, especially when the length of the repeated regions grows longer."
