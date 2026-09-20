---
name: precision-recall-f1-evaluation
description: Use when an evaluator's reported error coordinates must be scored against
  simulated or known ground-truth error loci to produce per-error-class recall, precision,
  and F1 values.
when_to_use_negative:
- No ground-truth error loci (simulated or independently validated) exist for the
  assembly being assessed — use AQI-style metrics instead
- Assessing only completeness rather than error detection (e.g. BUSCO-style single-copy
  ortholog benchmarking)
- Evaluating assemblies of species with higher ploidy levels, where the paper notes
  accuracy may be lower than the benchmarked diploid cases
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: CRAQ
  role: Reference-free evaluator whose CRE/CSE breakpoint calls are scored against
    simulated loci; overlaps counted if the breakpoint falls within the SV boundary
    or 50 bp adjacent
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps raw reads to the draft assembly ('-ax sr' for short reads, '-ax map-hifi'
    for HiFi long reads) to produce the BAM inputs CRAQ is scored on
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments ('-F 1796 -q 20') before error calling so precision/recall
    reflect high-confidence mappings
- name: QUAST-LG
  role: Reference-based baseline evaluator scored against the true hg38 reference
    in the same recall/precision/F1 table
- name: Inspector
  role: Baseline evaluator run with the simulated long reads; benchmarked for CRE
    F1 (~96%) and CSE recall (28%)
- name: Merqury
  role: k-mer-based baseline (k = 21) run on Illumina reads; errors validated when
    the boundary or 21-bp flanking region overlaps the simulated locus
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:4229cbb17eeb8f17bb3e9d4fa9fc936b6b2a45d8ae0aa9e4b647a465cc920f71
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
  - QUAST-LG
  - SAMtools
  - Wgsim
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# precision-recall-f1-evaluation

## Summary

Benchmark an assembly-evaluator's error calls (e.g. CRAQ CREs/CSEs, QUAST-LG, Inspector, Merqury) against a simulated ground-truth error set by matching reported error loci to simulated loci and computing per-class recall, precision, and F1, as in the CRAQ paper's Table 1 comparison.

## When to use

Use when an evaluator's reported error coordinates must be scored against simulated or known ground-truth error loci to produce per-error-class recall, precision, and F1 values.

## When NOT to use

- No ground-truth error loci (simulated or independently validated) exist for the assembly being assessed — use AQI-style metrics instead
- Assessing only completeness rather than error detection (e.g. BUSCO-style single-copy ortholog benchmarking)
- Evaluating assemblies of species with higher ploidy levels, where the paper notes accuracy may be lower than the benchmarked diploid cases

## Inputs

- Simulated error assembly (e.g. hg38_sim2 FASTA with embedded regional/structural/repeat errors) and the true reference hg38 (GCF_000001405.26)
- Simulated reads: PBSIM HiFi-like FASTQ (--depth 40, length-mean 10000, accuracy-min 0.95) and Wgsim Illumina-like FASTQ (150 bp PE, -e 0.0001)
- BAM alignments filtered with 'samtools view -F 1796 -q 20'
- Reported error coordinate sets from each evaluator (CRAQ CRE/CSE BED-like coordinates, QUAST-LG, Inspector, Merqury k=21 reports)
- Ground-truth simulated error locus list from RepeatBrowser2020 hg38 repeat database and embedded SVs

## Outputs

- Per-error-class recall, precision, and F1 table (Table 1 style: CRAQ vs QUAST-LG vs Inspector vs Merqury)
- Counts of true-positive, false-positive, and false-negative error calls per evaluator
- Matched-overlap records linking each validated call to its simulated error locus

## How to apply

