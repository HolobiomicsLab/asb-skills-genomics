---
name: long-read-contig-to-assembly-mapping
description: Use when you need to independently verify that assembled SV haplotype
  consensus contigs are real by aligning them to a finished diploid assembly and checking
  locus/orientation concordance.
when_to_use_negative:
- Input contigs were not assembled from mapped HiFi reads by sawfish (e.g. de novo
  whole-genome assemblies), since the expectation check depends on sawfish candidate
  breakpoint loci and breakend orientations
- Only a GRCh38 reference is available and no independent diploid assembly exists
  — mapping contigs back to the same reference used for discovery is circular and
  would not provide orthogonal validation
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sawfish
  role: Assembles candidate breakpoints into SV haplotype consensus contigs via POA
    and realignment
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2
  role: Aligns the SV haplotype contigs to the T2T-HG002-Q100v1.1 assembly (long-read
    preset, 16 threads)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:051b743412d624d130c9ea277e6c791c0510b88321afece3ef565b2e5176b893
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - pbmm2
  - sawfish
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# long-read-contig-to-assembly-mapping

## Summary

Validates locally assembled SV haplotype contigs from sawfish by aligning them with pbmm2 to an independent diploid assembly (T2T-HG002-Q100v1.1) and computing the fraction that map back to the expected locus and breakend orientation, serving as an orthogonal precision check on SV calls.

## When to use

Use when you need to independently verify that assembled SV haplotype consensus contigs are real by aligning them to a finished diploid assembly and checking locus/orientation concordance.

## When NOT to use

- Input contigs were not assembled from mapped HiFi reads by sawfish (e.g. de novo whole-genome assemblies), since the expectation check depends on sawfish candidate breakpoint loci and breakend orientations
- Only a GRCh38 reference is available and no independent diploid assembly exists — mapping contigs back to the same reference used for discovery is circular and would not provide orthogonal validation

## Inputs

- HG002 Revio HiFi BAM mapped to GRCh38 (pbmm2 1.13.1, --preset CCS), e.g. m84011_220902_175841_s1.hifi_reads.bam
- Assembled SV haplotype consensus contigs from sawfish 0.12.9 discover, one per candidate breakpoint
- T2T-HG002-Q100v1.1 diploid assembly FASTA underlying the GIAB draft T2T SV benchmark (V0.019-20241113)
- Candidate breakpoint coordinates and breakend orientations from the sawfish discover step

## Outputs

- Per-contig alignment records of SV haplotype contigs to the T2T-HG002-Q100v1.1 assembly
- Per-contig classification as expected/not-expected based on locus and breakend orientation concordance
- contig_t2t_alignment_fraction.txt containing the fraction of contigs aligning as expected (paper value: 99.4%)

## How to apply

First run sawfish 0.12.9 discover on the pbmm2 1.13.1-mapped HG002 Revio HiFi BAM (GRCh38 reference, --preset CCS) so that each candidate breakpoint is assembled into SV haplotype consensus contigs via its POA/realignment procedure. Obtain the T2T-HG002-Q100v1.1 diploid assembly underlying the GIAB draft T2T SV benchmark (V0.019-20241113). Align the contigs to this assembly with pbmm2 align using the long-read preset and 16 threads. Classify each contig alignment as 'expected' only when the contig maps back to both the locus and the breakend orientation implied by its candidate breakpoint (both breakends anchored as called). Compute the fraction of contigs meeting this expectation; for HG002 32x HiFi data the paper reports 99.4%, so results within a small tolerance of that value validate the workflow.

## Related tools

- **sawfish** (Assembles candidate breakpoints into SV haplotype consensus contigs via POA and realignment) — https://github.com/PacificBiosciences/sawfish
- **pbmm2** (Aligns the SV haplotype contigs to the T2T-HG002-Q100v1.1 assembly (long-read preset, 16 threads))

## Examples

```
pbmm2 --log-level INFO align -j 16 --preset CCS --sort --unmapped sv_haplotype_contigs.fasta T2T-HG002-Q100v1.1.fasta contigs_to_t2t.bam
```

## Evaluation signals

- The computed contig-to-assembly alignment fraction reproduces the paper-reported 99.4% for HG002 32x data (Results 3.1 / Supplementary Results) within a small tolerance
- Each 'expected' alignment maps to the locus implied by its candidate breakpoint
- Both breakends of each contig alignment are anchored in the orientation implied by the called breakpoint
- The fraction is consistent with variant-based precision assessments (truvari-based accuracy) reported in the paper

## Limitations

- Expectation classification requires both breakends to be anchored as called; contigs from breakpoints with ambiguous or unresolved breakends may be counted as not-expected even when biologically real
- Validation is specific to HG002 32-fold HiFi data and the T2T-HG002-Q100v1.1 assembly; other samples or assemblies have no published expected fraction
- VACmap is still unsupported in the sawfish ecosystem, so problematic alignments may lack clear error messages

## Evidence

- [methods] Align the SV haplotype contigs to the T2T-HG002-Q100v1.1 assembly with pbmm2 align (long-read preset, 16 threads).: [evidence span withheld — anchor status: not_found]
- [results] 99.4% of sawfish's assembled SV haplotype sequences align directly to the HG002 Q100 v1.1 diploid assembly as expected: [evidence span withheld — anchor status: not_found]
- [methods] candidate breakpoints are assembled into SV haplotype contigs, which are then aligned back to the genome: "candidate breakpoints are assembled into SV haplotype contigs, which are then aligned back to the genome"
- [methods] the GIAB draft SV benchmark (V0.019-20241113) based on the T2T-HG002-Q100v1.1 diploid assembly: "the GIAB draft SV benchmark (V0.019-20241113) based on the T2T-HG002-Q100v1.1 diploid assembly"
