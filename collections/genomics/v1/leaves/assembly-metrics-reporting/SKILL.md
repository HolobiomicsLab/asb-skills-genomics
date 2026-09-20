---
name: assembly-metrics-reporting
description: Use when you need to quantify inconsistent-overlap reduction between
  PECAT's first-round (haplotype-collapsed) and second-round (SNP-phased) string graphs
  on diploid long-read datasets, e.g. to validate that phasing-based overlap filtering
  worked.
when_to_use_negative:
- Input data is Nanopore duplex or PacBio HiFi, where the article reports PECAT achieves
  mediocre performance and corrected-read SNP calling alone is insufficient.
- The goal is generic contiguity reporting (e.g. NG50/QUAST-style metrics) rather
  than quantifying phasing-driven inconsistent-overlap removal between the two assembly
  rounds.
- The genomic regions under evaluation have a very low heterozygosity rate, where
  the article warns SNP calling accuracy is still low, making the inconsistent-overlap
  percentages unreliable.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: PECAT
  role: Runs the two-round diploid assembly whose first- and second-round simplified
    string graphs supply the inconsistent overlap fractions being tabulated
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Produces read-to-read overlaps and maps corrected reads to first-round primary.fasta
    (phase_rd2ctg_options) so SNP alleles can be scored per overlapping read pair
  repo: https://github.com/lh3/minimap2
- name: Clair3
  role: Calls SNPs on raw Nanopore reads to enable inconsistent overlap identification
    with the loose 6-allele threshold
  repo: https://github.com/HKU-BAL/Clair3
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:8fdf1f2d2d0d97999463b5dc4700dcffa52e492ada6fab36cb490fb3ea2d1038
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Clair3
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# assembly-metrics-reporting

## Summary

Quantifies the percentage of inconsistent overlaps removed from PECAT's simplified string graphs by comparing the first-round haplotype-collapsed assembly with the second-round SNP-phased assembly, and tabulates round-1 vs round-2 fractions across the seven diploid datasets (e.g. 36.98%→0.32% for the first dataset).

## When to use

Use when you need to quantify inconsistent-overlap reduction between PECAT's first-round (haplotype-collapsed) and second-round (SNP-phased) string graphs on diploid long-read datasets, e.g. to validate that phasing-based overlap filtering worked.

## When NOT to use

- Input data is Nanopore duplex or PacBio HiFi, where the article reports PECAT achieves mediocre performance and corrected-read SNP calling alone is insufficient.
- The goal is generic contiguity reporting (e.g. NG50/QUAST-style metrics) rather than quantifying phasing-driven inconsistent-overlap removal between the two assembly rounds.
- The genomic regions under evaluation have a very low heterozygosity rate, where the article warns SNP calling accuracy is still low, making the inconsistent-overlap percentages unreliable.

## Inputs

- Corrected long reads (FASTA/FASTQ) from a diploid dataset (e.g. S. cerevisiae SK1×Y12, A. thaliana Col-0×Cvi-0, D. melanogaster ISO1×A4, B. taurus Angus×Brahman, A. thaliana Col-0×C24, B. taurus Bison×Simmental, HG002)
- PECAT config file (platform-matching template such as cfg_cattle_ont from demo/configs)
- First-round haplotype-collapsed contigs ($PROJECT/3-assemble/primary.fasta from 'pecat.pl unzip')
- minimap2 read-to-read and read-to-contig overlap alignments (align_rd2rd_options, phase_rd2ctg_options)
- Heterozygous SNP calls from corrected reads (8-allele strict threshold) and/or raw reads (6-allele loose threshold; Clair3 for Nanopore raw reads)

## Outputs

- inconsistent_overlap_fractions.tsv with round-1 and round-2 inconsistent overlap percentages per dataset (reported round-1: 36.98, 16.00, 15.94, 36.55, 16.62, 18.08, 37.45%; round-2: 0.32, 0.57, 0.90, 0.63, 0.79, 0.03, 1.09%)
- First-round primary.fasta and second-round phased assembly contigs
- List of inconsistent overlaps removed from the simplified string graph (phase_filter_options step)

