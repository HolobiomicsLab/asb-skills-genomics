---
name: haplotype-phase-block-accuracy-assessment
description: Use when comparing diploid assembly pipelines on long noisy reads (PacBio
  CLR, Nanopore R9/R10) and you need phase block NG50, Hamming error, NG50 and BUSCO
  metrics tabulated per assembler/dataset.
when_to_use_negative:
- Input assemblies are from highly accurate reads (PacBio HiFi, Nanopore duplex) where
  PECAT-class noisy-read benchmarks and the phase-aware correction rationale do not
  apply — the paper reports only mediocre performance there
- The goal is de novo assembly itself rather than post-assembly accuracy/contiguity
  benchmarking of existing FASTA outputs
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Pomoxis
  role: Reference-based evaluation of base-level assembly accuracy (identity/QV underlying
    Hamming error estimates)
  repo: https://github.com/nanoporetech/pomoxis
- name: hapdiff
  role: SV calling between the two haplotypes of a diploid assembly and comparison
    against a curated SV set
  repo: https://github.com/KolmogorovLab/hapdiff
- name: minimap2
  role: Aligns assemblies/reads for metric computation; hapdiff bundles its own minimap2
    to keep results version-independent
  repo: https://github.com/lh3/minimap2
- name: Medaka
  role: Polishing step whose effect on final assembly accuracy is measured by the
    evaluation metrics (used for Nanopore assemblies)
  repo: https://github.com/nanoporetech/medaka
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:858d8dc512501b5703806bdea5d6ad314567b72c8d5b9a2bd45cc18d2d46a47f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/pomoxis
  tools:
  - Clair3
  - FALCON-Unzip
  - Medaka
  - NECAT
  - PECAT
  - PacBio CLR
  - Pomoxis
  - hapdiff
  - minimap2
  - racon
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# haplotype-phase-block-accuracy-assessment

## Summary

Benchmarking protocol for diploid long-read assemblies that scores contiguity (NG50, Merqury phase block NG50), completeness (BUSCO), base-level accuracy (Pomoxis Hamming error), and SV consistency (hapdiff) per assembler and dataset, as used to evaluate PECAT against Canu+purge_dups, FALCON-Unzip, Flye+HapDup and Shasta.

## When to use

Use when comparing diploid assembly pipelines on long noisy reads (PacBio CLR, Nanopore R9/R10) and you need phase block NG50, Hamming error, NG50 and BUSCO metrics tabulated per assembler/dataset.

## When NOT to use

- Input assemblies are from highly accurate reads (PacBio HiFi, Nanopore duplex) where PECAT-class noisy-read benchmarks and the phase-aware correction rationale do not apply — the paper reports only mediocre performance there
- The goal is de novo assembly itself rather than post-assembly accuracy/contiguity benchmarking of existing FASTA outputs

## Inputs

- Haplotype-resolved assembly FASTA files (two haplotypes) per assembler per dataset
- Reference genomes for the seven diploid datasets (yeast, Arabidopsis, fly, cattle, HG002)
- Assemblies deposited at Zenodo DOI 10.5281/zenodo.8380113
- Curated structural-variant call set for SV-concordance checks
- Raw long reads (PacBio CLR, Nanopore R9, Nanopore R10) for Merqury k-mer-based metrics

## Outputs

- Per-dataset/per-assembler table of NG50 (QUAST-style) and Merqury phase block NG50 in Mb
- Hamming error (base-level substitution rate) per haplotype from Pomoxis
- BUSCO completeness scores per assembly
- hapdiff SV calls (VCF) for the haplotype pair, compared against the curated SV set
- Comparison against published Tables 1-2 headline values (e.g., phase block NG50 79.6/86.1 Mb for B. taurus Bison×Simmental)

## How to apply

For each of the seven diploid datasets (S. cerevisiae SK1×Y12, A. thaliana Col-0×Cvi-0, A. thaliana Col-0×C24, D. melanogaster ISO1×A4, B. taurus Angus×Brahman, B. taurus Bison×Simmental, HG002) and each assembler output, compute NG50 QUAST-style against the reference size, and Merqury phase block NG50 and Hamming error from the two haplotype-specific contig sets. Use the reference-based Pomoxis tool for base-level accuracy, and BUSCO for completeness. Run hapdiff on the haplotype pair against a curated SV set to check structural-variant concordance; note hapdiff bundles its own minimap2 because results can depend on aligner version and parameters. Validate the whole benchmark by confirming headline reproductions such as phase block NG50 59.4/58.0 Mb for HG002 Nanopore R10 and 79.6/86.1 Mb for B. taurus Bison×Simmental match the published Tables 1-2 values for fixed parameters.

## Related tools

- **Pomoxis** (Reference-based evaluation of base-level assembly accuracy (identity/QV underlying Hamming error estimates)) — https://github.com/nanoporetech/pomoxis
- **hapdiff** (SV calling between the two haplotypes of a diploid assembly and comparison against a curated SV set) — https://github.com/KolmogorovLab/hapdiff
- **minimap2** (Aligns assemblies/reads for metric computation; hapdiff bundles its own minimap2 to keep results version-independent) — https://github.com/lh3/minimap2
- **Medaka** (Polishing step whose effect on final assembly accuracy is measured by the evaluation metrics (used for Nanopore assemblies)) — https://github.com/nanoporetech/medaka

## Evaluation signals

- Reproduced phase block NG50 values match published Tables 1-2 (e.g., 59.4/58.0 Mb for HG002 Nanopore R10) within the tolerance of the fixed parameter settings
- NG50 values are computed QUAST-style against reference genome size, so smaller genomes (e.g., S. cerevisiae NG50 0.8/0.8 Mb) and large genomes (B. taurus 24.5/11.9 Mb on PacBio CLR) fall in the published ranges
- Hamming error from Pomoxis agrees with the paper's reported base-level accuracy per dataset/sequencing chemistry
- hapdiff VCF calls, generated with its bundled minimap2, are concordant with the curated SV set (no excess structural-variant discrepancies between haplotypes)
- BUSCO completeness is consistent across assemblers for the same dataset, confirming differences are phasing/contiguity-driven rather than completeness-driven

## Limitations

- hapdiff results can depend on the aligner version and parameters, which is why it ships its own bundled minimap2 — mixing minimap2 versions across benchmarked assemblers can change SV concordance
- Reproducibility gap: the exact assembly parameters being benchmarked are described only in Supplementary Notes 4 and 5, not the main text
- Reference-free phasing metrics (Merqury) require k-mer contexts and are only as good as the read set; ultra-low heterozygosity regions degrade SNP-call-based phase block estimates

## Evidence

- [abstract] phase block NG50 with 59.4/58.0 Mb for HG002 using Nanopore R10 reads: "phase block NG50 with 59.4/58.0 Mb for HG002 using Nanopore R10 reads"
- [discussion] Pomoxis as reference-based evaluation tool: "We use the reference-based evaluation tool Pomoxis (https://github.com/nanoporetech/pomoxis)"
- [discussion] hapdiff SV checks against curated set: "using hapdiff (https://github.com/KolmogorovLab/hapdiff) and compare them against the curated set of SVs"
- [readme] hapdiff bundles minimap2 for reproducibility: [evidence span withheld — anchor status: external_doc]
