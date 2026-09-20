---
name: nanopore-cpg-methylation-calling
description: Use when nanopore FAST5/FASTQ data needs per-read CpG (5mC) methylation
  calls, especially alongside 6mA detection, or when comparing nanoHiMe vs nanopolish
  vs Megalodon accuracy.
when_to_use_negative:
- When the goal is calling 6mA-containing regions genome-wide — use nanoHiMe 6mA mode
  instead, since genome-wide mCG-style calls will not localize modified-adenine windows
- When reads are not aligned to k-mer-level events (no nanopolish eventalign file)
  — nanoHiMe requires event-level alignments, not just FASTQ
- When DNA carries modifications outside the trained model space (e.g. 5-hmC, 5-fC,
  4-mC, GpC methylation) — no unified calling model exists yet
edam_operation: http://edamontology.org/operation_3186
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: nanoHiMe
  role: HMM-based CpG methylation caller run in mCG mode on eventalign files, classifying
    CpGs by log likelihood ratio thresholds (+1.5 / -1.5)
  repo: github.com/YinLabTJ/nanoHiMe
- name: nanopolish
  role: v0.13.2 used for read indexing, event-to-k-mer alignment (--scale-events)
    and as an independent 5mC-in-CpG caller for benchmarking
- name: Megalodon
  role: v2.5.0 with model res_dna_r941_min_modbases-all-context_v001.cfg, an ONT baseline
    5mC caller for benchmarking
- name: Minimap2
  role: v2.17 aligns basecalled ONT reads to hg19 in map-ont mode with 8 threads before
    event alignment
- name: Samtools
  role: v1.9 sorts/indexes BAMs and retains mapped MAPQ>=20 reads via 'samtools view
    -b -q 20 -F 4'
- name: Guppy
  role: v4.4.2 high-accuracy basecaller converting FAST5 signals to FASTQ
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:57b7ad83c345660d73b7302e956a10bd1e9960d7a3d07624071e451db2e13320
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - Megalodon
  - Minimap2
  - Samtools
  - nanoHiMe
  - nanopolish
  - samtools
  edam_operation: http://edamontology.org/operation_3186
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# nanopore-cpg-methylation-calling

## Summary

Call CpG methylation from nanopore reads using nanoHiMe's hidden Markov model on nanopolish eventalign files, benchmarking against nanopolish and Megalodon with AUCs against enzymatic-treatment ground truth and Pearson correlations with WGBS.

## When to use

Use when nanopore FAST5/FASTQ data needs per-read CpG (5mC) methylation calls, especially alongside 6mA detection, or when comparing nanoHiMe vs nanopolish vs Megalodon accuracy.

## When NOT to use

- When the goal is calling 6mA-containing regions genome-wide — use nanoHiMe 6mA mode instead, since genome-wide mCG-style calls will not localize modified-adenine windows
- When reads are not aligned to k-mer-level events (no nanopolish eventalign file) — nanoHiMe requires event-level alignments, not just FASTQ
- When DNA carries modifications outside the trained model space (e.g. 5-hmC, 5-fC, 4-mC, GpC methylation) — no unified calling model exists yet

## Inputs

- nanopore FAST5 signal files plus basecalled FASTQ (e.g. Guppy v4.4.2 high accuracy output)
- sorted, MAPQ>=20-filtered BAM aligned with minimap2 map-ont to hg19 or E. coli K12 MG1655 reference
- nanopolish eventalign output file (test.eventalign.txt, plain or gzip)
- upper-cased, samtools-faidx-indexed reference FASTA (ref_upper.fa)
- ground-truth methylation annotations (M.SssI/Hia5 enzymatic treatments; ENCODE WGBS data ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ, ENCFF369YQW)

## Outputs

- output.mCG.eventalign.txt with per-read CpG log likelihood ratios (CHR, Chain, Start, End, Read_ID, Log(LIKELIHOOD_RATIO)_mCG, Ref_sequence)
- per-tool, per-condition AUC comparison table (auc_comparison_table.csv)
- pairwise per-CpG methylation correlation matrix vs WGBS and CUT&Tag-BS (methylation_correlation_matrix.csv)

## How to apply

