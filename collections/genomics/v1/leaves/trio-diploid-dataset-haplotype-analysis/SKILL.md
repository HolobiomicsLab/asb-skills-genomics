---
name: trio-diploid-dataset-haplotype-analysis
description: Use when corrected long reads from diploid (trio-classified) datasets
  must be scored for how faithfully each parental haplotype's k-mers are retained,
  e.g. comparing haplotype-aware vs. standard error correction.
when_to_use_negative:
- Input reads are Nanopore duplex or PacBio HiFi, where the article reports PECAT
  achieves mediocre performance (Supplementary Table 12 and Supplementary Figs. 21–24)
- No trio classification or parent-specific k-mer sets are available, since the consistency/completeness
  metrics are defined over parent-specific k-mer pairs
- SNP-calling from corrected reads alone is the goal — the article notes that after
  error correction it is not possible to accurately call SNPs just by aligning corrected
  reads as Hifiasm did for HiFi reads
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: PECAT
  role: Haplotype-aware error correction via `pecat.pl correct cfgfile`, extracting
    longest raw reads and producing 1-correct/corrected_reads.fasta; its consistency
    values are the target being validated
  repo: https://github.com/lemene/PECAT
- name: NECAT
  role: Correction method compared against PECAT; extracts the longest 80X raw reads
    (or all reads below 80X) for error correction
- name: Canu
  role: Correction module run on the same inputs as a baseline corrector
- name: FALCON
  role: Correction module run on the same inputs as a baseline corrector
- name: MECAT2
  role: Correction module run on the same inputs as a baseline corrector
- name: minimap2
  role: Used in the PECAT pipeline for candidate read overlaps (e.g. '-x ava-pb' for
    PacBio CLR reads) prior to correction-based evaluation
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:02d3aa454bc71a81664f47675ba564ff943372d83a213e54db79f5e850dd6382
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Canu
  - FALCON
  - FALCON-Unzip
  - MECAT2
  - NECAT
  - PECAT
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# trio-diploid-dataset-haplotype-analysis

## Summary

Evaluates haplotype-specific k-mer consistency and completeness of error-corrected long reads on seven trio-classified diploid datasets (S. cerevisiae SK1×Y12, A. thaliana Col-0×Cvi-0, A. thaliana Col-0×C24, D. melanogaster ISO1×A4, B. taurus Angus×Brahman, B. taurus Bison×Simmental, HG002), benchmarking PECAT's haplotype-aware correction against Canu, FALCON, MECAT2, and NECAT.

## When to use

Use when corrected long reads from diploid (trio-classified) datasets must be scored for how faithfully each parental haplotype's k-mers are retained, e.g. comparing haplotype-aware vs. standard error correction.

## When NOT to use

- Input reads are Nanopore duplex or PacBio HiFi, where the article reports PECAT achieves mediocre performance (Supplementary Table 12 and Supplementary Figs. 21–24)
- No trio classification or parent-specific k-mer sets are available, since the consistency/completeness metrics are defined over parent-specific k-mer pairs
- SNP-calling from corrected reads alone is the goal — the article notes that after error correction it is not possible to accurately call SNPs just by aligning corrected reads as Hifiasm did for HiFi reads

## Inputs

- Raw long reads (Nanopore R9/R10 or PacBio CLR) for the seven trio-classified diploid datasets: S. cerevisiae (SK1×Y12), A. thaliana (Col-0×Cvi-0), A. thaliana (Col-0×C24), D. melanogaster (ISO1×A4), B. taurus (Angus×Brahman), B. taurus (Bison×Simmental), HG002
- Trio parent-specific k-mer sets inherited from each parental haplotype (k-mers present at least 4 times in the 40X longest reads)
- PECAT config file per dataset (from demo/configs templates, with correct genome_size)
- Corrected read FASTA files from PECAT (1-correct/corrected_reads.fasta) and from Canu, FALCON, MECAT2, NECAT

## Outputs

