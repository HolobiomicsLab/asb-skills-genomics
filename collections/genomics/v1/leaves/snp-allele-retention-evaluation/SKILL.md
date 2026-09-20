---
name: snp-allele-retention-evaluation
description: Use when evaluating whether an error-correction pipeline for long noisy
  reads (PacBio CLR or Nanopore) retains heterozygote alleles, or when comparing correct-then-assemble
  tools on simulated diploid data with known SNP truth labels.
when_to_use_negative:
- Input reads are PacBio HiFi or Nanopore duplex — the paper reports PECAT achieves
  only mediocre performance on Nanopore duplex and PacBio HiFi reads, so this CLR/ONT
  noisy-read benchmark does not transfer
- Truth haplotypes are unknown (real diploid data without simulated SNP labels) —
  retained-allele accuracy cannot be computed without sequence truth
- The goal is assembly-level phasing quality (phase block NG50, hapdiff SV comparison)
  rather than error-correction SNP retention
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0102
tools:
- name: PECAT
  role: Haplotype-aware error correction of raw reads whose SNP allele retention is
    being measured; run via 'pecat.pl correct cfgfile' with minimap2 for read-to-read
    overlaps
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Read-to-read overlap detection for PECAT correction ('-x ava-pb' for PacBio
    CLR) and mapping corrected reads for SNP retention assessment
  repo: https://github.com/lh3/minimap2
- name: Canu
  role: Baseline error corrector run with default pipeline on the same simulated reads
    for comparison
- name: MECAT
  role: Baseline error corrector run with default pipeline on the same simulated reads
    for comparison
- name: NECAT
  role: Baseline error corrector run with default pipeline on the same simulated reads
    for comparison
- name: pomoxis
  role: Reference-based evaluation tool from the paper's validation toolkit, usable
    for assessing corrected-read accuracy against truth
  repo: https://github.com/nanoporetech/pomoxis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:3cdb4d7844c33f177e74b433180965564ea763610fed40025a67a2a046b1deb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Canu
  - MECAT
  - NECAT
  - PECAT
  - minimap2
  - pomoxis
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# snp-allele-retention-evaluation

## Summary

A benchmarking skill that quantifies how well haplotype-aware error correction (PECAT) versus baseline correctors (Canu, MECAT, NECAT) preserve heterozygous SNP alleles in corrected long reads, using simulated PacBio CLR and Nanopore diploid datasets across a gradient of heterozygosity rates.

## When to use

Use when evaluating whether an error-correction pipeline for long noisy reads (PacBio CLR or Nanopore) retains heterozygote alleles, or when comparing correct-then-assemble tools on simulated diploid data with known SNP truth labels.

## When NOT to use

- Input reads are PacBio HiFi or Nanopore duplex — the paper reports PECAT achieves only mediocre performance on Nanopore duplex and PacBio HiFi reads, so this CLR/ONT noisy-read benchmark does not transfer
- Truth haplotypes are unknown (real diploid data without simulated SNP labels) — retained-allele accuracy cannot be computed without sequence truth
- The goal is assembly-level phasing quality (phase block NG50, hapdiff SV comparison) rather than error-correction SNP retention

## Inputs

- Simulated diploid long-read FASTQ/FASTA sets with PacBio CLR error profile
- Simulated diploid long-read FASTQ/FASTA sets with Nanopore (R9/R10-style) error profile
- Simulated heterozygosity rates 0.0001 and >=0.0005 with sequence truth labels
- Reference genome used for read simulation

## Outputs

- snp_allele_accuracy_fig2a.csv: per-tool, per-heterozygosity-rate SNP allele accuracies
- Corrected reads per tool (e.g. $PROJECT/1-correct/corrected_reads.fasta from PECAT)
- Per-condition alignment of corrected reads against the simulated diploid truth

## How to apply

Simulate diploid long-read datasets with PacBio CLR and Nanopore error profiles at heterozygosity rates of 0.0001 and >=0.0005, retaining the simulated truth so heterozygous SNP alleles can be scored. Run PECAT's haplotype-aware correction by generating a config with 'pecat.pl config cfgfile', setting genome_size and coverage, then 'pecat.pl correct cfgfile', which uses minimap2 read-to-read overlaps (e.g. '-x ava-pb' for PacBio CLR) and writes corrected reads to $PROJECT/1-correct/corrected_reads.fasta. Run Canu, MECAT, and NECAT with their default pipelines on the identical simulated read sets. For each tool and heterozygosity condition, align corrected reads against the simulated diploid truth and compute the fraction of heterozygous SNP alleles retained after correction. Tabulate per-tool accuracies into a CSV mirroring the Fig. 2a layout of the PECAT paper (doi:10.1038/s41467-024-47349-7).

## Related tools

- **PECAT** (Haplotype-aware error correction of raw reads whose SNP allele retention is being measured; run via 'pecat.pl correct cfgfile' with minimap2 for read-to-read overlaps) — https://github.com/lemene/PECAT
- **minimap2** (Read-to-read overlap detection for PECAT correction ('-x ava-pb' for PacBio CLR) and mapping corrected reads for SNP retention assessment) — https://github.com/lh3/minimap2
- **Canu** (Baseline error corrector run with default pipeline on the same simulated reads for comparison)
- **MECAT** (Baseline error corrector run with default pipeline on the same simulated reads for comparison)
- **NECAT** (Baseline error corrector run with default pipeline on the same simulated reads for comparison)
- **pomoxis** (Reference-based evaluation tool from the paper's validation toolkit, usable for assessing corrected-read accuracy against truth) — https://github.com/nanoporetech/pomoxis

## Examples

```
pecat.pl config cfgfile && pecat.pl correct cfgfile  # outputs $PROJECT/1-correct/corrected_reads.fasta
```

## Evaluation signals

- PECAT corrected reads show SNP allele accuracy >99% at heterozygosity rates >=0.0005
- Canu falls below 80% and MECAT and NECAT fall below 60% SNP allele accuracy at the same >=0.0005 conditions
- PECAT output corrected_reads.fasta exists at $PROJECT/1-correct/corrected_reads.fasta for each condition
- The tabulated CSV reproduces a Fig. 2a-style layout with one accuracy value per tool per heterozygosity rate
- Both sequencing error profiles (PacBio CLR and Nanopore) are represented for every tool/condition cell

## Limitations

- PECAT's error correction may not distinguish small sequencing errors from heterozygote alleles in very low heterozygosity regions, and its SNP call accuracy is low for low heterozygosity rate regions
- Even after error correction, SNPs cannot be accurately called by just aligning corrected reads (unlike Hifiasm with HiFi reads); PECAT supplements with raw-read SNP calling via Clair3 for Nanopore
- PECAT achieves only mediocre performance on Nanopore duplex and PacBio HiFi reads, and needs higher data coverage for high-quality phased assembly

## Evidence

- [results] Haplotype-aware error correction retains SNP alleles with greater than 99% accuracy at heterozygosity rates >= 0.0005: [evidence span withheld — anchor status: not_found]
- [discussion] we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads: "we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads"
- [discussion] The PECAT error correction method is based on the partial-order alignment (POA) graph method: "The PECAT error correction method is based on the partial-order alignment (POA) graph method4,47."
- [discussion] our current error correction method may not be effective enough to distinguish small errors from heterozygote alleles in very low heterozygosity rate regions: "our current error correction method may not be effective enough to distinguish small errors from heterozygote alleles in very low heterozygosity rate regions"
- [readme] README: PECAT is a phased error correction and assembly tool with minimap2 dependency: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
