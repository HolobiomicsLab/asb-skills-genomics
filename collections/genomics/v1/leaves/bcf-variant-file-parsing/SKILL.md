---
name: bcf-variant-file-parsing
description: Use when a TRiCoLOR SAGE run has emitted a multi-sample BCF for a trio
  and the per-TR genotypes (child plus both parents) must be extracted into a tabular
  form to compute Mendelian-consistent fractions.
when_to_use_negative:
- Input is a raw multi-sample VCF/BCF from a different caller (e.g. GATK) rather than
  the TRiCoLOR SAGE output — the genotype semantics (reference-like vs child-TR-like
  per haplotype) are TRiCoLOR-specific
- The BCF contains only single-sample records, so the father/mother/child genotypes
  needed for Mendelian consistency cannot be jointly parsed
- TRs shorter than the ≥50 bp default were retained in the BCF by non-default settings,
  making the result incomparable to the paper's ~80% figure
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: TRiCoLOR
  role: Produces the BCF-compliant output of profiled TRs that this skill parses
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR SAGE
  role: Genotyping module whose multi-sample BCF output (with --mendel flags) is the
    parse target for trio Mendelian-consistency analysis
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:654a565098ba3e678b50109253fd04e9fec32158fb48806836250946b2a00aa2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - SPOA
  - TRiCoLOR
  - TRiCoLOR SAGE
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# BCF variant file parsing

## Summary

Parsing the BCF-compliant multi-sample variant output produced by TRiCoLOR SAGE to extract per-tandem-repeat genotype calls for trio members, enabling downstream Mendelian-consistency computation on the Puerto Rican PB trio (HG00731/HG00732/HG00733).

## When to use

Use when a TRiCoLOR SAGE run has emitted a multi-sample BCF for a trio and the per-TR genotypes (child plus both parents) must be extracted into a tabular form to compute Mendelian-consistent fractions.

## When NOT to use

- Input is a raw multi-sample VCF/BCF from a different caller (e.g. GATK) rather than the TRiCoLOR SAGE output — the genotype semantics (reference-like vs child-TR-like per haplotype) are TRiCoLOR-specific
- The BCF contains only single-sample records, so the father/mother/child genotypes needed for Mendelian consistency cannot be jointly parsed
- TRs shorter than the ≥50 bp default were retained in the BCF by non-default settings, making the result incomparable to the paper's ~80% figure

## Inputs

- Multi-sample BCF file produced by TRiCoLOR SAGE (one record per TR, samples = father HG00731, mother HG00732, child HG00733)
- Sample-to-pedigree-role mapping (father/mother/child) for the trio

## Outputs

- Per-TR TSV with TR genomic coordinates and genotype calls for the index child and both parents
- Overall Mendelian-consistency fraction (expected ~80% for the HG00733 PB trio with ≥10 bp reference-difference filtering)

## How to apply

TRiCoLOR stores profiled TRs — those ≥50 bp by default, varying between haplotypes or the reference — in BCF-compliant format, so the SAGE module's output is a multi-sample BCF containing one record per child TR region with genotypes for the index child and each parent. Read the BCF with a standard variant parser (e.g. pysam/bcftools), keeping the sample columns for HG00731 (father), HG00732 (mother), and HG00733 (child). For each TR record, flatten the per-sample genotype/allele fields into one row of a TSV with TR coordinates and the three genotype calls. Restrict the Mendelian-consistency assessment to variants differing from the GRCh38 reference by ≥10 bp, including multi-allelic variants differing by the same distance, as applied in the paper's PB trio analysis. Then compute the overall fraction of Mendelian-consistent TRs, which should approximate the ~80% consistency rate reported for the HG00733 Pacific Biosciences trio when --mendel is enabled.

## Related tools

- **TRiCoLOR** (Produces the BCF-compliant output of profiled TRs that this skill parses) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR SAGE** (Genotyping module whose multi-sample BCF output (with --mendel flags) is the parse target for trio Mendelian-consistency analysis) — https://github.com/davidebolo1993/TRiCoLOR

## Evaluation signals

- The per-TR TSV contains genotype calls for all three trio members (HG00731, HG00732, HG00733) for every TR record in the BCF
- Record count of the parsed TSV matches the record count of the SAGE BCF (no TRs silently dropped)
- The computed Mendelian-consistent fraction approximates the ~80% reported for the PB trio when restricted to variants differing from the reference by ≥10 bp
- Every genotype field resolves to reference-like or child-TR-like (no unparsed allele encodings) before consistency assessment

## Limitations

- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres), so the BCF will lack such loci
- The tool does not support polyploid species or haploid chromosomes (e.g., the human Y chromosome), constraining which BCF records are interpretable
- Mendelian consistency figures depend on the ≥10 bp reference-difference and same-distance multi-allelic filters; parsing the BCF without applying them yields a different (higher) consistency fraction than the paper's

## Evidence

- [methods] TRs stored in BCF format: "TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format"
- [abstract] Mendelian consistency tracing in pedigrees: "The tool includes methods to interactively visualize the identified repeats and to trace their Mendelian consistency in pedigrees."
- [results] ≥10 bp filter for Mendelian analysis: "Filtering for variants differing from the reference for ≥10 bp and for multi-allelic variants differing from each other by the same distance"
- [results] PB trio datasets: "the PB sequencing data for HG00731 (Puerto Rican, father)"
- [readme] Readme citation and project home: "TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing data"
