---
name: signal-based-cpg-methylation-detection
description: Use when POD5 or FAST5 Nanopore signal files from R9.4.1 or R10.4.1 flowcells
  must be converted into per-read and per-site CpG 5mC methylation calls (MM/ML-tagged
  BAM, BED outputs) without bisulfite conversion.
when_to_use_negative:
- Detection of modifications other than 5mCpG (e.g. 5hmC, 6mA) — DeepMod2 currently
  only provides 5mCpG models and there is no 5hmC detected in GIAB cell lines HG001-HG007
- R9-only third-party callers would be the wrong choice for R10.4.1 data — tools like
  Nanopolish, DeepMP, DeepSignal, Tombo, methBERT and Rockfish provide models only
  for R9.4 flowcells
- Input is already a methylation feature table or bedMethyl with no raw signal — signal-level
  detection is unnecessary
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: DeepMod2
  role: Core BiLSTM/Transformer signal-based 5mC caller; runs 'detect' on signal files
    plus aligned BAM to emit MM/ML tags and per-read/per-site calls
  repo: https://github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecaller used with --emit-moves to produce BAMs with move tables needed
    for signal-to-basecall alignment
  repo: https://github.com/nanoporetech/dorado
- name: Guppy
  role: Alternative basecaller used with --bam_out --moves_out to generate move-table
    BAMs
- name: minimap2
  role: Aligns basecalled reads to the reference (-ax map-ont, -y) preserving move-table
    tags for reference-anchored calling
- name: samtools
  role: Sorts/indexes the methylation-tagged output BAM (samtools sort --write-index)
    and pipes reads for alignment
- name: modkit
  role: Converts modBAM outputs to bedMethyl tables and summarizes modified-base statistics
  repo: https://github.com/nanoporetech/modkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:0ed0c39e1c2d253536f700100b3275721bd2019cdb4f43e879691d0de4e0286b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - Guppy
  - minimap2
  - modkit
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# signal-based-cpg-methylation-detection

## Summary

Detect 5mC methylation at CpG sites directly from Oxford Nanopore ionic current signal (FAST5/POD5) using DeepMod2's BiLSTM/Transformer models, producing methylation-tagged BAM files with MM/ML tags plus per-read and per-site outputs.

## When to use

Use when POD5 or FAST5 Nanopore signal files from R9.4.1 or R10.4.1 flowcells must be converted into per-read and per-site CpG 5mC methylation calls (MM/ML-tagged BAM, BED outputs) without bisulfite conversion.

## When NOT to use

- Detection of modifications other than 5mCpG (e.g. 5hmC, 6mA) — DeepMod2 currently only provides 5mCpG models and there is no 5hmC detected in GIAB cell lines HG001-HG007
- R9-only third-party callers would be the wrong choice for R10.4.1 data — tools like Nanopolish, DeepMP, DeepSignal, Tombo, methBERT and Rockfish provide models only for R9.4 flowcells
- Input is already a methylation feature table or bedMethyl with no raw signal — signal-level detection is unnecessary

## Inputs

- POD5 signal files (R10.4.1) or FAST5 signal files (R9.4.1)
- Basecalled BAM with move tables from Dorado (--emit-moves) or Guppy (--bam_out --moves_out)
- minimap2-aligned BAM against a reference genome (e.g. GRCh38 analysis-set FASTA with .fai index)
- Chemistry-matched DeepMod2 model (e.g. bilstm_r10.4.1_5khz_v4.3)

## Outputs

- Methylation-tagged output BAM with MM/ML 5mC modification tags
- Per-read methylation TSV (tab-separated per-read CpG predictions)
- Per-site BED output with aggregated methylation frequencies
- Sorted/indexed BAM produced via samtools sort --write-index

## How to apply

