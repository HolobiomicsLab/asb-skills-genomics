---
name: cross-assembler-benchmarking
description: Use when comparing contiguity and phasing quality of diploid genome assemblies
  produced by multiple long-read assemblers on the same Nanopore R9/R10 or PacBio
  CLR datasets.
when_to_use_negative:
- Evaluating assemblers on PacBio HiFi or Nanopore duplex reads — the paper reports
  PECAT achieves only mediocre performance on Nanopore duplex and PacBio HiFi reads,
  so benchmarking conclusions do not transfer
- Single-haploid (non-diploid) genome assembly comparisons where phase block NG50
  and Hamming error phasing metrics are not meaningful
- Benchmarking against trio-binning pipelines under their native settings — note trio-binning
  methods require additional parental reads that PECAT does not use
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: PECAT
  role: Primary assembler under benchmark; run per dataset from demo/configs templates
    with 'pecat.pl unzip cfgfile'
  repo: https://github.com/lemene/PECAT
- name: NECAT
  role: Comparison assembler using the longest 80X raw reads for error correction
    and assembly
- name: FALCON-Unzip
  role: Comparison assembler that phases with raw reads in its unzip step
- name: minimap2
  role: Candidate overlap detection with '-x ava-pb' for PacBio CLR reads and read-to-contig
    mapping for evaluation
  repo: https://github.com/lh3/minimap2
- name: Clair3
  role: Second-round SNP calling on raw Nanopore reads within the PECAT config (phase_clair3_command)
  repo: https://github.com/HKU-BAL/Clair3
- name: Medaka
  role: Consensus polishing of Nanopore assemblies via polish_medaka_command
  repo: https://github.com/nanoporetech/medaka
- name: racon
  role: Consensus/polishing dependency of the PECAT pipeline (v1.4.21+)
  repo: https://github.com/lbcb-sci/racon
- name: Pomoxis
  role: Reference-based evaluation of assembly base-level accuracy
  repo: https://github.com/nanoporetech/pomoxis
- name: hapdiff
  role: SV calling between haplotype pairs, compared against curated SV sets
  repo: https://github.com/KolmogorovLab/hapdiff
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:54c097c67029e69044de03f31b1fab423fe45972d4d8ed67a3739afc600274d3
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
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# cross-assembler-benchmarking

## Summary

Benchmark PECAT against other diploid assembly pipelines (Canu+purge_dups, FALCON-Unzip, Flye+HapDup, Shasta, NECAT) on seven diploid long-read datasets using NG50, Merqury phase block NG50, Hamming error, BUSCO and Pomoxis metrics.

## When to use

Use when comparing contiguity and phasing quality of diploid genome assemblies produced by multiple long-read assemblers on the same Nanopore R9/R10 or PacBio CLR datasets.

## When NOT to use

- Evaluating assemblers on PacBio HiFi or Nanopore duplex reads — the paper reports PECAT achieves only mediocre performance on Nanopore duplex and PacBio HiFi reads, so benchmarking conclusions do not transfer
- Single-haploid (non-diploid) genome assembly comparisons where phase block NG50 and Hamming error phasing metrics are not meaningful
- Benchmarking against trio-binning pipelines under their native settings — note trio-binning methods require additional parental reads that PECAT does not use

## Inputs

- Seven diploid long-read datasets: PacBio CLR, Nanopore R9 and Nanopore R10 FASTQ (S. cerevisiae SK1×Y12, A. thaliana Col-0×Cvi-0, A. thaliana Col-0×C24, D. melanogaster ISO1×A4, B. taurus Angus×Brahman, B. taurus Bison×Simmental, HG002)
- Deposited assemblies from Zenodo DOI 10.5281/zenodo.8380113
- Dataset-matched PECAT config templates from demo/configs (cfg_yeast_clr, cfg_arab_clr, cfg_dro_clr, cfg_cattle_clr, cfg_arab_ont, cfg_cattle_ont, cfg_hg002_ont)
- Reference genomes and k-mer databases for QUAST, Merqury, BUSCO and Pomoxis evaluation

## Outputs

- Per-dataset, per-assembler table of NG50, Merqury phase block NG50 and Hamming error (Tables 1-2 style)
- BUSCO completeness scores per assembly
- Pomoxis reference-based base-level accuracy (QV) per assembly
- hapdiff structural-variant call sets (VCF) comparing haplotype pairs against curated SV sets

## How to apply

