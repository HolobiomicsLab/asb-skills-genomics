---
name: alignment-success-fraction-quantification
description: Use when you need to validate an SV caller's assembled haplotype contigs
  by remapping them to an independent high-quality diploid assembly and computing
  the fraction that map back as expected at the called breakpoint.
when_to_use_negative:
- You are evaluating call-set accuracy against a variant benchmark (use truvari bench/refine
  instead of contig realignment)
- The contigs come from raw read assemblies rather than sawfish's candidate-breakpoint
  SV haplotype contigs, so no expected locus/orientation is defined
- No independent diploid assembly of the same sample is available to serve as an alignment
  target
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sawfish 0.12.9
  role: Produces the assembled SV haplotype consensus contigs per candidate breakpoint
    via the discover step
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2 1.13.1
  role: Aligns the SV haplotype contigs to the T2T-HG002-Q100v1.1 assembly (long-read
    preset, 16 threads)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:35c13cba03299edd051e0a199550d57ab5be6a2ff1bf3a392ca67cb462f660f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - pbmm2
  - pbmm2 1.13.1
  - sawfish
  - sawfish 0.12.9
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# alignment-success-fraction-quantification

## Summary

Quantifies the fraction of assembled SV haplotype consensus contigs that align directly (correct locus and breakend orientation) to an independent diploid assembly, providing an assembly-level precision check for an SV caller's local assemblies.

## When to use

Use when you need to validate an SV caller's assembled haplotype contigs by remapping them to an independent high-quality diploid assembly and computing the fraction that map back as expected at the called breakpoint.

## When NOT to use

- You are evaluating call-set accuracy against a variant benchmark (use truvari bench/refine instead of contig realignment)
- The contigs come from raw read assemblies rather than sawfish's candidate-breakpoint SV haplotype contigs, so no expected locus/orientation is defined
- No independent diploid assembly of the same sample is available to serve as an alignment target

## Inputs

- pbmm2 1.13.1-mapped HG002 Revio HiFi BAM (m84011_220902_175841_s1.hifi_reads.bam aligned to human_GRCh38_no_alt_analysis_set, --preset CCS)
- sawfish 0.12.9 discover output: assembled SV haplotype consensus contigs per candidate breakpoint
- T2T-HG002-Q100v1.1 diploid assembly FASTA (underlying the GIAB draft T2T SV benchmark V0.019-20241113)

## Outputs

- Per-contig alignment classifications (expected vs. not expected) from pbmm2 contig-to-assembly alignments
- contig_t2t_alignment_fraction.txt containing the fraction of contigs aligning as expected

## How to apply

Run sawfish 0.12.9 discover on the pbmm2 1.13.1-mapped HG002 Revio HiFi BAM (GRCh38 reference, --preset CCS) to produce assembled SV haplotype consensus contigs per candidate breakpoint. Align these contigs to the T2T-HG002-Q100v1.1 diploid assembly (the assembly underlying the GIAB draft T2T SV benchmark V0.019-20241113) using pbmm2 align with a long-read preset and 16 threads. Classify each contig as 'expected' only when its alignment lands at the locus with the breakend orientation implied by its candidate breakpoint (both breakends anchored as called). Compute the fraction of contigs meeting this expectation and write it to contig_t2t_alignment_fraction.txt. The rationale is that direct alignment of assembled contigs to a truth-quality assembly independently corroborates variant-based precision estimates.

## Related tools

- **sawfish 0.12.9** (Produces the assembled SV haplotype consensus contigs per candidate breakpoint via the discover step) — https://github.com/PacificBiosciences/sawfish
- **pbmm2 1.13.1** (Aligns the SV haplotype contigs to the T2T-HG002-Q100v1.1 assembly (long-read preset, 16 threads))

## Examples

```
pbmm2 align --preset SUBREAD -j 16 sv_haplotype_contigs.fasta T2T-HG002-Q100v1.1.fasta contigs_to_t2t.bam
```

## Evaluation signals

- The computed contig alignment fraction for HG002 32x data should reproduce the paper-reported 99.4% within a small tolerance
- A contig counts as expected only when both breakends are anchored as called at the candidate breakpoint locus and orientation
- All contigs should produce alignments in the pbmm2 output BAM; unaligned contigs count as failures
- The output file contig_t2t_alignment_fraction.txt must contain a single fraction value in [0,1]

## Limitations

- Assessment requires an independently assembled, high-quality diploid assembly (T2T-HG002-Q100v1.1) of the same sample; not applicable without such a truth assembly
- The expectation criterion (both breakends anchored as called) depends on the correctness of sawfish's candidate breakpoint annotations
- Contig alignment success reflects contig-level precision and may not capture small sequence errors within aligned contigs

## Evidence

- [methods] 99.4% of sawfish's assembled SV haplotype sequences align directly to the HG002 Q100 v1.1 diploid assembly as expected: [evidence span withheld — anchor status: not_found]
- [methods] candidate breakpoints are assembled into SV haplotype contigs, which are then aligned back to the genome: "candidate breakpoints are assembled into SV haplotype contigs, which are then aligned back to the genome"
- [methods] the GIAB draft SV benchmark (V0.019-20241113) based on the T2T-HG002-Q100v1.1 diploid assembly: "the GIAB draft SV benchmark (V0.019-20241113) based on the T2T-HG002-Q100v1.1 diploid assembly"
- [methods] These reads are mapped using pbmm2 1.13.1, see the above section on conda environment creation for a recommended installation scheme.: "These reads are mapped using pbmm2 1.13.1, see the above section on conda environment creation for a recommended installation scheme."
