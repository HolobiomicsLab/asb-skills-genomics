---
name: assembler-corrector-benchmark-comparison
description: Use when comparing how error correctors or assemblers preserve heterozygote
  alleles in corrected long reads across heterozygosity rates, e.g. validating a haplotype-aware
  corrector against Canu/MECAT/NECAT baselines.
when_to_use_negative:
- Evaluating assemblers on Nanopore duplex or PacBio HiFi reads — the paper reports
  PECAT achieves only mediocre performance on these high-accuracy data types, so the
  correction-retention benchmark is not representative
- Assessing haplotype-resolved assembly quality (phase block NG50, QV) on real diploid
  datasets rather than correction-stage allele retention on simulated reads
- Quantifying SNP calling from assemblies or raw reads — the paper notes corrected
  reads alone cannot accurately call SNPs the way Hifiasm does with HiFi reads
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3693
tools:
- name: PECAT
  role: 'Tool under test: haplotype-aware error correction run via ''pecat.pl correct
    cfgfile''; repo and docker image provided by authors'
  repo: github.com/lemene/PECAT
- name: minimap2
  role: Detects read-to-read overlaps for PECAT correction ('-x ava-pb' for PacBio
    CLR) and aligns corrected reads for SNP allele quantification
  repo: https://github.com/lh3/minimap2
- name: Canu
  role: Baseline error corrector run with default pipeline on the same simulated read
    sets
- name: MECAT
  role: Baseline error corrector run with default pipeline on the same simulated read
    sets
- name: NECAT
  role: Baseline error corrector; paper notes PECAT and NECAT extract the longest
    80X raw reads for error correction and assembly
- name: pomoxis
  role: Reference-based evaluation tool used for assessing assembly/correction quality
    in this study
  repo: https://github.com/nanoporetech/pomoxis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:04ee09471156b108aa39695f7bdb16a87366803ee814f721adf13a2059fc7bbb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Canu
  - MECAT
  - NECAT
  - PECAT
  - minimap2
  - pomoxis
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3693
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# assembler-corrector-benchmark-comparison

## Summary

Benchmark error-correction/assembling tools (PECAT vs Canu, MECAT, NECAT) on simulated diploid long reads by quantifying how well heterozygous SNP alleles survive correction, replicating the Fig. 2a SNP allele accuracy comparison.

## When to use

Use when comparing how error correctors or assemblers preserve heterozygote alleles in corrected long reads across heterozygosity rates, e.g. validating a haplotype-aware corrector against Canu/MECAT/NECAT baselines.

## When NOT to use

- Evaluating assemblers on Nanopore duplex or PacBio HiFi reads — the paper reports PECAT achieves only mediocre performance on these high-accuracy data types, so the correction-retention benchmark is not representative
- Assessing haplotype-resolved assembly quality (phase block NG50, QV) on real diploid datasets rather than correction-stage allele retention on simulated reads
- Quantifying SNP calling from assemblies or raw reads — the paper notes corrected reads alone cannot accurately call SNPs the way Hifiasm does with HiFi reads

## Inputs

- Simulated diploid PacBio CLR long-read FASTQ/FASTA set with haplotype truth labels
- Simulated diploid Nanopore long-read FASTQ/FASTA set with haplotype truth labels
- Reference genome (diploid truth sequences) for allele quantification
- Simulated data generated at heterozygosity rates 0.0001 and >=0.0005

## Outputs

- Corrected reads per tool (e.g. PECAT's $PROJECT/1-correct/corrected_reads.fasta)
- Per-tool, per-heterozygosity SNP allele accuracy table (snp_allele_accuracy_fig2a.csv, mirroring Fig. 2a layout)

## How to apply

Simulate diploid long-read datasets with PacBio CLR and Nanopore error profiles from a reference genome at heterozygosity rates 0.0001 and >=0.0005, keeping per-read haplotype truth labels. For each tool, run the correction stage: for PECAT generate a config with 'pecat.pl config cfgfile', set genome_size and coverage, then run 'pecat.pl correct cfgfile' (which uses minimap2 read-to-read overlaps and writes $PROJECT/1-correct/corrected_reads.fasta); run Canu, MECAT, and NECAT correction with their default pipelines on the same simulated reads. Align corrected reads against the simulated diploid truth and compute the fraction of heterozygous SNP alleles retained per tool and heterozygosity condition, tabulating results to snp_allele_accuracy_fig2a.csv. The rationale: heterozygote retention is the discriminating metric because correct-then-assemble pipelines that collapse alleles lose phasing information; validation requires PECAT >99% SNP allele accuracy at heterozygosity >=0.0005 while Canu <80% and MECAT/NECAT <60%.

## Related tools

- **PECAT** (Tool under test: haplotype-aware error correction run via 'pecat.pl correct cfgfile'; repo and docker image provided by authors) — github.com/lemene/PECAT
- **minimap2** (Detects read-to-read overlaps for PECAT correction ('-x ava-pb' for PacBio CLR) and aligns corrected reads for SNP allele quantification) — https://github.com/lh3/minimap2
- **Canu** (Baseline error corrector run with default pipeline on the same simulated read sets)
- **MECAT** (Baseline error corrector run with default pipeline on the same simulated read sets)
- **NECAT** (Baseline error corrector; paper notes PECAT and NECAT extract the longest 80X raw reads for error correction and assembly)
- **pomoxis** (Reference-based evaluation tool used for assessing assembly/correction quality in this study) — https://github.com/nanoporetech/pomoxis

## Examples

```
pecat.pl config cfgfile && pecat.pl correct cfgfile  # PECAT correction stage; corrected reads land in $PROJECT/1-correct/corrected_reads.fasta
```

## Evaluation signals

- PECAT SNP allele accuracy >99% at heterozygosity rates >=0.0005 in corrected reads
- Canu SNP allele accuracy falls below 80% and MECAT/NECAT fall below 60% at the same heterozygosity conditions
- PECAT correction output exists at $PROJECT/1-correct/corrected_reads.fasta after 'pecat.pl correct cfgfile'
- All four correctors were run on the identical simulated read sets and heterozygosity conditions (paired comparison, not separate simulations)
- CSV rows match the Fig. 2a layout with per-tool accuracy at each heterozygosity rate including the 0.0001 low-heterozygosity condition

## Limitations

- PECAT's error correction cannot distinguish small errors from heterozygote alleles in very low heterozygosity rate regions, and SNP call accuracy is low in low-heterozygosity regions
- PECAT needs higher data coverage to obtain a high-quality phased assembly, so comparisons at low coverage may not transfer
- Performance conclusions are for Nanopore R9/CLR-type noisy reads; PECAT achieves mediocre performance on Nanopore duplex and PacBio HiFi reads

## Evidence

- [other] PECAT's haplotype-aware error correction retains SNP alleles with greater than 99% accuracy: [evidence span withheld — anchor status: not_found]
- [readme] PECAT runs correct via pecat.pl with minimap2 overlaps: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
- [discussion] minimap2 overlap preset from article methods: "we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads"
- [discussion] Baseline extraction scheme for NECAT comparison: "For each of the above data, PECAT and NECAT extract the longest 80X raw reads or all reads, if the dataset is less than 80X, for error correction and assembly."
- [discussion] Limitation on low heterozygosity: "our current error correction method may not be effective enough to distinguish small errors from heterozygote alleles in very low heterozygosity rate regions"
- [discussion] Limitation on high-accuracy read types: "therefore it achieves mediocre performance on Nanopore duplex and PacBio HiFi reads (Supplementary Table 12 and Supplementary Figs. 21–24)"