Obtain the seven diploid datasets (S. cerevisiae SK1×Y12, A. thaliana Col-0×Cvi-0, Col-0×C24, D. melanogaster ISO1×A4, B. taurus Angus×Brahman, Bison×Simmental, HG002) and the deposited assemblies from Zenodo DOI 10.5281/zenodo.8380113. For each dataset, assemble with PECAT using the dataset-matched config template (e.g. cfg_cattle_ont) via 'pecat.pl unzip cfgfile', adding '-f 0.005' or '-f 0.002' to corr_rd2rd_options and align_rd2rd_options for large genomes and cleanup=1; for Nanopore data enable Clair3 (v0.1-r12) SNP calling and Medaka (v1.7.2) polishing. Extract the longest 80X raw reads for PECAT and NECAT on datasets exceeding 80X coverage, as in the paper. Run the comparison assemblers (FALCON-Unzip, Canu+purge_dups, Flye+HapDup, Shasta) on the same inputs, then compute NG50 (QUAST-style), Merqury phase block NG50 and Hamming error, BUSCO completeness, Pomoxis base-level accuracy, and use hapdiff to check structural variants. Validate reproduced values against the published Tables 1-2 headline numbers (e.g. phase block NG50 79.6/86.1 Mb for B. taurus Bison×Simmental and 59.4/58.0 Mb for HG002 Nanopore R10).

## Related tools

- **PECAT** (Primary assembler under benchmark; run per dataset from demo/configs templates with 'pecat.pl unzip cfgfile') — https://github.com/lemene/PECAT
- **NECAT** (Comparison assembler using the longest 80X raw reads for error correction and assembly)
- **FALCON-Unzip** (Comparison assembler that phases with raw reads in its unzip step)
- **minimap2** (Candidate overlap detection with '-x ava-pb' for PacBio CLR reads and read-to-contig mapping for evaluation) — https://github.com/lh3/minimap2
- **Clair3** (Second-round SNP calling on raw Nanopore reads within the PECAT config (phase_clair3_command)) — https://github.com/HKU-BAL/Clair3
- **Medaka** (Consensus polishing of Nanopore assemblies via polish_medaka_command) — https://github.com/nanoporetech/medaka
- **racon** (Consensus/polishing dependency of the PECAT pipeline (v1.4.21+)) — https://github.com/lbcb-sci/racon
- **Pomoxis** (Reference-based evaluation of assembly base-level accuracy) — https://github.com/nanoporetech/pomoxis
- **hapdiff** (SV calling between haplotype pairs, compared against curated SV sets) — https://github.com/KolmogorovLab/hapdiff

## Examples

```
pecat.pl config cfgfile && pecat.pl unzip cfgfile
```

## Evaluation signals

- Reproduced PECAT phase block NG50 for B. taurus (Bison×Simmental) Nanopore reads matches 79.6/86.1 Mb within tolerance
- Reproduced HG002 Nanopore R10 phase block NG50 matches 59.4/58.0 Mb
- Reproduced PECAT NG50 values on PacBio CLR datasets match 0.8/0.8, 14.3/7.8, 24.5/11.9, 72.4/2.8 Mb
- Each assembler's assemblies are evaluated with the same metric stack (QUAST-style NG50, Merqury phase block NG50/Hamming error, BUSCO, Pomoxis) for a fair comparison
- hapdiff SV calls against the curated SV set show consistency between the assembly pair being benchmarked

## Limitations

- Assembly parameters used in the study are only described in Supplementary Notes 4 and 5, not in the main text, complicating exact reproduction
- PECAT needs higher data coverage to obtain a high-quality phased assembly
- SNP calling accuracy is low in low heterozygosity rate regions, affecting phasing metrics there

## Evidence

- [methods] The assemblies are available at https://doi.org/10.5281/zenodo.8380113: "The assemblies are available at https://doi.org/10.5281/zenodo.8380113"
- [discussion] PECAT and NECAT extract longest 80X reads: "For each of the above data, PECAT and NECAT extract the longest 80X raw reads or all reads, if the dataset is less than 80X, for error correction and assembly."
- [discussion] Merqury/hapdiff evaluation: "using hapdiff (https://github.com/KolmogorovLab/hapdiff) and compare them against the curated set of SVs"
- [discussion] Pomoxis evaluation tool: "We use the reference-based evaluation tool Pomoxis (https://github.com/nanoporetech/pomoxis)"
- [abstract] Phase block NG50 for HG002 R10: "phase block NG50 with 59.4/58.0 Mb for HG002 using Nanopore R10 reads"
- [discussion] minimap2 overlap parameters: "we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads"
