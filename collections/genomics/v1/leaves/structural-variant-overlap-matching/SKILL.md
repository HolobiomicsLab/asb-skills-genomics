---
name: structural-variant-overlap-matching
description: Use when validating a genome-assembly error caller against known simulated
  error loci (e.g., SVs, indels, misjoins, inversions) and computing per-class recall,
  precision, and F1 with explicit overlap rules.
when_to_use_negative:
- Evaluating a real (non-simulated) assembly where no ground-truth error loci exist
  — the overlap match cannot establish precision without true positives defined in
  advance
- Assessing assembly completeness (e.g., BUSCO) or quality indices (AQI) rather than
  error-detection accuracy benchmarking
- Polyploid genome assemblies, where the article notes CRAQ accuracy 'may not be as
  accurate as the benchmarked cases presented here'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0102
tools:
- name: CRAQ
  role: Reference-free error caller producing CREs and CSEs whose breakpoints are
    overlap-matched to simulated error loci
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps reads to the draft assembly ('-ax sr' for short reads, '-ax map-hifi'
    for long reads) before CRAQ error calling
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments with '-F 1796 -q 20' to retain only high-quality primary
    alignments used for error calling
- name: QUAST
  role: Baseline evaluator run against the true hg38 reference; its error calls are
    overlap-matched to simulated loci
- name: Inspector
  role: Baseline evaluator using simulated long reads; its reported errors are matched
    to simulated error loci
- name: Merqury
  role: k-mer based baseline (k=21) on Illumina reads; errors validated via 21-bp
    flanking region overlap
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:e08cc45b48869f23b8c16d9f7a8babc774772af6f01e2aca050538cf92723341
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Inspector
  - Merqury
  - Minimap2
  - PBSIM
  - QUAST
  - SAMtools
  - Wgsim
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# structural-variant-overlap-matching

## Summary

Benchmark an assembly evaluator by matching its reported error loci to a set of ground-truth simulated error breakpoints and computing recall, precision, and F1 per error class, as done for CRAQ against QUAST-LG, Inspector, and Merqury on simulated hg38 assemblies.

## When to use

Use when validating a genome-assembly error caller against known simulated error loci (e.g., SVs, indels, misjoins, inversions) and computing per-class recall, precision, and F1 with explicit overlap rules.

## When NOT to use

- Evaluating a real (non-simulated) assembly where no ground-truth error loci exist — the overlap match cannot establish precision without true positives defined in advance
- Assessing assembly completeness (e.g., BUSCO) or quality indices (AQI) rather than error-detection accuracy benchmarking
- Polyploid genome assemblies, where the article notes CRAQ accuracy 'may not be as accurate as the benchmarked cases presented here'

## Inputs

- Ground-truth simulated error loci with breakpoints and boundaries (e.g., hg38_sim2 regional/structural errors and repeat errors)
- Evaluator error call sets (CRAQ CRE/CSE coordinates, QUAST-LG, Inspector, Merqury output)
- Reference assembly (hg38, GCF_000001405.26) and simulated assembly (hg38_sim2)
- Mapped alignments (SAM/BAM) from Minimap2 with '-ax sr' or '-ax map-hifi' presets

## Outputs

- Per-error-class recall, precision, and F1 table (e.g., Table 1: CRAQ vs QUAST-LG, Inspector, Merqury)
- Matched/overlapping error locus lists per evaluator with overlap classification (breakpoint-in-boundary, 50 bp adjacent, or k-mer flank overlap)

## How to apply

Generate a ground-truth set of simulated error loci with known boundaries (e.g., 7,000 regional/structural errors in hg38_sim2: 6,000 regional indels, 200 fragment indels, 400 contig misjoins, 400 inversions, plus 1,200 repeat errors from the RepeatBrowser2020 hg38 repeat database). Run each evaluator on the same inputs (CRAQ on Minimap2/SAMtools-filtered alignments; QUAST-LG against the true hg38 reference; Inspector with simulated long reads; Merqury with k-mer 21 on Illumina reads), then apply evaluator-specific matching rules: a CRAQ CRE/CSE counts as overlapping if its breakpoint falls within the SV boundary or within 50 bp adjacent, while a Merqury error is validated if the boundary or the 21-bp flanking region overlaps the simulated locus. Tabulate true positives, false positives, and false negatives per error class and compute recall, precision, and F1. Validate the benchmark against published values — CRAQ CRE F1 97.463%, CSE F1 97.067%, Inspector CRE F1 ~96% (CSE recall 28%), Merqury F1 87.7% — allowing a small tolerance of about +/-2 percentage points.

## Related tools

- **CRAQ** (Reference-free error caller producing CREs and CSEs whose breakpoints are overlap-matched to simulated error loci) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps reads to the draft assembly ('-ax sr' for short reads, '-ax map-hifi' for long reads) before CRAQ error calling) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments with '-F 1796 -q 20' to retain only high-quality primary alignments used for error calling)
- **QUAST** (Baseline evaluator run against the true hg38 reference; its error calls are overlap-matched to simulated loci)
- **Inspector** (Baseline evaluator using simulated long reads; its reported errors are matched to simulated error loci)
- **Merqury** (k-mer based baseline (k=21) on Illumina reads; errors validated via 21-bp flanking region overlap)

## Examples

```
samtools view -F 1796 -q 20 aln.bam | craq -q hg38_sim2.fa && python match_errors.py --calls craq_CRE.bed --truth hg38_sim2_loci.bed --flank 50
```

## Evaluation signals

- CRAQ CRE F1 reproduces at 97.463% and CSE F1 at 97.067% within +/-2 percentage points
- Inspector CRE F1 is approximately 96% with CSE recall of 28%
- Merqury F1 is approximately 87.7%
- Every reported error is classified as TP, FP, or FN against the simulated locus list under the evaluator-specific overlap rule (SV boundary + 50 bp for CRAQ; boundary or 21-bp flank for Merqury)
- Recall, precision, and F1 are computed per error class (regional indels, fragment indels, misjoins, inversions, repeat errors), not just aggregated

## Limitations

- Merqury error matching depends on k-mer size (k=21); changing k changes the flanking-region overlap rule and results are not directly comparable
- CRAQ accuracy is not established for polyploid genomes and may be lower than the benchmarked diploid cases
- Error breakpoint ratios can fall below 100% in practice due to sequencing errors or inaccurate read mapping, complicating exact matching
- Overlap rules are evaluator-specific; a naive uniform interval-overlap criterion will not reproduce the published Table 1

## Evidence

- [other] CRAQ CRE/CSE counted as overlapping if the breakpoint falls within the SV boundary or 50 bp adjacent: [evidence span withheld — anchor status: not_found]
- [other] F1 scores above 97%: [evidence span withheld — anchor status: not_found]
- [discussion] Filtering alignments: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’35"
- [abstract] Mapping presets: "By default, Minimap2 ‘–ax sr’ and  ‘–ax map-hifi’(‘map-hifi’ for PacBio HiFi,‘map-pb’ for PacBio CLR, ‘map-ont’ for ONT library) options were used"
- [discussion] Simulated errors in hg38: "we simulated structural and small-scale local assembly errors, as well as heterozygous regions, in the human reference genome hg38"
- [discussion] Polyploid limitation: "evaluation of genome assemblies for species with higher ploidy levels may not be as accurate as the benchmarked cases presented here"
