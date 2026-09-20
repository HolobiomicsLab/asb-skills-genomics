---
name: polishing-tool-benchmarking
description: Use when you need to compare two or more genome polishing pipelines on
  HiFi (or T2T) assemblies and must quantify both correction gains and overcorrection
  harm, rather than relying on a single QV number.
when_to_use_negative:
- The assembly is from noisy long reads alone and no high-coverage short reads (>=60X)
  exist — NextPolish2-style k-mer benchmarking relies heavily on short reads to judge
  whether a K-mer contains errors
- The goal is de novo assembly quality assessment (completeness/contiguity, e.g. yak
  inspect for completeness) rather than comparing polishing-tool error correction
  and overcorrection behavior
- Only a single polishing pipeline is being applied with no comparison tool — benchmarking
  metrics such as fold-change ratios require at least two pipelines on identical inputs
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: NextPolish2
  role: Polishing tool under evaluation; produces polished assemblies (asm.np2.fa)
    from sorted BAM plus k21/k31 yak datasets
  repo: https://github.com/Nextomics/NextPolish2
- name: Merqury
  role: Computes QV of polished assemblies against short-read k-mer spectra to quantify
    consensus accuracy
- name: yak
  role: Builds k=21 and k=31 short-read k-mer datasets (yak count -b37 to discard
    singletons) used by NextPolish2 and QV evaluation
  repo: https://github.com/lh3/yak
- name: meryl
  role: Counts k=15 k-mers in assemblies to generate repetitive k-mer lists for winnowmap
    and to classify changed/overcorrected k-mers
- name: winnowmap
  role: Maps HiFi reads to repetitive assemblies using weighted minimizers and the
    repetitive_k15.txt list to build the sorted BAM input for polishing
  repo: https://github.com/marbl/Winnowmap
- name: minimap2
  role: Alternative HiFi mapper (-ax map-pb or map-hifi) and asm-to-ref aligner (-cx
    asm10 for TE identity, -cx map-hifi for pseudo-read evaluation)
  repo: https://github.com/lh3/minimap2/
- name: fastp
  role: Quality control and filtering of short reads (adapter removal, trimming, q20
    filtering) before k-mer dataset construction
provenance:
  source_task_ids:
  - task_008
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:2f607e729bece61fdd4ba40d2b6a5969e8ae64baa157f2eb736b7c1ab186d3f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nextomics/NextPolish2
  tools:
  - Merqury
  - NextPolish2
  - RepeatMasker
  - fastp
  - meryl
  - minimap2
  - samtools
  - winnowmap
  - yak
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# polishing-tool-benchmarking

## Summary

A multi-pronged benchmarking strategy for comparing genome polishing tools (e.g. NextPolish2 vs Racon+Merfin pipelines) on HiFi-assembled genomes, using Merqury/k-mer QV, k-mer overcorrection counts, TE identity ratios, and pseudo-long-read alignment-based accuracy as complementary evaluation axes.

## When to use

Use when you need to compare two or more genome polishing pipelines on HiFi (or T2T) assemblies and must quantify both correction gains and overcorrection harm, rather than relying on a single QV number.

## When NOT to use

- The assembly is from noisy long reads alone and no high-coverage short reads (>=60X) exist — NextPolish2-style k-mer benchmarking relies heavily on short reads to judge whether a K-mer contains errors
- The goal is de novo assembly quality assessment (completeness/contiguity, e.g. yak inspect for completeness) rather than comparing polishing-tool error correction and overcorrection behavior
- Only a single polishing pipeline is being applied with no comparison tool — benchmarking metrics such as fold-change ratios require at least two pipelines on identical inputs

## Inputs

- Unpolished HiFi hifiasm assembly FASTA (e.g. A. thaliana Col-XJTU primary contigs, HG002/HG005 primary assembly)
- PacBio HiFi reads FASTQ/FASTA for mapping (winnowmap/minimap2 -ax map-pb)
- High-coverage short reads (>=60X recommended) for yak k=21 and k=31 datasets, fastp-filtered
- meryl k=15 repetitive k-mer list from the assembly (greater-than distinct=0.9998) for winnowmap -W
- Ground-truth reference FASTA (e.g. CHM13 T2T) split into ~6250 pseudo-long reads of 500 kb
- Reference TE sequences (e.g. 158 Col-CEN centromeric TEs from TAIR)

## Outputs

- Per-assembly QV table across polishing methods and short-read coverage levels (Merqury)
- overcorrection_kmer_counts.csv with changed K-mers, potential overcorrection K-mers, and RM/NextPolish2 fold-change ratios
- te_identity_summary.csv with per-TE alignment identity ratios for each polishing method
- qv_report.tsv with per-assembly alignment-identity QV and fraction of 100%-identity pseudo-reads
- Stratification of erroneous k-mers by HiFi mapping depth and spanning-read identity (fraction at depth <=3, mean identity per genome)

## How to apply

