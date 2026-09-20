---
name: cpg-methylation-likelihood-ratio-calling
description: Use when you have nanopolish eventalign output from MinION nanoHiMe-seq
  or plain nanopore reads and need per-read/per-site CpG methylation calls (mCG) robust
  at 10×–30× coverage without bisulfite conversion.
when_to_use_negative:
- Input is already bisulfite-converted (WGBS/BiSulfate) data — nanoHiMe calls methylation
  from nanopore signal events, not bisulfite read counts.
- You need joint detection of all modified-base combinations (5-hmC, 5-fC, 4-mC, GpC
  methylation) — no unified calling model for these exists yet per the authors.
- You only need 6mA-containing region calls — use nanoHiMe's '6mA' mode with a peak
  BED instead, since whole-genome mCG-style calling is slow and answers a different
  question.
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3674
tools:
- name: nanoHiMe
  role: HMM-based methylation caller; run in mCG-calling mode to emit per-CpG log-likelihood
    ratios with +1.5/−1.5 thresholds
  repo: https://github.com/YinLabTJ/nanoHiMe
- name: nanopolish
  role: Aligns nanopore electric-current events to reference k-mers (--scale-events)
    to produce the eventalign input for nanoHiMe
- name: minimap2
  role: Aligns ONT basecalled reads to hg19 with the map-ont preset before event alignment
- name: samtools
  role: Sorts/indexes BAMs, filters mapped reads with MAPQ>=20 (view -b -q 20 -F 4),
    and faidx-indexes the uppercase reference
- name: Guppy
  role: Basecalls FAST5 to FASTQ using the high-accuracy model (v4.4.2) prior to alignment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:8515a25c7a18fd6252b591f23d98bfbef7bdf299ba9ef545b5da74cf9a5a1f7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YinLabTJ/nanoHiMe
  tools:
  - Guppy
  - minimap2
  - nanoHiMe
  - nanopolish
  - samtools
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cpg-methylation-likelihood-ratio-calling

## Summary

Call per-CpG methylation states from nanopore signal alignments using nanoHiMe's hidden Markov model in mCG-calling mode, classifying each CpG by log-likelihood-ratio thresholds (+1.5 methylated, −1.5 unmethylated) to enable low-coverage CpG methylation profiling comparable to WGBS.

## When to use

Use when you have nanopolish eventalign output from MinION nanoHiMe-seq or plain nanopore reads and need per-read/per-site CpG methylation calls (mCG) robust at 10×–30× coverage without bisulfite conversion.

## When NOT to use

- Input is already bisulfite-converted (WGBS/BiSulfate) data — nanoHiMe calls methylation from nanopore signal events, not bisulfite read counts.
- You need joint detection of all modified-base combinations (5-hmC, 5-fC, 4-mC, GpC methylation) — no unified calling model for these exists yet per the authors.
- You only need 6mA-containing region calls — use nanoHiMe's '6mA' mode with a peak BED instead, since whole-genome mCG-style calling is slow and answers a different question.

## Inputs

- nanopolish eventalign output (test.eventalign.txt or gzipped) from nanopore reads aligned to hg19
- Uppercase, samtools-faidx-indexed reference FASTA (e.g. hg19.fa converted with upper.pl)
- Basecalled FASTQ + FAST5 signal files, MAPQ>=20-filtered sorted BAM

## Outputs

- output.mCG.eventalign.txt: per-read, per-window CpG rows with CHR, Chain, Start, End, Read_ID, Log(LIKELIHOOD_RATIO)_mCG, Ref_sequence
- Per-CpG methylation levels (after LLR thresholding at +1.5/−1.5) usable for correlation vs WGBS

## How to apply