Build a ground truth by embedding known errors in a reference (e.g. hg38_sim2 with 6,000 regional indels, 200 fragment indels, 400 contig misjoins, 400 inversions, and repeat errors drawn from the RepeatBrowser2020 hg38 database) and simulate reads (PBSIM --depth 40 --accuracy-min 0.95; Wgsim -e 0.0001, 150 bp PE). Run each evaluator on identical inputs — CRAQ on alignments mapped with Minimap2 ('-ax sr' for short reads, '-ax map-hifi' for long reads) and filtered with 'samtools view -F 1796 -q 20'; QUAST-LG against the true hg38; Inspector with the long reads; Merqury with k = 21 on the Illumina reads. Define an explicit overlap rule per evaluator: a CRAQ CRE/CSE overlaps a simulated locus if its breakpoint falls within the SV boundary or 50 bp adjacent, while a Merqury error is validated if the boundary or the 21-bp (k-mer-size) flanking region overlaps the simulated locus. From the matched/total counts compute recall = TP/(TP+FN), precision = TP/(TP+FP), and F1 per error class (CRE, CSE, regional, structural, repeat). Judge success by agreement with published benchmarks within a tolerance (e.g. CRAQ CRE F1 97.463% and CSE F1 97.067%, Inspector CRE F1 ~96% with CSE recall 28%, Merqury F1 87.7%; accept ±2 percentage points).

## Related tools

- **CRAQ** (Reference-free evaluator whose CRE/CSE breakpoint calls are scored against simulated loci; overlaps counted if the breakpoint falls within the SV boundary or 50 bp adjacent) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps raw reads to the draft assembly ('-ax sr' for short reads, '-ax map-hifi' for HiFi long reads) to produce the BAM inputs CRAQ is scored on) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments ('-F 1796 -q 20') before error calling so precision/recall reflect high-confidence mappings)
- **QUAST-LG** (Reference-based baseline evaluator scored against the true hg38 reference in the same recall/precision/F1 table)
- **Inspector** (Baseline evaluator run with the simulated long reads; benchmarked for CRE F1 (~96%) and CSE recall (28%))
- **Merqury** (k-mer-based baseline (k = 21) run on Illumina reads; errors validated when the boundary or 21-bp flanking region overlaps the simulated locus)

## Examples

```
minimap2 -ax sr hg38_sim2.fa illumina_1.fq illumina_2.fq | samtools view -F 1796 -q 20 -b -o filtered.bam && craq -i filtered.bam -o craq_out
```

## Evaluation signals

- CRAQ CRE F1 ≈ 97.463% and CSE F1 ≈ 97.067% on the simulated hg38 dataset (accept within ±2 percentage points of the reported F1 > 97%)
- Inspector shows CRE F1 ~96% but markedly lower CSE recall (~28%); Merqury F1 ≈ 87.7% — cross-check the ordering CRAQ > Inspector > Merqury
- Every counted true positive must satisfy the pre-declared overlap rule (breakpoint within SV boundary or ±50 bp for CRAQ; boundary or 21-bp flanking region for Merqury)
- Filtered BAM must contain no low-MAPQ (<20), unmapped, secondary, QC-failed, or PCR-duplicated alignments before CRAQ calling
- Recall + FN accounting must reconcile: TP + FN equals the total number of simulated errors of that class

## Limitations

- Key benchmarking results (Table 1) and data sources are deferred to supplementary materials (Table 1, Supplementary Fig. 13, Supplementary Data 1), so exact per-class counts may not be reproducible from the main text
- Evaluation of assemblies for species with higher ploidy levels may not be as accurate as the benchmarked cases presented
- Error ratios can fall below 100% in practice due to sequencing errors or inaccurate read mapping, complicating the ground-truth matching

## Evidence

- [discussion] samtools filter rationale: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’35"
- [discussion] error classes evaluated: "Assembly errors were classified as CREs or CSEs."
- [discussion] benchmark framing: "demonstrating the accuracy of this tool and the importance of removing heterozygous loci during assembly assessment (Table 1, Supplementary Data 1)"
- [readme] CRAQ output scored in the benchmark: "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs)"
