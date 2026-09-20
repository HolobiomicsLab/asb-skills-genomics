---
name: sv-haplotype-contig-alignment
description: Use when an assembly/locus-based SV caller (e.g. sawfish discover) emits
  per-breakpoint SV haplotype consensus contigs and you need orthogonal, sequence-level
  evidence that the calls are real and correctly placed.
when_to_use_negative:
- The goal is variant-level precision/recall benchmarking against a truth VCF — use
  truvari bench/refine (with MAFFT refinement) or hap-eval instead of contig-to-assembly
  alignment
- The caller produces only variant records without assembled haplotype contigs — there
  is no contig set to realign, so this validation is inapplicable
- The sample lacks a high-quality diploid assembly or T2T benchmark, leaving no independent
  reference to distinguish expected from spurious contig placements
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sawfish
  role: Discovers SVs and assembles the SV haplotype consensus contigs per candidate
    breakpoint that are subsequently validated (version 0.12.9)
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2
  role: Aligns the SV haplotype contigs to the T2T-HG002-Q100v1.1 assembly (long-read
    preset, 16 threads); also maps the HiFi reads for the original discover run (1.13.1)
- name: spoa
  role: Underlying POA library used inside sawfish to build consensus contigs from
    assigned trimmed reads
  repo: https://github.com/rvaser/spoa
- name: WFA2-lib
  role: Wavefront alignment library used within sawfish's local haplotype assembly
  repo: https://github.com/smarco/WFA2-lib
- name: truvari
  role: Complementary variant-based accuracy assessment (4.2.2 with MAFFT) against
    which the contig-alignment fraction should be consistent
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:aeaa234869de20b19650f63cf3dce145614df5b8c127a9954644d7a4a8e60f90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - WFA2-lib
  - pbmm2
  - sawfish
  - spoa
  - truvari
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sv-haplotype-contig-alignment

## Summary

Validates structural variant (SV) caller output by aligning the assembled SV haplotype consensus contigs back to an independent high-quality diploid assembly (T2T-HG002-Q100v1.1) and computing the fraction that map to their called locus and breakend orientation, reproducing the paper's 99.4% expected-alignment rate for HG002 32x data.

## When to use

Use when an assembly/locus-based SV caller (e.g. sawfish discover) emits per-breakpoint SV haplotype consensus contigs and you need orthogonal, sequence-level evidence that the calls are real and correctly placed.

## When NOT to use

- The goal is variant-level precision/recall benchmarking against a truth VCF — use truvari bench/refine (with MAFFT refinement) or hap-eval instead of contig-to-assembly alignment
- The caller produces only variant records without assembled haplotype contigs — there is no contig set to realign, so this validation is inapplicable
- The sample lacks a high-quality diploid assembly or T2T benchmark, leaving no independent reference to distinguish expected from spurious contig placements

## Inputs

- Assembled SV haplotype consensus contigs from sawfish 0.12.9 discover (one contig set per candidate breakpoint) derived from HG002 Revio HiFi BAM m84011_220902_175841_s1.hifi_reads.bam mapped with pbmm2 1.13.1 --preset CCS to GRCh38
- T2T-HG002-Q100v1.1 diploid assembly (GIAB draft T2T SV benchmark V0.019-20241113) as the alignment target
- Candidate breakpoint coordinates and breakend orientations from the sawfish discover output

## Outputs

- Per-contig pbmm2 alignment records against T2T-HG002-Q100v1.1
- Binary classification of each contig as 'expected' (maps to called locus and breakend orientation, both breakends anchored) or not
- contig_t2t_alignment_fraction.txt containing the fraction of contigs with expected alignments (target ~0.994)

## How to apply

First run sawfish 0.12.9 discover on the pbmm2 1.13.1-mapped HG002 Revio HiFi BAM (GRCh38 human_GRCh38_no_alt_analysis_set reference, --preset CCS) so that candidate breakpoints are assembled into SV haplotype consensus contigs via the POA (spoa) and wavefront-alignment (WFA2-lib) machinery. Next obtain the T2T-HG002-Q100v1.1 diploid assembly underlying the GIAB draft T2T SV benchmark (V0.019-20241113), which serves as a truth reference independent of the GRCh38 alignment used during calling. Align the SV haplotype contigs to the T2T-HG002-Q100v1.1 assembly with pbmm2 align using a long-read preset and 16 threads. Classify each contig alignment as 'expected' only when the contig maps back to the locus and breakend orientation implied by its candidate breakpoint, with both breakends anchored as called; then compute the fraction of contigs meeting this expectation and write it to contig_t2t_alignment_fraction.txt. The rationale is that contigs assembled from real breakpoint-spanning reads should recapitulate the true diploid sequence, so a high direct-alignment fraction (paper value: 99.4% at 32-fold HG002 HiFi depth) corroborates the variant-based precision assessments.

## Related tools

- **sawfish** (Discovers SVs and assembles the SV haplotype consensus contigs per candidate breakpoint that are subsequently validated (version 0.12.9)) — https://github.com/PacificBiosciences/sawfish
- **pbmm2** (Aligns the SV haplotype contigs to the T2T-HG002-Q100v1.1 assembly (long-read preset, 16 threads); also maps the HiFi reads for the original discover run (1.13.1))
- **spoa** (Underlying POA library used inside sawfish to build consensus contigs from assigned trimmed reads) — https://github.com/rvaser/spoa
- **WFA2-lib** (Wavefront alignment library used within sawfish's local haplotype assembly) — https://github.com/smarco/WFA2-lib
- **truvari** (Complementary variant-based accuracy assessment (4.2.2 with MAFFT) against which the contig-alignment fraction should be consistent)

## Examples

```
pbmm2 align -j 16 --preset CCS --sort --unmapped sv_haplotypes.fasta T2T-HG002-Q100v1.1.fasta contigs_to_t2t.bam
```

## Evaluation signals

- The computed fraction in contig_t2t_alignment_fraction.txt reproduces the paper-reported 99.4% contig alignment success rate for HG002 32x data within a small tolerance
- Every 'expected' contig alignment has both breakends anchored at the locus and orientation implied by its candidate breakpoint
- The contig-alignment fraction is consistent with the high precision observed in the truvari-based variant accuracy assessments (truvari 4.2.2 with MAFFT refinement)
- Contig count matches the number of candidate breakpoints emitted by sawfish discover on the 32-fold HG002 HiFi input

## Limitations

- Contigs from wrong or noisy breakpoint candidates will fail the both-breakends-anchored criterion, so the fraction is a lower bound on contig quality
- The validation depends on the accuracy of the T2T-HG002-Q100v1.1 diploid assembly itself; errors in the assembly could misclassify correct contigs
- Results are specific to HG002 Revio HiFi data at ~32-fold coverage; alignment fractions may differ at lower depths or other samples

## Evidence

- [results] 99.4% of sawfish's assembled SV haplotype sequences align directly to the HG002 Q100 v1.1 diploid assembly as expected: [evidence span withheld — anchor status: not_found]
- [methods] The analysis uses pbmm2 1.13.1 for read mapping, and sawfish 0.12.9 for SV calling.: "The analysis uses pbmm2 1.13.1 for read mapping, and sawfish 0.12.9 for SV calling."
- [methods] candidate breakpoints are assembled into SV haplotype contigs, which are then aligned back to the genome: [evidence span withheld — anchor status: not_found]
- [abstract] Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly: "Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy"
- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