- Haplotype-specific k-mer consistency per corrector per dataset (sum of max(kp,km) / sum of kp+km over parent-specific k-mer pairs)
- Haplotype-specific k-mer completeness per corrector per dataset (% parent-specific k-mers retained in the 40X longest corrected reads)
- A single CSV table of consistency and completeness values for all five correctors across all seven datasets, comparable to Fig. 2c-e

## How to apply

For each dataset, prepare a PECAT config from the demo/configs templates with the correct genome_size and run the correction step via `pecat.pl correct cfgfile`, which extracts the longest raw reads and writes corrected reads to 1-correct/corrected_reads.fasta; run the correction modules of Canu, FALCON, MECAT2, and NECAT on the same inputs. For each corrected read set, count parent-specific k-mers inherited from each trio haplotype, restricting to k-mers occurring at least 4 times in the 40X longest raw reads. Compute consistency as sum of max(kp,km) divided by sum of (kp+km) over parent-specific k-mer pairs, and completeness as the percentage of parent-specific k-mers retained in the 40X longest corrected reads. Assemble the consistency and completeness values for all five correctors across all seven datasets into a single CSV table. Note that PECAT and NECAT extract the longest 80X raw reads (or all reads if coverage is below 80X) for error correction.

## Related tools

- **PECAT** (Haplotype-aware error correction via `pecat.pl correct cfgfile`, extracting longest raw reads and producing 1-correct/corrected_reads.fasta; its consistency values are the target being validated) — https://github.com/lemene/PECAT
- **NECAT** (Correction method compared against PECAT; extracts the longest 80X raw reads (or all reads below 80X) for error correction)
- **Canu** (Correction module run on the same inputs as a baseline corrector)
- **FALCON** (Correction module run on the same inputs as a baseline corrector)
- **MECAT2** (Correction module run on the same inputs as a baseline corrector)
- **minimap2** (Used in the PECAT pipeline for candidate read overlaps (e.g. '-x ava-pb' for PacBio CLR reads) prior to correction-based evaluation) — https://github.com/lh3/minimap2

## Examples

```
pecat.pl correct cfgfile
```

## Evaluation signals

- PECAT haplotype-specific k-mer consistency reaches at least 99.4% on every one of the seven datasets, reproducing the ordering of correctors reported in Fig. 2c-e
- Consistencies of the competing methods (Canu, FALCON, MECAT2, NECAT) are all <= 94.8% on the evaluated datasets
- K-mer counting is restricted to parent-specific k-mers occurring at least 4 times in the 40X longest reads, and completeness is measured on the 40X longest corrected reads
- Corrected reads are present at the expected path 1-correct/corrected_reads.fasta for each PECAT run
- The assembled CSV contains values for all five correctors across all seven diploid datasets

## Limitations

- PECAT's SNP call accuracy is still low for low heterozygosity rate regions, and error correction may not distinguish small errors from heterozygote alleles in very low heterozygosity regions
- PECAT achieves mediocre performance on Nanopore duplex and PacBio HiFi reads, so this benchmark's conclusions should not be extrapolated to those data types
- PECAT needs higher data coverage to obtain high-quality phased assemblies

## Evidence

- [methods] For each corrected read set, count parent-specific k-mers inherited from each haplotype of the trio, restricting to k-mers occurring at least 4 times in the 40X longest reads.: [evidence span withheld — anchor status: not_found]
- [methods] Consistency and completeness formulas.: [evidence span withheld — anchor status: not_found]
- [methods] PECAT correction invocation and output path.: [evidence span withheld — anchor status: not_found]
- [methods] Validation criterion for the benchmark.: [evidence span withheld — anchor status: not_found]
- [discussion] The seven diploid datasets evaluated.: "seven diploid datasets (Supplementary Note 1 and Supplementary Table 17): S. cerevisiae (SK1×Y12), A. thaliana (Col-0×Cvi-0), D. melanogaster (ISO1×A4)"
- [discussion] Read extraction coverage rule for PECAT/NECAT correction.: "For each of the above data, PECAT and NECAT extract the longest 80X raw reads or all reads, if the dataset is less than 80X, for error correction and assembly."
- [readme] PECAT tool description from its README.: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
