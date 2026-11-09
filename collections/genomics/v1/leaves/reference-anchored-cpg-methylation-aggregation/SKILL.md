---
name: reference-anchored-cpg-methylation-aggregation
description: Use when you have POD5/FAST5 Nanopore signal from an R9.4.1 or R10.4.1
  flowcell and need per-site CpG methylation frequencies anchored to a reference genome
  (GRCh38) for comparison or validation against other datasets.
when_to_use_negative:
- The BAM is unaligned and no reference FASTA is available — reference-anchored per-site
  frequencies and signal alignment require an aligned BAM
- You need methylation types other than 5mCpG — DeepMod2 currently only provides 5mCpG
  models due to lack of training data for other modifications
- Your signal files come from a flowcell without a matching DeepMod2 model (models
  are provided for R10.4.1 and R9.4.1 only)
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DeepMod2
  role: BiLSTM/Transformer-based 5mC methylation caller producing per-read predictions,
    MM/ML-tagged BAMs, and reference-anchored per-site methylation frequencies from
    POD5/FAST5
  repo: github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecalls POD5/FAST5 with --emit-moves to produce a BAM with move tables;
    can be given a reference to output aligned BAMs
  repo: https://github.com/nanoporetech/dorado
- name: modkit
  role: Converts modBAM (MM/ML-tagged BAM) to bedMethyl per-site tables using best
    practices
  repo: https://github.com/nanoporetech/modkit
- name: minimap2
  role: Aligns basecalled reads to the reference genome while retaining move tables
    when Dorado was not given a reference
- name: samtools
  role: Converts the basecalled BAM to FASTQ with mv,ts tags retained for realignment
- name: bedtools
  role: Intersects per-site methylation BED files from two datasets to keep overlapping
    CpGs before correlation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:c549f8c3361b9ad542594c150b22ec756830223e84bb34cb3889dd6b87bb0619
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepMod2
  - Dorado
  - bedtools
  - minimap2
  - modkit
  - samtools
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# reference-anchored-cpg-methylation-aggregation

## Summary

Convert Nanopore methylation calls into reference-anchored per-CpG methylation frequencies by basecalling POD5/FAST5 with Dorado/Guppy with move tables, aligning to a reference genome, running DeepMod2 detect, and combining per-read predictions into per-site methylation percentages for downstream comparison (e.g., RRMS vs. whole-genome correlation).

## When to use

Use when you have POD5/FAST5 Nanopore signal from an R9.4.1 or R10.4.1 flowcell and need per-site CpG methylation frequencies anchored to a reference genome (GRCh38) for comparison or validation against other datasets.

## When NOT to use

- The BAM is unaligned and no reference FASTA is available — reference-anchored per-site frequencies and signal alignment require an aligned BAM
- You need methylation types other than 5mCpG — DeepMod2 currently only provides 5mCpG models due to lack of training data for other modifications
- Your signal files come from a flowcell without a matching DeepMod2 model (models are provided for R10.4.1 and R9.4.1 only)

## Inputs

