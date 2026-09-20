---
name: tandem-repeat-motif-genotyping
description: Use when haplotype-resolved long-read (PacBio or ONT) BAM alignments
  are available and genome-wide, de novo TR motifs, multiplicities, and genotypes
  must be called without prior motif or locus annotation.
when_to_use_negative:
- The target repeats lie in unassembled regions such as human centromeres or telomeres,
  which TRiCoLOR cannot profile.
- The organism is polyploid or the target chromosome is haploid (e.g., the human Y
  chromosome), which TRiCoLOR does not support.
- Only short-read data are available and no haplotype-resolved long-read alignments
  exist, since TRiCoLOR requires haplotype-resolved long-read alignments as input.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0102
tools:
- name: TRiCoLOR
  role: Overall TR profiling pipeline providing SENSoR (entropy-based repeat identification),
    REFER (haplotype-resolved repeat detection and genotyping), SAGE (trio genotyping),
    and --mendel consistency tracing.
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: SPOA
  role: Partial order alignment used by TRiCoLOR to compute haplotype-specific low-error
    consensus sequences (~77%/~88% error reduction on ONT/PB).
- name: minimap2
  role: Aligns SAGE-generated parental consensus sequences to GRCh38 for reference-like
    vs child-TR-like genotyping.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:21b4b6bab0585c36accc0cfef46af78383b93fdc293121b95eb44457cc0eeece
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
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tandem-repeat-motif-genotyping

## Summary

De novo identification and haplotype-resolved genotyping of tandem repeats (TRs) from error-prone long-read alignments using TRiCoLOR's SENSoR, REFER, and SAGE modules, with optional Mendelian-consistency tracing in trios.

## When to use

Use when haplotype-resolved long-read (PacBio or ONT) BAM alignments are available and genome-wide, de novo TR motifs, multiplicities, and genotypes must be called without prior motif or locus annotation.

## When NOT to use

- The target repeats lie in unassembled regions such as human centromeres or telomeres, which TRiCoLOR cannot profile.
- The organism is polyploid or the target chromosome is haploid (e.g., the human Y chromosome), which TRiCoLOR does not support.
- Only short-read data are available and no haplotype-resolved long-read alignments exist, since TRiCoLOR requires haplotype-resolved long-read alignments as input.

## Inputs

- Haplotype-resolved long-read BAM alignments (e.g., HGSVC PacBio blasr-reheader BAMs for HG00731/HG00732/HG00733; ONT FASTQ aligned to GRCh38 with minimap2)
- GRCh38 reference genome
- Child TR region BED file produced by TRiCoLOR SENSoR/REFER (for SAGE genotyping)
- Trio pedigree structure (child + both parents)

## Outputs

- BCF-compliant catalog of TR loci with haplotype-specific motif, multiplicity, and genotype calls
- TRiCoLOR SAGE multi-sample BCF with parental genotype calls per child TR
- Per-TR genotyping/Mendelian-consistency table (TSV) covering the index child and both parents
- Fraction of Mendelian-consistent TRs (~80% expected for the HG00733 PB trio under the >= 10 bp reference-difference filter)

## How to apply

Run TRiCoLOR SENSoR to flag low-entropy repetitive segments using the Shannon entropy of DNA in non-overlapping 20 bp sliding windows (entropy <= 1.23), requiring >= 5 reads and merging hits within 100 bp; filter candidates for mean coverage (ONT >8x, PB >10x). Use TRiCoLOR REFER to profile each TR by partial order alignment (POA, via SPOA) to build haplotype-specific low-error consensus sequences (~77% ONT and ~88% PB error reduction), resolve motif (length <= 6, >= 5 perfect repeats, edit distance <= 1 approximations) and multiplicity with a RegEx-based string matcher plus an N-gram model for competing predictions, and store TRs >= 50 bp in BCF format. For trio genotyping, use TRiCoLOR SAGE to collect spanning reads per haplotype per parent over child TR regions, trim and compute SPOA consensus, align parental consensus to GRCh38 with minimap2, and classify each parental allele as reference-like or child-TR-like by edit distance. Enable the --mendel parameter to flag each TR as Mendelian consistent or inconsistent, then restrict consistency assessment to variants differing from the reference by >= 10 bp (including multi-allelic variants differing by the same distance).

## Related tools

- **TRiCoLOR** (Overall TR profiling pipeline providing SENSoR (entropy-based repeat identification), REFER (haplotype-resolved repeat detection and genotyping), SAGE (trio genotyping), and --mendel consistency tracing.) — https://github.com/davidebolo1993/TRiCoLOR
- **SPOA** (Partial order alignment used by TRiCoLOR to compute haplotype-specific low-error consensus sequences (~77%/~88% error reduction on ONT/PB).)
- **minimap2** (Aligns SAGE-generated parental consensus sequences to GRCh38 for reference-like vs child-TR-like genotyping.)

## Evaluation signals

- The Mendelian-consistent fraction for the HG00731/HG00732/HG00733 PB trio approximates the ~80% reported in the paper after filtering for variants differing from the reference by >= 10 bp.
- The per-TR output table contains genotype calls for the index child and both parents (trio-complete genotypes).
- TR calls respect the default thresholds: entropy <= 1.23 in 20 bp windows, >= 5 reads, motifs <= 6 bp with >= 5 perfect repeats, edit distance <= 1, TRs >= 50 bp.
- Coverage filters applied correctly: ONT low-entropy regions retain mean coverage >8 and PB regions >10x.
- SAGE consensus construction reproduces the ~88% error reduction expected on PacBio data.

## Limitations

- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres).
- No support for polyploid species or haploid chromosomes (e.g., the human Y chromosome).
- Applied extensively only to micro-satellites; profiling of mini-satellites is not validated.
- Mendelian-consistency assessment should be restricted to variants differing from the reference by >= 10 bp, including multi-allelic variants differing by the same distance.

## Evidence

- [abstract] The tool includes methods to interactively visualize the identified repeats and to trace their Mendelian consistency in pedigrees.: "The tool includes methods to interactively visualize the identified repeats and to trace their Mendelian consistency in pedigrees."
- [methods] TRiCoLOR requires haplotype-resolved long-read alignments as input.: "TRiCoLOR (Tandem Repeats Caller for Long Reads) requires haplotype-resolved long-read alignments as input"
- [methods] to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [results] Filtering for variants differing from the reference for >= 10 bp and for multi-allelic variants differing by the same distance.: "Filtering for variants differing from the reference for ≥10 bp and for multi-allelic variants differing from each other by the same distance"
- [readme] TRiCoLOR is available to be installed from source or as a Docker container; see the TRiCoLOR documentation for any installation or usage questions.: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
