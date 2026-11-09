---
name: parent-specific-kmer-completeness-assessment
description: Use when comparing error correctors on trio-classified diploid long-read
  datasets and you need to measure, per parental haplotype, which parent-specific
  k-mers survive correction in the 40X longest corrected reads.
when_to_use_negative:
- Data without trio classification or parent-specific k-mer annotations — the kp/km
  consistency and completeness metrics cannot be defined
- Evaluating correctors on Nanopore duplex or PacBio HiFi reads, where PECAT itself
  achieves only mediocre performance
- Assessing polished assembly base accuracy against a reference (use reference-based
  tools like Pomoxis instead of k-mer retention)
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: PECAT
  role: Haplotype-aware corrector under evaluation; run via `pecat.pl correct cfgfile`
    to produce 1-correct/corrected_reads.fasta whose k-mer consistency is expected
    to reach >= 99.4%
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Used within the PECAT pipeline to find candidate overlaps between raw reads
    ("-x ava-pb" for PacBio CLR reads), the overlaps on which correction and phasing
    operate
  repo: https://github.com/lh3/minimap2
- name: Canu / FALCON / MECAT2 / NECAT
  role: Baseline error correctors run on the same raw inputs to provide comparison
    consistency/completeness values (all <= 94.8% consistency)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:22ae5f285660f040238c52453294f30ab5d69282a4c75ae4bd3aa0926d290801
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Canu / FALCON / MECAT2 / NECAT
  - FALCON-Unzip
  - NECAT
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# parent-specific-kmer-completeness-assessment

## Summary

Evaluates haplotype-specific k-mer consistency and completeness of corrected long reads on trio-classified diploid datasets, to quantify how well an error corrector (e.g., PECAT vs Canu, FALCON, MECAT2, NECAT) retains heterozygote alleles from each parental haplotype.

## When to use

Use when comparing error correctors on trio-classified diploid long-read datasets and you need to measure, per parental haplotype, which parent-specific k-mers survive correction in the 40X longest corrected reads.

## When NOT to use

- Data without trio classification or parent-specific k-mer annotations — the kp/km consistency and completeness metrics cannot be defined
- Evaluating correctors on Nanopore duplex or PacBio HiFi reads, where PECAT itself achieves only mediocre performance
- Assessing polished assembly base accuracy against a reference (use reference-based tools like Pomoxis instead of k-mer retention)

## Inputs

- Raw long reads for the seven trio-classified diploid datasets (Nanopore R9, PacBio CLR or Nanopore R10)
- Parent-specific k-mer sets inherited from each haplotype of the trio, filtered to k-mers occurring at least 4 times in the 40X longest raw reads
- Corrected read sets from each corrector, e.g. PECAT output 1-correct/corrected_reads.fasta and equivalent outputs of Canu, FALCON, MECAT2, NECAT
- A PECAT config per dataset (from demo/configs templates) with the correct genome_size

## Outputs

- Haplotype-specific k-mer consistency per corrector per dataset (sum of max(kp,km) / sum of (kp+km) over parent-specific k-mer pairs)
- Haplotype-specific k-mer completeness per corrector per dataset (percentage of parent-specific k-mers retained in the 40X longest corrected reads)
- A single CSV table of consistency and completeness values for all five correctors across all seven datasets, reproducing the ordering of correctors in Fig. 2c-e

## How to apply

Start from the raw long reads of the seven trio-classified diploid datasets (S. cerevisiae SK1×Y12, A. thaliana Col-0×Cvi-0, A. thaliana Col-0×C24, D. melanogaster ISO1×A4, B. taurus Angus×Brahman, B. taurus Bison×Simmental, HG002) and run the correction step of each corrector, e.g. `pecat.pl correct cfgfile` which writes 1-correct/corrected_reads.fasta. Count parent-specific k-mers inherited from each haplotype of the trio, restricting to k-mers occurring at least 4 times in the 40X longest raw reads. Compute haplotype-specific k-mer consistency as sum of max(kp,km) divided by sum of (kp+km) over parent-specific k-mer pairs, and completeness as the percentage of parent-specific k-mers retained in the 40X longest corrected reads. Assemble consistency and completeness for all five correctors (PECAT, Canu, FALCON, MECAT2, NECAT) across all seven datasets into a single CSV table. The rationale is that PECAT's haplotype-aware correction should keep most heterozygote alleles, so its consistency should far exceed collapsed-haplotype correctors.

## Related tools

- **PECAT** (Haplotype-aware corrector under evaluation; run via `pecat.pl correct cfgfile` to produce 1-correct/corrected_reads.fasta whose k-mer consistency is expected to reach >= 99.4%) — https://github.com/lemene/PECAT
- **minimap2** (Used within the PECAT pipeline to find candidate overlaps between raw reads ("-x ava-pb" for PacBio CLR reads), the overlaps on which correction and phasing operate) — https://github.com/lh3/minimap2
- **Canu / FALCON / MECAT2 / NECAT** (Baseline error correctors run on the same raw inputs to provide comparison consistency/completeness values (all <= 94.8% consistency))

## Examples

```
pecat.pl correct cfgfile   # produces 1-correct/corrected_reads.fasta, then compute sum(max(kp,km))/sum(kp+km) over parent-specific k-mer pairs
```

## Evaluation signals

- PECAT haplotype-specific k-mer consistency reaches at least 99.4% on every one of the seven datasets
- Consistency values for Canu, FALCON, MECAT2, and NECAT are all <= 94.8% on the same datasets
- Completeness values are reproducible as the percentage of parent-specific k-mers (occurring >= 4 times in the 40X longest raw reads) retained in the 40X longest corrected reads
- The assembled CSV reproduces the ordering of correctors reported in Fig. 2c-e
- The k-mer universe is restricted to parent-specific k-mer pairs (kp, km) from the trio, not all genomic k-mers

## Limitations

- PECAT needs higher data coverage to obtain a high-quality phased assembly; correcting on a small subset of reads changes which parent-specific k-mers can be retained
- SNP calling accuracy is low in low heterozygosity rate regions, and error correction may not distinguish small errors from heterozygote alleles in very low heterozygosity regions
- PECAT achieves only mediocre performance on Nanopore duplex and PacBio HiFi reads, so k-mer completeness comparisons there are less favorable

## Evidence

- [discussion] we first develop a haplotype-aware error-correct method to keep most of the heterozygote alleles while correcting sequencing errors: "we first develop a haplotype-aware error-correct method to keep most of the heterozygote alleles while correcting sequencing errors"
- [results] PECAT-corrected reads have haplotype-specific k-mer consistency >= 99.4% on all datasets, exceeding other methods whose consistencies are all <= 94.8%.: [evidence span withheld — anchor status: not_found]
- [methods] count parent-specific k-mers inherited from each haplotype of the trio, restricting to k-mers occurring at least 4 times in the 40X longest reads: [evidence span withheld — anchor status: not_found]
- [discussion] seven diploid datasets (Supplementary Note 1 and Supplementary Table 17): S. cerevisiae (SK1×Y12), A. thaliana (Col-0×Cvi-0): "seven diploid datasets (Supplementary Note 1 and Supplementary Table 17): S. cerevisiae (SK1×Y12), A. thaliana (Col-0×Cvi-0)"
- [readme] PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method.: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
