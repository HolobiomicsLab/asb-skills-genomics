---
name: haplotype-phased-methylation-calling
description: Use when ONT POD5/FAST5 + BAM data from diploid samples (e.g. Ashkenazim
  trio HG002/HG003/HG004) must be phased into parental haplotypes and allele-specific
  methylation (epihaplotypes) or imprinted regions must be identified.
when_to_use_negative:
- The sample region falls in long runs of homozygosity where read phasing (and therefore
  phased methylation analysis of imprinted regions) is not possible.
- Only aggregate per-site methylation is needed without allele/phase resolution —
  a direct DeepMod2/Guppy/Dorado per-site call on the unphased BAM suffices.
- Non-CpG or non-5mC modifications are of interest — DeepMod2 currently provides models
  only for 5mCpG methylation due to lack of training data for other modifications.
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DeepMod2
  role: Deep learning (BiLSTM/Transformer) per-read 5mC calling from ONT signal on
    phased BAMs; reference-anchored mode produces phased per-site methylation frequencies
  repo: https://github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecalls POD5 signal with --emit-moves to generate BAM with move tables required
    for signal alignment
  repo: https://github.com/nanoporetech/dorado
- name: NanoCaller
  role: Phases reads into parental haplotypes (--mode snps --phase) to produce the
    phased BAM input for DeepMod2
- name: minimap2
  role: Aligns basecalled ONT reads to GRCh38 (-ax map-ont -y) while retaining move
    tables for reference anchoring
- name: samtools
  role: Sorts, indexes, and merges per-chromosome phased BAMs (samtools cat) into
    phased.bam
- name: bedtools
  role: Intersects phased per-site methylation BED files with curated ICR coordinates
- name: modkit
  role: Converts modBAM MM/ML tags to bedMethyl summary tables using ONT best practices
  repo: https://github.com/nanoporetech/modkit
- name: DSS
  role: Differential methylation detection on phased methylation counts under criteria
    of ≥100 bp length, ≥15 CpGs, ≥50% significantly differentially methylated CpGs,
    p<1e-5
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:aacc2684f11d52b744a4d2b82baf12a8d70faff454d1237744a43dfe74b21f80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DSS
  - DeepMod2
  - Dorado
  - NanoCaller
  - bedtools
  - minimap2
  - modkit
  - samtools
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# haplotype-phased-methylation-calling

## Summary

Detect per-read 5mC methylation from Oxford Nanopore signal with DeepMod2 on haplotype-phased BAMs (NanoCaller phasing), yielding phased per-site methylation frequencies (mod_percentage_phase1/phase2) that can be classified into epihaplotype states to predict imprinted control regions.

## When to use

Use when ONT POD5/FAST5 + BAM data from diploid samples (e.g. Ashkenazim trio HG002/HG003/HG004) must be phased into parental haplotypes and allele-specific methylation (epihaplotypes) or imprinted regions must be identified.

## When NOT to use

- The sample region falls in long runs of homozygosity where read phasing (and therefore phased methylation analysis of imprinted regions) is not possible.
- Only aggregate per-site methylation is needed without allele/phase resolution — a direct DeepMod2/Guppy/Dorado per-site call on the unphased BAM suffices.
- Non-CpG or non-5mC modifications are of interest — DeepMod2 currently provides models only for 5mCpG methylation due to lack of training data for other modifications.

## Inputs

- POD5 or FAST5 raw signal files from ONT R10.4.1 (or R9.4.1) flowcells
- BAM file basecalled with Dorado/Guppy including move tables (MM/ML tag support)
- Reference genome FASTA (GRCh38)
- Curated imprinting control region coordinates (Jima et al. Supplementary Table S1 / humanicr.org)
- DSS differential methylation results (DMRs: ≥100 bp, ≥15 CpGs, ≥50% significantly differentially methylated CpGs, merge within 100 bp, p<1e-5)

## Outputs

- Phased BAM (phased.bam) with reads assigned to parental haplotypes
- Methylation-tagged BAM with per-read MM/ML modification tags and per-read TSV
- Phased per-site methylation output (output.per_site, output.per_site.aggregated) with mod_percentage_phase1/phase2 columns
- bedMethyl tables and ICR epihaplotype classification with parent-of-origin labels
- Concordance statistics of predicted imprinted regions across HG002, HG003, HG004

## How to apply

