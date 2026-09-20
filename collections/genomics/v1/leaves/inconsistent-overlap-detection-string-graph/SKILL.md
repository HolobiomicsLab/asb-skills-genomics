---
name: inconsistent-overlap-detection-string-graph
description: 'Use when reconstructing haplotype-specific contigs from long noisy reads:
  after a first-round haplotype-collapsed assembly, to phase reads by SNP alleles
  and flag cross-haplotype overlaps before reassembly.'
when_to_use_negative:
- Input reads are PacBio HiFi or Nanopore duplex — the article reports PECAT achieves
  only mediocre performance on these accurate read types
- SNP sites lie in very low heterozygosity-rate regions, where SNP calling accuracy
  is too low to reliably distinguish haplotypes
- Dataset coverage is below the SNP-calling floor (sites outside coverage range [10,
  1000] cannot be phased)
edam_operation: http://edamontology.org/operation_0527
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: PECAT
  role: Runs the phasing pipeline (pecat.pl unzip), bisecting k-means read grouping,
    and inconsistent overlap flagging/removal
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Maps corrected reads to first-round contigs and finds candidate overlaps ('-x
    map-pb -c -p 0.5 -r 1000'; '-x ava-pb')
  repo: https://github.com/lh3/minimap2
- name: Clair3
  role: Re-calls SNPs on raw Nanopore reads for the loose 6-allele inconsistent overlap
    threshold
  repo: https://github.com/HKU-BAL/Clair3
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:99ff53bd6e1f7cbad2541672614abf872285e6ac2e7d860700ddaa2d91f0173a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Clair3
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_0527
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# inconsistent-overlap-detection-string-graph

## Summary

Identifies and removes overlaps between reads from different haplotypes (inconsistent overlaps) in PECAT's second-round string graph assembly, using SNP allele vectors called from corrected/raw reads on the haplotype-collapsed first-round contigs.

## When to use

Use when reconstructing haplotype-specific contigs from long noisy reads: after a first-round haplotype-collapsed assembly, to phase reads by SNP alleles and flag cross-haplotype overlaps before reassembly.

## When NOT to use

- Input reads are PacBio HiFi or Nanopore duplex — the article reports PECAT achieves only mediocre performance on these accurate read types
- SNP sites lie in very low heterozygosity-rate regions, where SNP calling accuracy is too low to reliably distinguish haplotypes
- Dataset coverage is below the SNP-calling floor (sites outside coverage range [10, 1000] cannot be phased)

## Inputs

- Corrected long reads (PacBio CLR or Nanopore R9/R10) in FASTA/FASTQ
- First-round haplotype-collapsed contigs ($PROJECT/3-assemble/primary.fasta)
- PECAT config file with phase_rd2ctg_options, phase_phase_options, phase_filter_options (e.g. cfg_cattle_ont template)
- Optionally raw Nanopore reads for Clair3 SNP calling and loose-threshold flagging

## Outputs

- inconsistent_read_pairs.tsv listing flagged cross-haplotype read pairs for removal
- Second-round haplotype-specific contigs assembled from the simplified string graph
- Per-dataset inconsistent overlap percentages for round-1 vs round-2 simplified string graphs

## How to apply

Map corrected reads back to the first-round haplotype-collapsed contigs (PECAT's primary.fasta) with minimap2 using '-x map-pb -c -p 0.5 -r 1000' for PacBio CLR. Call heterozygous SNP sites from alignment base frequencies, keeping only sites with coverage in [10, 1000] whose second-most-common base counts pass r_l=0.4, r_h=0.2, c_l=10, c_h=100; for Nanopore raw reads, re-call SNPs with Clair3. Assign each read a SNP-allele vector and cluster reads with a modified bisecting k-means algorithm to verify and correct assigned alleles. A pair of reads with differing SNP alleles is inferred to come from different haplotypes: flag the overlap as inconsistent if the pair contains at least 8 different SNP alleles for corrected reads (strict) or at least 6 for raw reads (loose), additionally checking overlap direction/distance against the contigs (distance difference < 1000; raw-read distance threshold max(2500, 0.05*Dc)). Remove the flagged overlaps and run the second-round string graph assembly.

## Related tools

- **PECAT** (Runs the phasing pipeline (pecat.pl unzip), bisecting k-means read grouping, and inconsistent overlap flagging/removal) — https://github.com/lemene/PECAT
- **minimap2** (Maps corrected reads to first-round contigs and finds candidate overlaps ('-x map-pb -c -p 0.5 -r 1000'; '-x ava-pb')) — https://github.com/lh3/minimap2
- **Clair3** (Re-calls SNPs on raw Nanopore reads for the loose 6-allele inconsistent overlap threshold) — https://github.com/HKU-BAL/Clair3

## Examples

```
pecat.pl unzip cfg_cattle_ont
```

## Evaluation signals

- inconsistent_read_pairs.tsv exists, is parseable, and every flagged pair meets its allele threshold (>=8 different SNP alleles for corrected reads, >=6 for raw reads)
- Round-2 percentage of inconsistent overlaps in the simplified string graph is sharply lower than round-1 (article: e.g. 36.98% -> 0.32%, 16.00% -> 0.57%, 37.45% -> 1.09% across the seven diploid datasets)
- All called heterozygous SNP sites have coverage within [10, 1000]
- Flagged pairs pass the contig-consistency check: overlap direction/distance matches the contigs with distance difference < 1000 (raw reads: threshold max(2500, 0.05*Dc))

## Limitations

- SNP calling accuracy is low in low heterozygosity-rate regions, weakening inconsistent overlap identification there
- Corrected reads alone are insufficient for accurate SNP calling; raw Nanopore reads require Clair3 re-calling
- Loose (6-allele) raw-read threshold trades sensitivity for precision; PECAT needs higher data coverage for high-quality phased assembly

## Evidence

- [discussion] Strict vs loose allele thresholds and grouping: "using corrected reads with the strict threshold that the pair of reads should contain 8 different SNP alleles"
- [discussion] Loose raw-read threshold: "we use raw reads to identify inconsistent overlaps with the loose threshold that the pair of reads should contain 6 different SNP alleles"
- [discussion] Mapping and SNP calling procedure: [evidence span withheld — anchor status: not_found]
- [discussion] SNP site coverage filter: "(1) Its coverage is in the range [10, 1000]."
- [discussion] Read grouping by SNP alleles: "we use a modified bisecting k-means algorithm49 to divide the group"
- [discussion] Clair3 for Nanopore raw reads: "we use Clair3 to call SNPs again for Nanopore reads."
- [readme] Tool purpose from README: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