Assemble the evaluation around four axes: (1) QV via Merqury on short-read yak k-mer datasets (k=21/31 built with 'yak count -b37' to discard singletons) to capture overall consensus accuracy before/after polishing; (2) k-mer change accounting — count k-mers (meryl count k=15) in unpolished and polished assemblies, classify 'changed K-mers' and 'potential overcorrection K-mers' against the short-read spectrum, and report RM/NextPolish2 fold-change ratios; (3) targeted repeat fidelity — polish assembled Col-CEN transposable elements, align polished vs reference TE sequences with minimap2 -cx asm10, and compute per-TE identity fractions (e.g. NextPolish2 raising 100%-identity TEs from 68.46% to 91.95% while RM dropped to 12.75%); (4) reference-grounded pseudo-read evaluation — split a trusted reference (e.g. CHM13 T2T) into ~500 kb pseudo-long reads, align to each assembly with minimap2 -cx map-hifi, filter alignments at query coverage >90% and identity >98%, and tabulate QV and the 100%-identity read fraction. Decision point: accept a polishing tool only when it raises QV/identity without inflating overcorrection k-mers or lowering identity for a large fraction of targets.

## Related tools

- **NextPolish2** (Polishing tool under evaluation; produces polished assemblies (asm.np2.fa) from sorted BAM plus k21/k31 yak datasets) — https://github.com/Nextomics/NextPolish2
- **Merqury** (Computes QV of polished assemblies against short-read k-mer spectra to quantify consensus accuracy)
- **yak** (Builds k=21 and k=31 short-read k-mer datasets (yak count -b37 to discard singletons) used by NextPolish2 and QV evaluation) — https://github.com/lh3/yak
- **meryl** (Counts k=15 k-mers in assemblies to generate repetitive k-mer lists for winnowmap and to classify changed/overcorrected k-mers)
- **winnowmap** (Maps HiFi reads to repetitive assemblies using weighted minimizers and the repetitive_k15.txt list to build the sorted BAM input for polishing) — https://github.com/marbl/Winnowmap
- **minimap2** (Alternative HiFi mapper (-ax map-pb or map-hifi) and asm-to-ref aligner (-cx asm10 for TE identity, -cx map-hifi for pseudo-read evaluation)) — https://github.com/lh3/minimap2/
- **fastp** (Quality control and filtering of short reads (adapter removal, trimming, q20 filtering) before k-mer dataset construction)

## Examples

```
./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa
```

## Evaluation signals

- QV-vs-coverage table must show the identical assembly gaining QV as short-read coverage increases (e.g. HG005 QV rising from 53.8544 to 55.6257 after NextPolish2 polishing)
- RM/NextPolish2 fold-change ratios for changed K-mers and potential overcorrection K-mers must fall within reported ranges (~5.31-547.64x and ~4.98-63.33x) with no significant QV improvement for RM
- TE identity summary must reproduce NextPolish2 raising 100%-identity TEs from 68.46% to 91.95% with no overcorrection, versus RM at 12.75% and ~80.54% of TEs losing identity
- Pseudo-read evaluation (5868 retained alignments, ~2.93 Gb after coverage >90% and identity >98% filters) must show NextPolish2 raising the 100%-identity fraction from 9.48% to 9.58% with only 14% of reads decreasing in identity
- Error k-mer stratification must recover paper-reported low-depth fractions (Col-XJTU 34.52% at avg depth 104.82; HG002 76.29% at avg depth 32.75) and mean spanning-read identities (98.31% and 95.37%)

## Limitations

- NextPolish2 performance relies heavily on short reads to check whether a K-mer contains errors; high-coverage short reads are needed to minimize uncorrected errors caused by short-read sequencing biases, and no specific coverage threshold is quantified beyond the >=60X recommendation
- Erroneous k-mers reported by Merqury in polished assemblies cluster disproportionately in regions with very low short-read mapping depth (e.g. 34.52% at depth <=3 despite average input depth of 104.82), so QV alone conflates insufficient-coverage regions with true errors
- No version numbers stated for NextPolish2 or its dependencies, complicating exact reproducibility of benchmark numbers

## Evidence

- [abstract] NextPolish2 can fix base errors in HiFi long-read assemblies without introducing excessive overcorrections and haplotype switch errors: "which can fix base errors remaining in those “highly accurate” genomes assembled from HiFi long reads without introducing excessive overcorrections and haplotype switch errors"
- [methods] winnowmap mapping with repetitive k-mers: "meryl count k=15 output merylDB asm.fa.gz
meryl print greater-than distinct=0.9998 merylDB > repetitive_k15.txt"
- [methods] NextPolish2 invocation: "./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa"
- [methods] HG005 QV improvement: "`NextPolish2` improved the QV of the assembled genome from 53.8544 to 55.6257 and reduced the number of homozygous high-quality variants (potential errors) from 14,155 to 6,955."
- [discussion] short-read dependence: "The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors."
- [readme] yak singleton removal: "# build k-mer hash tables for high-coverage reads; discard singletons
./yak count -b37 -t32 -o ccs.yak ccs-reads.fq.gz"