Basecall POD5 signal with Dorado (e.g. dna_r10.4.1_e8.2_400bps_hac@v4.3.0 with --emit-moves), align to GRCh38 with minimap2 (-ax map-ont -y) while retaining move tables, then sort/index with samtools. Phase reads into parental haplotypes using NanoCaller (--mode snps --phase --wgs_contigs chr1-22XY --cpu 8), merging per-chromosome phased BAMs with samtools cat into a single phased.bam. Run DeepMod2 detect with a model matching the flowcell (e.g. bilstm_r10.4.1_5khz_v4.3) on the phased BAM plus POD5 input and reference in reference-anchored mode (--threads 8), which writes output.per_site and output.per_site.aggregated files containing mod_percentage_phase1/phase2 columns. Intersect phased per-site BED files with curated imprinting control region coordinates (Jima et al. Supplementary Table S1 / humanicr.org) using bedtools intersect, and classify each ICR's epihaplotype state per phase: near-0% methylation in one phase and near-100% in the other indicates an unmethylated vs methylated parental allele, enabling parent-of-origin assignment across trio members.

## Related tools

- **DeepMod2** (Deep learning (BiLSTM/Transformer) per-read 5mC calling from ONT signal on phased BAMs; reference-anchored mode produces phased per-site methylation frequencies) — https://github.com/WGLab/DeepMod2
- **Dorado** (Basecalls POD5 signal with --emit-moves to generate BAM with move tables required for signal alignment) — https://github.com/nanoporetech/dorado
- **NanoCaller** (Phases reads into parental haplotypes (--mode snps --phase) to produce the phased BAM input for DeepMod2)
- **minimap2** (Aligns basecalled ONT reads to GRCh38 (-ax map-ont -y) while retaining move tables for reference anchoring)
- **samtools** (Sorts, indexes, and merges per-chromosome phased BAMs (samtools cat) into phased.bam)
- **bedtools** (Intersects phased per-site methylation BED files with curated ICR coordinates)
- **modkit** (Converts modBAM MM/ML tags to bedMethyl summary tables using ONT best practices) — https://github.com/nanoporetech/modkit
- **DSS** (Differential methylation detection on phased methylation counts under criteria of ≥100 bp length, ≥15 CpGs, ≥50% significantly differentially methylated CpGs, p<1e-5)

## Examples

```
python deepmod2 detect --bam phased.bam --input pod5/ --model bilstm_r10.4.1_5khz_v4.3 --file_type pod5 --threads 8 --ref GRCh38.fa --output mod_calls
```

## Evaluation signals

- A correctly classified ICR shows one phase completely unmodified and the other nearly completely modified (e.g. chr11:2699000-2702000 with mod_percentage_phase1 = 0 and mod_percentage_phase2 approaching 1).
- 19, 20, and 19 of 21 known imprinting control regions are detected in HG002, HG003, and HG004 respectively with the NanoCaller-DeepMod2-DSS pipeline.
- output.per_site.aggregated contains valid mod_percentage_phase1/phase2 columns for phased sites, and the methylation-tagged BAM carries MM/ML modification tags.
- Parent-of-origin labels assigned per phase are concordant across the trio members (HG002 child, HG003/HG004 parents).
- Overall DeepMod2 per-read F1-score on the Ashkenazim trio R9.4.1 datasets falls in the 95.7–97.1% range against short-read/microarray ground truth.

## Limitations

- Depending solely on phased methylation counts for imprinted-region analysis fails where read phasing is not possible in regions with runs of homozygosity.
- Move tables only approximate signal-to-basecall alignment, which can produce uninformative signal summary statistics for some bases.
- Filtering intermediate per-read probability predictions (e.g. around the 0.5 threshold) reduces per-site coverage and statistical power, especially for low-coverage samples.
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels.
- Current differential methylation tools (e.g. DSS) use binary methylated/unmethylated counts and ignore per-read methylation probabilities and long-read co-occurrence of modifications.

## Evidence

- [intro] we demonstrate that phased methylation calls from DeepMod2 can be used to accurately predict imprinted regions in human cell-lines: "we demonstrate that phased methylation calls from DeepMod2 can be used to accurately predict imprinted regions in human cell-lines"
- [results] The BAM file is recommended to be aligned to a reference genome to allow reference anchoring of signals and estimation of per-site methylation levels: "The BAM file is recommended to be aligned to a reference genome to allow reference anchoring of signals and estimation of per-site methylation levels"
- [readme] It uses a BiLSTM model to predict per-read and per-site 5mC methylations for CpG sites and produces a methylation tagged BAM file.: "It uses a BiLSTM model to predict per-read and per-site 5mC methylations for CpG sites and produces a methylation tagged BAM file."
- [results] read phasing may not be possible in regions with runs of homozygosity: "This illustrates a drawback in depending solely upon phased methylation counts for analyzing imprinted regions, because read phasing may not be possible in regions with runs of homozygosity"
