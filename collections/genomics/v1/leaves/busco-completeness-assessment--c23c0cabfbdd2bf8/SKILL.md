---
name: busco-completeness-assessment
description: Use when scoring de novo diploid assemblies built from PacBio CLR, Nanopore
  R9 or Nanopore R10 reads, where completeness (BUSCO) must be reported next to NG50,
  phase block NG50 and Hamming error to compare pipelines.
when_to_use_negative:
- The input reads are Nanopore duplex or PacBio HiFi, where PECAT achieves only mediocre
  performance and HiFi-specific assemblers like Hifiasm are more appropriate
- The genome region has very low heterozygosity, where PECAT SNP calling and haplotype-aware
  error correction are unreliable, making phased-assembly completeness comparisons
  moot
- The task is base-level accuracy rather than gene-space completeness — use the reference-based
  Pomoxis tool instead
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BUSCO
  role: Computes gene-space completeness of each haplotype-specific assembly
- name: Pomoxis
  role: Reference-based evaluation tool for base-level accuracy of the assemblies
  repo: https://github.com/nanoporetech/pomoxis
- name: hapdiff
  role: Calls structural variants between haplotype pairs, compared against a curated
    SV set
  repo: https://github.com/KolmogorovLab/hapdiff
- name: PECAT
  role: Produces the phased diploid assemblies being assessed for completeness and
    contiguity
  repo: https://github.com/lemene/PECAT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:c8fc73e63550cd4dec248034119fb55ada6103993c250426d4f597d509ef87d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/pomoxis
  tools:
  - BUSCO
  - Clair3
  - FALCON-Unzip
  - Medaka
  - NECAT
  - PECAT
  - PacBio CLR
  - Pomoxis
  - hapdiff
  - racon
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# busco-completeness-assessment

## Summary

Evaluate haplotype-specific contigs produced by diploid long-read assemblers (PECAT, Canu+purge_dups, FALCON-Unzip, Flye+HapDup, Shasta) for gene-space completeness with BUSCO, alongside Merqury phase block NG50, Hamming error and Pomoxis reference-based accuracy, to reproduce the paper's Tables 1-2 comparisons.

## When to use

Use when scoring de novo diploid assemblies built from PacBio CLR, Nanopore R9 or Nanopore R10 reads, where completeness (BUSCO) must be reported next to NG50, phase block NG50 and Hamming error to compare pipelines.

## When NOT to use

- The input reads are Nanopore duplex or PacBio HiFi, where PECAT achieves only mediocre performance and HiFi-specific assemblers like Hifiasm are more appropriate
- The genome region has very low heterozygosity, where PECAT SNP calling and haplotype-aware error correction are unreliable, making phased-assembly completeness comparisons moot
- The task is base-level accuracy rather than gene-space completeness — use the reference-based Pomoxis tool instead

## Inputs

- Haplotype-specific assembly FASTA files for each dataset/assembler (PECAT, Canu+purge_dups, FALCON-Unzip, Flye+HapDup, Shasta, NECAT)
- Reference genomes for BUSCO lineage datasets and Pomoxis reference-based evaluation
- Deposited assemblies from Zenodo DOI 10.5281/zenodo.8380113

## Outputs

- Per-assembly BUSCO completeness metrics (complete, duplicated, single-copy, fragmented, missing gene counts)
- Table of NG50, Merqury phase block NG50, Hamming error, BUSCO and Pomoxis accuracy values per dataset/assembler (mirroring Tables 1-2)

## How to apply

Run BUSCO on each haplotype-specific contig set from the seven diploid datasets (S. cerevisiae SK1×Y12, A. thaliana Col-0×Cvi-0, Col-0×C24, D. melanogaster ISO1×A4, B. taurus Angus×Brahman, Bison×Simmental, HG002) obtained from Zenodo DOI 10.5281/zenodo.8380113. Pair completeness with Merqury-derived phase block NG50 and Hamming error, use hapdiff for structural-variant checks, and the Pomoxis reference-based evaluation tool for base-level accuracy. Tabulate every metric per dataset/assembler and compare against published headline values (e.g., phase block NG50 79.6/86.1 Mb for B. taurus Bison×Simmental and 59.4/58.0 Mb for HG002 Nanopore R10), verifying reproduction within the tolerance of the fixed parameter settings (e.g., '-f 0.005' or '-f 0.002' for large-genome PECAT configs).

## Related tools

- **BUSCO** (Computes gene-space completeness of each haplotype-specific assembly)
- **Pomoxis** (Reference-based evaluation tool for base-level accuracy of the assemblies) — https://github.com/nanoporetech/pomoxis
- **hapdiff** (Calls structural variants between haplotype pairs, compared against a curated SV set) — https://github.com/KolmogorovLab/hapdiff
- **PECAT** (Produces the phased diploid assemblies being assessed for completeness and contiguity) — https://github.com/lemene/PECAT

## Evaluation signals

- Reproduced BUSCO completeness values match the published Tables 1-2 values for each dataset/assembler within tolerance
- Reproduced Merqury phase block NG50 equals 59.4/58.0 Mb for HG002 Nanopore R10 and 79.6/86.1 Mb for B. taurus Bison×Simmental under the fixed parameter settings
- hapdiff SV calls are consistent with the curated set of structural variants
- Hamming error from Merqury stays within the reported ranges for each sequencing chemistry (PacBio CLR, Nanopore R9, Nanopore R10)

## Limitations

- PECAT needs higher data coverage to obtain a high-quality phased assembly, which affects downstream completeness comparisons
- SNP calling accuracy is low in low heterozygosity rate regions, degrading phasing and hence haplotype-specific completeness metrics
- PECAT's error correction may not distinguish small errors from heterozygote alleles in very low heterozygosity regions, biasing accuracy metrics

## Evidence

- [discussion] We use the reference-based evaluation tool Pomoxis (https://github.com/nanoporetech/pomoxis): "We use the reference-based evaluation tool Pomoxis (https://github.com/nanoporetech/pomoxis)"
- [abstract] PECAT generates more contiguous haplotype-specific contigs compared to other assemblers.: "PECAT generates more contiguous haplotype-specific contigs compared to other assemblers."
- [discussion] using hapdiff (https://github.com/KolmogorovLab/hapdiff) and compare them against the curated set of SVs: "using hapdiff (https://github.com/KolmogorovLab/hapdiff) and compare them against the curated set of SVs"
- [other] Compute completeness and base-level accuracy with BUSCO and the reference-based Pomoxis evaluation tool.: [evidence span withheld — anchor status: not_found]
