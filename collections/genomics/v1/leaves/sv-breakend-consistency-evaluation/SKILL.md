---
name: sv-breakend-consistency-evaluation
description: Use when validating that locally assembled SV haplotype contigs (e.g.
  from sawfish discover on HiFi BAMs) reproduce the expected diploid sequence at called
  breakpoints, independent of variant-based benchmarking.
when_to_use_negative:
- The goal is variant-level benchmarking against a truth VCF (use truvari bench/refine
  or hap-eval instead of contig-to-assembly alignment)
- Input reads are short reads or non-HiFi data, since sawfish's local haplotype assembly
  and this assessment are designed for mapped high-quality HiFi sequencing reads
- No independent high-quality diploid assembly is available to serve as the alignment
  target
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sawfish
  role: discovers candidate breakpoints and assembles SV haplotype consensus contigs
    from mapped HiFi reads via the discover step
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2
  role: aligns the assembled SV haplotype contigs to the T2T-HG002-Q100v1.1 diploid
    assembly (long-read preset, 16 threads) for breakend-consistency classification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:a3bc9f4b9714a61c7c2115a4f8cfd2951c047d11de1c6c265a82458b48417e4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - pbmm2
  - sawfish
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sv-breakend-consistency-evaluation

## Summary

Evaluates the accuracy of assembled structural variant (SV) haplotype consensus contigs by aligning them back to an independent high-quality diploid assembly (T2T-HG002-Q100v1.1) and scoring how many map to the locus and breakend orientation implied by their candidate breakpoints.

## When to use

Use when validating that locally assembled SV haplotype contigs (e.g. from sawfish discover on HiFi BAMs) reproduce the expected diploid sequence at called breakpoints, independent of variant-based benchmarking.

## When NOT to use

- The goal is variant-level benchmarking against a truth VCF (use truvari bench/refine or hap-eval instead of contig-to-assembly alignment)
- Input reads are short reads or non-HiFi data, since sawfish's local haplotype assembly and this assessment are designed for mapped high-quality HiFi sequencing reads
- No independent high-quality diploid assembly is available to serve as the alignment target

## Inputs

- pbmm2 1.13.1-mapped HG002 Revio HiFi BAM (GRCh38 human_GRCh38_no_alt_analysis_set reference, --preset CCS), e.g. m84011_220902_175841_s1.hifi_reads.bam
- T2T-HG002-Q100v1.1 diploid assembly FASTA underlying the GIAB draft T2T SV benchmark V0.019-20241113
- sawfish 0.12.9 discover output: assembled SV haplotype consensus contigs per candidate breakpoint

## Outputs

- Per-contig alignment classification (expected vs. unexpected) based on locus and breakend-orientation agreement
- contig_t2t_alignment_fraction.txt containing the fraction of contigs whose alignments meet the breakend-consistency expectation (target ~99.4% for HG002 32x)

## How to apply

Run sawfish 0.12.9 discover on the pbmm2 1.13.1-mapped HG002 Revio HiFi BAM (GRCh38 reference, --preset CCS) to obtain assembled SV haplotype consensus contigs per candidate breakpoint. Align each contig to the T2T-HG002-Q100v1.1 diploid assembly underlying the GIAB draft T2T SV benchmark (V0.019-20241113) using pbmm2 align with a long-read preset and 16 threads. Classify an alignment as expected only when the contig maps back to both the locus and the breakend orientation implied by its candidate breakpoint, i.e. both breakends anchor as called. Compute the fraction of contigs meeting this expectation; for HG002 32-fold HiFi data the paper reports 99.4%, so results should fall within a small tolerance of that value.

## Related tools

- **sawfish** (discovers candidate breakpoints and assembles SV haplotype consensus contigs from mapped HiFi reads via the discover step) — https://github.com/PacificBiosciences/sawfish
- **pbmm2** (aligns the assembled SV haplotype contigs to the T2T-HG002-Q100v1.1 diploid assembly (long-read preset, 16 threads) for breakend-consistency classification)

## Examples

```
pbmm2 align -j 16 --preset CCS hg002_sv_haplotypes.fa T2T-HG002-Q100v1.1.fasta hg002_contigs_to_t2t.bam
```

## Evaluation signals

- The computed contig alignment fraction reproduces the paper-reported 99.4% for HG002 32x data within a small tolerance
- Each contig classified as expected maps to both the locus and the breakend orientation implied by its candidate breakpoint (both breakends anchored as called)
- Alignment was performed with pbmm2 align long-read preset on 16 threads against T2T-HG002-Q100v1.1, matching the documented workflow
- Result is consistent with the high precision observed in variant-based accuracy assessments against the GIAB draft T2T SV benchmark
- Output file contig_t2t_alignment_fraction.txt exists and contains a fraction in [0,1] close to 0.994

## Limitations

- The assessment relies on the GIAB draft T2T SV benchmark V0.019-20241113 status of the T2T-HG002-Q100v1.1 assembly; results may shift as the benchmark matures
- T2T confident regions in the associated workflow are restricted to autosomes (grep -v chr[XY]), so sex chromosomes are not fully covered by this validation strategy
- The expectation criterion depends on correct candidate breakpoint calling by sawfish; a mis-called breakend would be scored as unexpected even if the contig sequence is accurate

## Evidence

- [methods] candidate breakpoints are assembled into SV haplotype contigs, which are then aligned back to the genome: "candidate breakpoints are assembled into SV haplotype contigs, which are then aligned back to the genome"
- [results] 99.4% of sawfish's assembled SV haplotype sequences align directly to the HG002 Q100 v1.1 diploid assembly as expected: [evidence span withheld — anchor status: not_found]
- [methods] These reads are mapped using pbmm2 1.13.1, see the above section on conda environment creation for a recommended installation scheme.: "These reads are mapped using pbmm2 1.13.1, see the above section on conda environment creation for a recommended installation scheme."
- [readme] Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads.: "Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads."