## How to apply

Run the first round of PECAT ('pecat.pl unzip cfgfile', e.g. with the platform-matching template cfg_cattle_ont from demo/configs) to produce haplotype-collapsed contigs in $PROJECT/3-assemble/primary.fasta, and record the fraction of inconsistent overlaps removed from this first-round simplified string graph. For the second round, map corrected reads to primary.fasta with minimap2 (phase_rd2ctg_options; phase_use_reads=1), call heterozygous SNPs with phase_phase_options — the strict threshold requiring a pair of reads to contain 8 different SNP alleles for corrected reads, or the loose 6-allele threshold for raw reads (Clair3 for Nanopore raw reads) — counting only SNP sites whose coverage is in the range [10, 1000]. Remove the flagged inconsistent overlaps with phase_filter_options and reassemble with asm2_assemble_options, then record the second-round inconsistent-overlap percentage from the simplified string graph. PECAT can do this cheaply because it reuses the overlaps built in the first round and only simplifies them before reassembly. Tabulate round-1 vs round-2 percentages per dataset in inconsistent_overlap_fractions.tsv and verify that round-2 < round-1 for every dataset.

## Related tools

- **PECAT** (Runs the two-round diploid assembly whose first- and second-round simplified string graphs supply the inconsistent overlap fractions being tabulated) — https://github.com/lemene/PECAT
- **minimap2** (Produces read-to-read overlaps and maps corrected reads to first-round primary.fasta (phase_rd2ctg_options) so SNP alleles can be scored per overlapping read pair) — https://github.com/lh3/minimap2
- **Clair3** (Calls SNPs on raw Nanopore reads to enable inconsistent overlap identification with the loose 6-allele threshold) — https://github.com/HKU-BAL/Clair3

## Examples

```
pecat.pl unzip cfg_cattle_ont
```

## Evaluation signals

- For every one of the seven diploid datasets, the round-2 inconsistent-overlap percentage must be strictly lower than the round-1 percentage.
- Round-1 values reproduce 36.98%, 16.00%, 15.94%, 36.55%, 16.62%, 18.08%, 37.45% and round-2 values reproduce 0.32%, 0.57%, 0.90%, 0.63%, 0.79%, 0.03%, 1.09% within reporting tolerance in inconsistent_overlap_fractions.tsv.
- All percentages fall in [0, 100] and each dataset has both a round-1 and a round-2 entry in inconsistent_overlap_fractions.tsv.
- Only SNP sites with coverage in the range [10, 1000] contributed to inconsistent-overlap identification.
- The second round reused first-round overlaps rather than recomputing them, only removing inconsistent ones before reassembly.

## Limitations

- The accuracy of the corrected-read SNP call is still low for low heterozygosity rate regions, which can distort inconsistent-overlap percentages.
- PECAT needs higher coverage of data to obtain a high-quality phased assembly; low-coverage datasets may not reproduce the reported reductions.
- Details of the assembly parameters used in this study are only described in Supplementary Notes 4 and 5, not in the main text, complicating exact reproduction.

## Evidence

- [abstract] We combine a corrected read SNP caller and a raw read SNP caller to further improve the identification of inconsistent overlaps in the string graph.: "We combine a corrected read SNP caller and a raw read SNP caller to further improve the identification of inconsistent overlaps in the string graph."
- [discussion] Reuse first-round overlaps and simplify: "reuse the overlaps built in the first round of assembly and just simplify the overlaps by removing inconsistent ones before reassembling the diploid genome"
- [discussion] Strict corrected-read threshold: "using corrected reads with the strict threshold that the pair of reads should contain 8 different SNP alleles"
- [discussion] Coverage range filter for SNP sites: "(1) Its coverage is in the range [10, 1000]."
- [readme] PECAT README description: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
