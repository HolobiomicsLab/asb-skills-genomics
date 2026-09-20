---
name: nanopore-snp-calling-and-polishing
description: Use when assembling Nanopore R9/R10 diploid datasets with PECAT, when
  corrected reads alone give insufficient SNP calls for phasing, or when Nanopore
  assemblies require post-assembly polishing before evaluation.
when_to_use_negative:
- Input data are PacBio CLR reads — Clair3-based SNP calling and Medaka polishing
  are enabled specifically for Nanopore datasets
- Input data are Nanopore duplex or PacBio HiFi reads, where PECAT achieves only mediocre
  performance and this polishing path is not the designed workflow
- Running a non-Nanopore comparison assembler (e.g. FALCON-Unzip) whose pipeline does
  not expose phase_clair3_command/polish_medaka_command config hooks
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: Clair3
  role: Calls SNPs again on raw Nanopore reads to improve phasing and inconsistent-overlap
    identification (v0.1-r12+)
  repo: https://github.com/HKU-BAL/Clair3
- name: Medaka
  role: Consensus polishing of Nanopore assemblies via neural networks applied to
    read pileups (v1.7.2+)
  repo: https://github.com/nanoporetech/medaka
- name: PECAT
  role: Host pipeline exposing phase_clair3_command and polish_medaka_command config
    parameters for the Nanopore assembly workflow
  repo: https://github.com/lemene/PECAT
- name: Pomoxis
  role: Reference-based evaluation of the polished assembly's base-level accuracy
  repo: https://github.com/nanoporetech/pomoxis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:f9584f72809520fe96a3f8f51ba32d8d0c167514111484ad49a1ac1124419ecd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/Clair3
  tools:
  - Clair3
  - FALCON-Unzip
  - Medaka
  - NECAT
  - Nanopore R10
  - Nanopore R9
  - PECAT
  - PacBio CLR
  - Pomoxis
  - racon
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nanopore-snp-calling-and-polishing

## Summary

Enables Clair3-based SNP calling on raw Nanopore reads and Medaka consensus polishing within the PECAT diploid assembly pipeline, improving phasing and base-level accuracy when corrected reads alone cannot resolve true heterozygous SNPs.

## When to use

Use when assembling Nanopore R9/R10 diploid datasets with PECAT, when corrected reads alone give insufficient SNP calls for phasing, or when Nanopore assemblies require post-assembly polishing before evaluation.

## When NOT to use

- Input data are PacBio CLR reads — Clair3-based SNP calling and Medaka polishing are enabled specifically for Nanopore datasets
- Input data are Nanopore duplex or PacBio HiFi reads, where PECAT achieves only mediocre performance and this polishing path is not the designed workflow
- Running a non-Nanopore comparison assembler (e.g. FALCON-Unzip) whose pipeline does not expose phase_clair3_command/polish_medaka_command config hooks

## Inputs

- Raw Nanopore R9 or R10 long reads (FASTQ/FASTA)
- PECAT config file generated from dataset-matched template (e.g. cfg_arab_ont, cfg_cattle_ont, cfg_hg002_ont)
- Corrected reads and haplotype-collapsed genome from PECAT error correction stage
- Reference genome (for downstream Pomoxis/hapdiff evaluation of the polished assembly)

## Outputs

- VCF/variant call sets of SNPs on raw Nanopore reads from Clair3
- Medaka-polished haplotype-resolved assembly contigs
- Final diploid assembly suitable for QUAST-style NG50, Merqury phase block NG50/Hamming error, BUSCO and Pomoxis evaluation

## How to apply

For Nanopore (R9 and R10) datasets, set the PECAT config parameters phase_clair3_command and polish_medaka_command to invoke Clair3 v0.1-r12 and Medaka v1.7.2 (via singularity or docker if conda installation conflicts occur) before running 'pecat.pl unzip cfgfile'. Rationale: even after PECAT's haplotype-aware error correction, aligning corrected reads alone cannot accurately call SNPs the way Hifiasm does with HiFi reads, so Clair3 re-calls SNPs on the raw Nanopore reads to strengthen inconsistent-overlap detection in the string graph. After contig construction, Medaka performs consensus/variant refinement to improve assembly quality. For large genomes, add '-f 0.005' or '-f 0.002' to corr_rd2rd_options and align_rd2rd_options alongside this step, and set cleanup=1.

## Related tools

- **Clair3** (Calls SNPs again on raw Nanopore reads to improve phasing and inconsistent-overlap identification (v0.1-r12+)) — https://github.com/HKU-BAL/Clair3
- **Medaka** (Consensus polishing of Nanopore assemblies via neural networks applied to read pileups (v1.7.2+)) — https://github.com/nanoporetech/medaka
- **PECAT** (Host pipeline exposing phase_clair3_command and polish_medaka_command config parameters for the Nanopore assembly workflow) — https://github.com/lemene/PECAT
- **Pomoxis** (Reference-based evaluation of the polished assembly's base-level accuracy) — https://github.com/nanoporetech/pomoxis

## Examples

```
pecat.pl config cfgfile; # set phase_clair3_command and polish_medaka_command for clair3 v0.1-r12 / medaka v1.7.2, then: pecat.pl unzip cfgfile
```

## Evaluation signals

- Polished HG002 Nanopore R10 assembly reaches phase block NG50 of 59.4/58.0 Mb as reported in the paper
- B. taurus (Bison×Simmental) Nanopore R9 assembly reaches phase block NG50 of 79.6/86.1 Mb, exceeding trio-binning's 68.5/70.6 Mb
- Pomoxis reference-based evaluation confirms improved base-level accuracy after Medaka polishing
- Reproduced metric values match published Tables 1-2 within tolerance for the fixed parameter settings
- hapdiff comparison against curated SV sets shows no increase in structural-variant errors relative to published results

## Limitations

- Conda installation of Clair3/Medaka can conflict; invoke clair3 v0.1-r12 and medaka v1.7.2 through singularity or docker instead
- Even after error correction, SNP calls from aligning corrected reads alone are not accurate enough (unlike Hifiasm with HiFi reads), motivating but also constraining this step
- SNP calling accuracy remains low in low heterozygosity rate regions
- PECAT needs higher data coverage for high-quality phased assembly

## Evidence

- [discussion] Therefore, we use Clair3 to call SNPs again for Nanopore reads.: "Therefore, we use Clair3 to call SNPs again for Nanopore reads."
- [discussion] we use Medaka (https://github.com/nanoporetech/medaka) to further improve the assembly quality.: "we use Medaka (https://github.com/nanoporetech/medaka) to further improve the assembly quality."
- [discussion] even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads.: "even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads."
- [readme]  Clair3 v0.1-r12 and Medaka v1.7.2 through singularity or docker if conda conflicts occur: [evidence span withheld — anchor status: not_found]
- [readme] clair3 (optional dependency of PECAT, v0.1-r12+): "+ [clair3](https://github.com/HKU-BAL/Clair3)  (v0.1-r12+) (optional)"