- POD5 or FAST5 raw signal files from R9.4.1 or R10.4.1 Nanopore flowcells
- BAM file basecalled with Dorado --emit-moves (or Guppy --bam_out --moves_out) containing move tables
- Reference genome FASTA (e.g., GRCh38.fa) for reference anchoring
- Optionally a BED file of target regions (e.g., RRMS targets from ONT's Adaptive Sampling Catalogue)

## Outputs

- Methylation-tagged BAM with MM and ML tags
- Per-read methylation TSV from DeepMod2
- Per-site methylation frequency BED files (output.per_site / output.per_site.aggregated, or bedMethyl from modkit)
- Paired per-site methylation values and Pearson correlation (e.g., methylation_correlation_scatter.tsv) when comparing two datasets

## How to apply

Basecall POD5 with Dorado using --emit-moves and a reference (e.g., GRCh38.fa) with model dna_r10.4.1_e8.2_400bps_hac@v4.3.0 so an aligned BAM with move tables is produced; alternatively align basecalled reads with minimap2 -ax map-ont while retaining mv,ts tags. Reference anchoring is what enables per-site aggregation, so an aligned BAM is required. Run DeepMod2's detect module (e.g., model bilstm_r10.4.1_5khz_v4.3, --file_type pod5, --ref GRCh38.fa) to obtain reference-anchored methylation calls; DeepMod2 adds MM/ML modification tags to the BAM and writes per-read TSV output, then in post-processing combines per-read predictions for each site into a percentage of methylated reads. Collect per-site frequencies from output.per_site / output.per_site.aggregated BED files (modkit can also convert modBAM to bedMethyl with best practices). For cross-dataset comparison (e.g., RRMS vs. whole-genome HG002), intersect per-site outputs with bedtools, drop low-coverage sites (<10X Nanopore), and compute Pearson correlation over overlapping CpGs.

## Related tools

- **DeepMod2** (BiLSTM/Transformer-based 5mC methylation caller producing per-read predictions, MM/ML-tagged BAMs, and reference-anchored per-site methylation frequencies from POD5/FAST5) — github.com/WGLab/DeepMod2
- **Dorado** (Basecalls POD5/FAST5 with --emit-moves to produce a BAM with move tables; can be given a reference to output aligned BAMs) — https://github.com/nanoporetech/dorado
- **modkit** (Converts modBAM (MM/ML-tagged BAM) to bedMethyl per-site tables using best practices) — https://github.com/nanoporetech/modkit
- **minimap2** (Aligns basecalled reads to the reference genome while retaining move tables when Dorado was not given a reference)
- **samtools** (Converts the basecalled BAM to FASTQ with mv,ts tags retained for realignment)
- **bedtools** (Intersects per-site methylation BED files from two datasets to keep overlapping CpGs before correlation)

## Examples

```
python deepmod2 detect --model bilstm_r10.4.1_5khz_v4.3 --file_type pod5 --bam aligned.bam --input signal_dir --ref GRCh38.fa --threads 8
```

## Evaluation signals

- The aligned BAM retains move tables (mv tag) after realignment; loss of mv tags breaks signal-to-basecall alignment
- Per-site methylation frequencies are only produced for CpG sites when a reference FASTA is supplied and the BAM is aligned
- Nanopore coverage filter of ≥10X applied to overlapping CpG sites before correlation
- Pearson r computed between RRMS and whole-genome per-site methylation percentages over intersected CpGs should closely match the reported r = 0.96 (95.78–96.45% range)
- Per-site correlation with short-read ground truth should exceed r > 0.95, consistent with DeepMod2's reported ~99% per-site F1

## Limitations

- Move tables provide only approximate signal-to-basecall alignment; boundaries between consecutive bases can be mis-demarcated, leading to uninformative signal summary statistics in some cases
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Filtering intermediate per-read predictions (e.g., 0.2–0.8 probability band) reduces coverage and statistical power, especially for low-coverage samples
- No models available for modifications other than 5mCpG; no 5hmC detected in GIAB cell lines HG001-HG007

## Evidence

- [results] It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls: "It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls"
- [results] In the post-processing step, all per-read predictions for a site are combined to calculate the percentage of total reads with methylated cytosine: "In the post-processing step, all per-read predictions for a site are combined to calculate the percentage of total reads with methylated cytosine"
- [results] The BAM file is recommended to be aligned to a reference genome to allow reference anchoring of signals and estimation of per-site methylation levels: "The BAM file is recommended to be aligned to a reference genome to allow reference anchoring of signals and estimation of per-site methylation levels"
- [readme] samtools fastq basecall.bam -T mv,ts | minimap2 -ax map-ont ref.fa - -y -t NUM_THREADS |samtools view -o aligned.bam: [evidence span withheld — anchor status: external_doc]
- [results] after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively): "after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively)"