Basecall FAST5 with Guppy v4.4.2 (high accuracy model), index reads with 'nanopolish index -d fast5', align to the reference with minimap2 map-ont, filter with 'samtools view -b -q 20 -F 4' (MAPQ >=20), and produce event-level alignments with 'nanopolish eventalign -n -t 16 --scale-events'. Convert the reference to upper case ('perl upper.pl ref.fa > ref_upper.fa'), faidx-index it, and run nanoHiMe mCG mode, which outputs per-read-region log likelihood ratios where a threshold of 1.5 calls methylated CpGs and -1.5 calls unmethylated CpGs. In parallel, call 5mC with Megalodon v2.5.0 (model res_dna_r941_min_modbases-all-context_v001.cfg) and nanopolish v0.13.2 on the same data. Benchmark by computing ROC/AUC against enzymatic ground truth (M.SssI-methylated CpGs on E. coli K12 MG1655 PCR data, with/without Hia5-introduced 6mA) or by Pearson correlation of per-CpG fractions with WGBS at sites covered >=20 by both WGBS and nanoHiMe-seq.

## Related tools

- **nanoHiMe** (HMM-based CpG methylation caller run in mCG mode on eventalign files, classifying CpGs by log likelihood ratio thresholds (+1.5 / -1.5)) — github.com/YinLabTJ/nanoHiMe
- **nanopolish** (v0.13.2 used for read indexing, event-to-k-mer alignment (--scale-events) and as an independent 5mC-in-CpG caller for benchmarking)
- **Megalodon** (v2.5.0 with model res_dna_r941_min_modbases-all-context_v001.cfg, an ONT baseline 5mC caller for benchmarking)
- **Minimap2** (v2.17 aligns basecalled ONT reads to hg19 in map-ont mode with 8 threads before event alignment)
- **Samtools** (v1.9 sorts/indexes BAMs and retains mapped MAPQ>=20 reads via 'samtools view -b -q 20 -F 4')
- **Guppy** (v4.4.2 high-accuracy basecaller converting FAST5 signals to FASTQ)

## Examples

```
./nanoHiMe mCG ref_upper.fa test.eventalign.txt.gz output.mCG
```

## Evaluation signals

- On E. coli sites containing 6mA, nanoHiMe mCG AUC should reproduce ~0.922 (vs nanopolish 0.919, Megalodon 0.603) within tolerance; on 6mA-free sites ~0.953/0.951/0.934
- nanoHiMe vs WGBS per-CpG Pearson correlation should approach r=0.931, exceeding Megalodon and nanopolish consistency (nanoHiMe vs nanopolish r=0.953, Megalodon vs nanopolish r=0.885)
- Output eventalign table must contain positive Log(LIKELIHOOD_RATIO)_mCG values supporting methylation and negative values supporting unmethylation
- Correlation analysis restricted to CpG sites with coverage depth >=20 by both WGBS and nanoHiMe-seq data
- auc_comparison_table.csv must exist in CSV format with rows for all three tools under both 6mA-absent and 6mA-present conditions

## Limitations

- Megalodon could not accurately identify methylated CpG sites, particularly on DNA molecules that also carry 6mA, where its AUC dropped to ~0.603
- Haplotype phasing of methylation calls depends on the density of heterozygous SNVs/INDELs at the locus
- A unified calling model covering all combinations of introduced modifications (5-hmC, 5-fC, 4-mC, GpC methylation) does not yet exist and is deferred to future work

## Evidence

- [readme] mCG-calling thresholds: "A positive value in Log(LIKELIHOOD_RATIO)_mCG shows the support for mCG methylation, and a negative value indicates the support for unmethylation."
- [readme] mCG command line: "./nanoHiMe mCG ref_upper.fa test.eventalign.txt(.gz) output.mCG"
- [methods] event align step: "The alignment of the electric current events to k-mers of a known substring from the reference genome was obtained using nanopolish v0.13.2."
- [discussion] calling thresholds: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs"
- [discussion] coverage filter: "calculated for the CpG sites with coverage depth ≥20 by both WGBS and nanoHiMe-seq data"
- [discussion] Megalodon benchmark finding: "Megalodon couldn’t accurately identify methylated CpG sites, and its accuracy for predicting 6mA-containing sites was also significantly lower than the accuracy of nanoHiMe"
