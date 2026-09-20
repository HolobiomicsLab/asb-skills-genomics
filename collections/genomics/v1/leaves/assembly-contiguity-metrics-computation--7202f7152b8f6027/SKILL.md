---
name: assembly-contiguity-metrics-computation
description: Use when comparing diploid genome assemblies from long noisy reads (PacBio
  CLR, Nanopore R9/R10) and you need NG50, phase block NG50, Hamming error, BUSCO,
  or reference-based accuracy figures to reproduce published Tables 1-2.
when_to_use_negative:
- Input is PacBio HiFi or Nanopore duplex data for which the paper reports PECAT achieves
  'mediocre performance' — the comparison framework is designed for long noisy reads
- You lack a reference genome or curated SV set, since NG50, Pomoxis accuracy, and
  hapdiff SV checks are all reference-dependent
- You need trio-binned phasing output (e.g., comparison against methods requiring
  additional parental reads) rather than haplotype-resolved de novo assembly metrics
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PECAT
  role: Produces the diploid assemblies whose NG50 and phase block NG50 are computed;
    config templates (cfg_yeast_clr ... cfg_hg002_ont) run via 'pecat.pl unzip cfgfile'
  repo: https://github.com/lemene/PECAT
- name: Pomoxis
  role: Reference-based evaluation tool for base-level assembly accuracy
  repo: https://github.com/nanoporetech/pomoxis
- name: hapdiff
  role: SV calling between haplotype pairs of the assembly, compared against a curated
    SV set
  repo: https://github.com/KolmogorovLab/hapdiff
- name: Clair3
  role: SNP calling on Nanopore assemblies/reads (v0.1-r12) feeding phasing metrics
  repo: https://github.com/HKU-BAL/Clair3
- name: Medaka
  role: Consensus polishing (v1.7.2) of Nanopore assemblies prior to metric computation
  repo: https://github.com/nanoporetech/medaka
- name: minimap2
  role: Read-overlap and read-to-contig alignment used within the PECAT pipeline whose
    assemblies are evaluated
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:8dd75b07b7e370e40b9f6a2f5d6f83c56d987fe90e93d2d60e3f007b2792c37e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
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
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# assembly-contiguity-metrics-computation

## Summary

Compute contiguity and phasing quality metrics (QUAST-style NG50, Merqury phase block NG50, Hamming error) plus completeness/accuracy (BUSCO, Pomoxis) for diploid long-read assemblies, enabling direct comparison of PECAT against Canu+purge_dups, FALCON-Unzip, Flye+HapDup, and Shasta.

## When to use

Use when comparing diploid genome assemblies from long noisy reads (PacBio CLR, Nanopore R9/R10) and you need NG50, phase block NG50, Hamming error, BUSCO, or reference-based accuracy figures to reproduce published Tables 1-2.

## When NOT to use

- Input is PacBio HiFi or Nanopore duplex data for which the paper reports PECAT achieves 'mediocre performance' — the comparison framework is designed for long noisy reads
- You lack a reference genome or curated SV set, since NG50, Pomoxis accuracy, and hapdiff SV checks are all reference-dependent
- You need trio-binned phasing output (e.g., comparison against methods requiring additional parental reads) rather than haplotype-resolved de novo assembly metrics

## Inputs

- Haplotig/primary contig FASTA assemblies from PECAT and comparator pipelines (Canu+purge_dups, FALCON-Unzip, Flye+HapDup, Shasta)
- Seven diploid long-read datasets (PacBio CLR, Nanopore R9, Nanopore R10 ultra-long/duplex, PacBio HiFi) from Zenodo DOI 10.5281/zenodo.8380113
- Reference genomes for QUAST-style NG50, Pomoxis, and BUSCO runs
- Curated set of structural variants for hapdiff comparison

## Outputs

- Per-dataset, per-assembler table of NG50 and phase block NG50 values (e.g., PECAT NG50 0.8/0.8, 14.3/7.8, 24.5/11.9, 72.4/2.8 Mb on PacBio CLR)
- Merqury phase block NG50 and Hamming error per haplotype
- BUSCO completeness scores and Pomoxis reference-based accuracy (QV) per assembly
- hapdiff VCF structural-variant calls compared against the curated SV set