Basecall FAST5 with Guppy v4.4.2 (high accuracy model), index with nanopolish, align to hg19 with minimap2 -ax map-ont, and retain mapped reads with MAPQ >= 20 (samtools view -b -q 20 -F 4). Run nanopolish eventalign -n -t 16 --scale-events to align signal events to reference k-mers, then convert the reference FASTA to uppercase with nanoHiMe's upper.pl and index it (samtools faidx), since nanoHiMe requires an uppercase, indexed reference. Run nanoHiMe in mCG-calling mode: ./nanoHiMe mCG ref_upper.fa eventalign.txt(.gz) output.mCG — the HMM (trained via Gaussian-mixture EM and 5 Viterbi realignment iterations) emits a Log(LIKELIHOOD_RATIO)_mCG per CpG-containing window; classify sites as methylated if LLR > +1.5 and unmethylated if LLR < −1.5. For WGBS-comparability analyses, restrict to CpG sites with coverage depth >= 20 by both WGBS and nanoHiMe-seq and compute Pearson correlations of methylation levels.

## Related tools

- **nanoHiMe** (HMM-based methylation caller; run in mCG-calling mode to emit per-CpG log-likelihood ratios with +1.5/−1.5 thresholds) — https://github.com/YinLabTJ/nanoHiMe
- **nanopolish** (Aligns nanopore electric-current events to reference k-mers (--scale-events) to produce the eventalign input for nanoHiMe)
- **minimap2** (Aligns ONT basecalled reads to hg19 with the map-ont preset before event alignment)
- **samtools** (Sorts/indexes BAMs, filters mapped reads with MAPQ>=20 (view -b -q 20 -F 4), and faidx-indexes the uppercase reference)
- **Guppy** (Basecalls FAST5 to FASTQ using the high-accuracy model (v4.4.2) prior to alignment)

## Examples

```
perl nanoHiMe/perl_script/upper.pl hg19.fa > hg19_upper.fa; samtools faidx hg19_upper.fa; ./nanoHiMe mCG hg19_upper.fa test.eventalign.txt.gz output.mCG
```

## Evaluation signals

- Every CpG row in output.mCG.eventalign.txt carries a numeric Log(LIKELIHOOD_RATIO)_mCG; positive values support methylation, negative values support unmethylation (sanity-check sign balance).
- Correlation gate: nanoHiMe CpG methylation vs ENCODE WGBS (ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ, ENCFF369YQW) should reach r≈0.82 at 10× coverage and ≈0.92 at 30× coverage when restricted to CpGs with depth >= 20 in both datasets; 10× vs 50× self-consistency r≈0.914 (recorded in coverage_correlation.csv).
- Input reference check: nanoHiMe requires the FASTA converted to uppercase (upper.pl) and faidx-indexed; calling on a lower-case reference is invalid.
- Alignment filter check: only reads with MAPQ >= 20 and mapped flag (samtools view -b -q 20 -F 4) should feed eventalign, matching the paper's model-training read restriction.

## Limitations

- CpG methylation calling accuracy depends on coverage; correlations with WGBS plateau only near 30× (r≈0.92), and low depths still need one or two MinION flowcells.
- Megalodon (v2.5.0) could not accurately identify methylated CpG sites in this comparison, so ONT-provided tools are not an adequate substitute for nanoHiMe's HMM calls.
- Haplotype-level phasing of CpG methylation states depends on the density of heterozygous SNVs/INDELs and degrades at loci with low variant density.
- Assignment of trained EM parameters to 6-mers with methyladenines at different positions is imprecise, so such k-mers are collapsed to an averaged k-mer, a reproducibility caveat for the underlying HMM.

## Evidence

- [discussion] we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs"
- [discussion] calculated for the CpG sites with coverage depth ≥20 by both WGBS and nanoHiMe-seq data: "calculated for the CpG sites with coverage depth ≥20 by both WGBS and nanoHiMe-seq data"
- [readme] nanoHiMe mCG ref_upper.fa test.eventalign.txt(.gz) output.mCG: "./nanoHiMe mCG ref_upper.fa test.eventalign.txt(.gz) output.mCG"
- [readme] A positive value in Log(LIKELIHOOD_RATIO)_mCG shows the support for mCG methylation, and a negative value indicates the support for unmethylation.: "A positive value in Log(LIKELIHOOD_RATIO)_mCG shows the support for mCG methylation, and a negative value indicates the support for unmethylation."
- [readme] samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