Basecall the signal files with Dorado (--emit-moves, e.g. model dna_r10.4.1_e8.2_400bps_hac@v4.3.0) or Guppy (--bam_out --moves_out) so BAMs carry move tables for signal-to-basecall alignment. Align basecalls to GRCh38 with minimap2 -ax map-ont -y to preserve move-table tags, then run DeepMod2 detect in reference-anchored mode with a chemistry-matched model (e.g. bilstm_r10.4.1_5khz_v4.3 for R10.4.1) so 5mC is called at read CpG motifs and reference CpGs. The tool extracts 19 signal summary statistics per base in a 21-bp window centered at each CpG (excluding windows with <0.75 read-reference identity), normalizes raw signal by median absolute deviation, and feeds the feature matrix to the BiLSTM or Transformer model for per-read prediction at a 0.5 probability threshold. Per-read predictions are written as MM/ML modification tags into the BAM plus a TSV, and post-processing combines per-read calls into per-site methylation frequencies; sort/index outputs with samtools sort --write-index and optionally summarize with modkit into bedMethyl.

## Related tools

- **DeepMod2** (Core BiLSTM/Transformer signal-based 5mC caller; runs 'detect' on signal files plus aligned BAM to emit MM/ML tags and per-read/per-site calls) — https://github.com/WGLab/DeepMod2
- **Dorado** (Basecaller used with --emit-moves to produce BAMs with move tables needed for signal-to-basecall alignment) — https://github.com/nanoporetech/dorado
- **Guppy** (Alternative basecaller used with --bam_out --moves_out to generate move-table BAMs)
- **minimap2** (Aligns basecalled reads to the reference (-ax map-ont, -y) preserving move-table tags for reference-anchored calling)
- **samtools** (Sorts/indexes the methylation-tagged output BAM (samtools sort --write-index) and pipes reads for alignment)
- **modkit** (Converts modBAM outputs to bedMethyl tables and summarizes modified-base statistics) — https://github.com/nanoporetech/modkit

## Examples

```
python PATH_TO_DEEPMOD2/deepmod2 detect --bam aligned.bam --input pod5_dir --model bilstm_r10.4.1_5khz_v4.3 --file_type pod5 --threads 8 --ref GRCh38.fa --output mod_calls
```

## Evaluation signals

- All output.sorted.bam files exist, are valid sorted/indexable BAMs, and carry MM/ML 5mC modification tags
- Per-read benchmark F1 falls within the paper-reported 95.7–97.1% range on R9.4.1 Ashkenazim trio (HG002, HG003, HG004); per-site F1 approaches ~99%
- Correlation with short-read sequencing ground truth exceeds r > 0.95
- Per-read ground-truth comparison restricted to sites ≥90% or <10% methylation frequency with ≥10X short-read coverage; per-site ground truth at ≥80%/<20% with ≥10X (HG002-4) or ≥5X (NIH3T3) coverage
- Per-site methylation frequencies derived as the percentage of total reads with methylated cytosine match expectations (per-site calls labeled methylated at ≥50% frequency)

## Limitations

- Move tables provide only approximate signal-to-basecall alignment; boundaries between consecutive bases can be mis-demarcated, yielding uninformative signal summary statistics
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Only 5mCpG models exist due to lack of training data for other modifications; no 5hmC detected in GIAB cell lines HG001-HG007
- Filtering intermediate per-read predictions reduces coverage and statistical power for downstream per-site analyses, especially in low-coverage samples

## Evidence

- [abstract] DeepMod2 implements both a bidirectional long short-term memory (BiLSTM) model and a Transformer model and can analyze POD5 and FAST5 signal files generated on R9 and R10 flowcells: "DeepMod2 implements both a bidirectional long short-term memory (BiLSTM) model and a Transformer model and can analyze POD5 and FAST5 signal files"
- [readme] basecalling with move tables: [evidence span withheld — anchor status: external_doc]
- [results] feature extraction window: "get signal summary statistics and alignment information for each base of a read in a 21-bp window centered at the CpG of interest"
- [results] per-read performance: "For R9.4.1 datasets of the Ashkenazim trio HG002, HG003 and HG004, the F1-score of DeepMod2 models is in the 95.7–97.1% range"
- [results] output tags: "DeepMod2 adds per-read predictions to the input BAM file as modification tags and also produces a tab separated text file output"
- [results] feature identity filter: "if the read sequence had less than 0.75 percent identity with aligned the reference sequence in the 21-bp window centered at the CpG site, we excluded that feature matrix"