## How to apply

Assemble each diploid dataset (S. cerevisiae SK1×Y12, A. thaliana Col-0×Cvi-0, Col-0×C24, D. melanogaster ISO1×A4, B. taurus Angus×Brahman, Bison×Simmental, HG002) with PECAT via 'pecat.pl unzip cfgfile' using dataset-matched demo/configs templates, and with the four comparison assemblers. For Nanopore runs, enable Clair3 (v0.1-r12) SNP calling via phase_clair3_command and Medaka (v1.7.2) polishing via polish_medaka_command, invoked through singularity or docker if conda conflicts occur. Compute NG50 QUAST-style, Merqury phase block NG50 and Hamming error for every assembly, use hapdiff for structural-variant checks against a curated SV set, BUSCO for completeness, and the reference-based Pomoxis tool for base-level accuracy. Tabulate all metrics per dataset/assembler and validate against headline values such as phase block NG50 79.6/86.1 Mb for B. taurus Bison×Simmental and 59.4/58.0 Mb for HG002 Nanopore R10.

## Related tools

- **PECAT** (Produces the diploid assemblies whose NG50 and phase block NG50 are computed; config templates (cfg_yeast_clr ... cfg_hg002_ont) run via 'pecat.pl unzip cfgfile') — https://github.com/lemene/PECAT
- **Pomoxis** (Reference-based evaluation tool for base-level assembly accuracy) — https://github.com/nanoporetech/pomoxis
- **hapdiff** (SV calling between haplotype pairs of the assembly, compared against a curated SV set) — https://github.com/KolmogorovLab/hapdiff
- **Clair3** (SNP calling on Nanopore assemblies/reads (v0.1-r12) feeding phasing metrics) — https://github.com/HKU-BAL/Clair3
- **Medaka** (Consensus polishing (v1.7.2) of Nanopore assemblies prior to metric computation) — https://github.com/nanoporetech/medaka
- **minimap2** (Read-overlap and read-to-contig alignment used within the PECAT pipeline whose assemblies are evaluated) — https://github.com/lh3/minimap2

## Evaluation signals

- Reproduced metric values match published Tables 1-2 values within the tolerance expected for the fixed parameter settings
- PECAT phase block NG50 for HG002 Nanopore R10 equals 59.4/58.0 Mb
- PECAT phase block NG50 for B. taurus (Bison×Simmental) Nanopore R9 equals 79.6/86.1 Mb, exceeding the trio-binning method's 68.5/70.6 Mb
- PacBio CLR NG50 values for PECAT reproduce 0.8/0.8, 14.3/7.8, 24.5/11.9, 72.4/2.8 Mb across the four PacBio datasets
- Hamming error and hapdiff SV calls are consistent with the curated set of structural variants for each assembly

## Limitations

- PECAT needs higher data coverage to obtain high-quality phased assembly; extracting the longest 80X raw reads (or all reads if <80X) is required for PECAT and NECAT
- SNP calling accuracy is low in low heterozygosity rate regions, which can affect phasing-based metrics
- Clair3 (v0.1-r12) and Medaka (v1.7.2) may hit conda conflicts; the paper invokes them via singularity or docker instead
- Pomoxis evaluation is reference-based, so it cannot be applied to assemblies lacking a suitable reference

## Evidence

- [abstract] phase block NG50 with 59.4/58.0 Mb for HG002 using Nanopore R10 reads: "phase block NG50 with 59.4/58.0 Mb for HG002 using Nanopore R10 reads"
- [discussion] We use the reference-based evaluation tool Pomoxis: "We use the reference-based evaluation tool Pomoxis (https://github.com/nanoporetech/pomoxis)"
- [discussion] using hapdiff and compare them against the curated set of SVs: "using hapdiff (https://github.com/KolmogorovLab/hapdiff) and compare them against the curated set of SVs"
- [discussion] we use Medaka to further improve the assembly quality: "we use Medaka (https://github.com/nanoporetech/medaka) to further improve the assembly quality."
- [readme] Pomoxis comprises a set of basic bioinformatic tools: [evidence span withheld — anchor status: external_doc]
