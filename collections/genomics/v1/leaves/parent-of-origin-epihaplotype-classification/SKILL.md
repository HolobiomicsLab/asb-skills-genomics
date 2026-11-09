---
name: parent-of-origin-epihaplotype-classification
description: Use when you have trio (e.g. Ashkenazim HG002/HG003/HG004) Oxford Nanopore
  POD5/BAM data and need to detect imprinted regions or assign parent-of-origin to
  haplotype-specific (epihaplotype) methylation patterns.
when_to_use_negative:
- Samples lacking trio/parent-offspring structure or phased variant calls — haplotype
  phasing and parent-of-origin assignment cannot be determined
- Imprinted regions falling in long runs of homozygosity, where read phasing may not
  be possible and phased methylation counts alone are unreliable
- Modifications other than 5mCpG — DeepMod2 currently only provides models for 5mCpG
  due to lack of training data for other methylation types
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3307
tools:
- name: DeepMod2
  role: Reference-anchored phased per-site methylation detection on phased BAM with
    POD5 signal input (bilstm_r10.4.1_5khz_v4.3 model)
  repo: https://github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecalls POD5 signal files with --emit-moves to produce BAM with move tables
    for signal alignment
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Aligns basecalled ONT reads to GRCh38 (-ax map-ont -y) while retaining move
    tables
- name: NanoCaller
  role: Phases reads into parental haplotypes (--mode snps --phase --wgs_contigs chr1-22XY)
- name: samtools
  role: Sorts/indexes BAMs and merges per-chromosome phased BAMs (samtools cat) into
    phased.bam
- name: bedtools
  role: Intersects phased per-site methylation BED files with curated ICR coordinates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:482a6bd33b12aa1901d2db5faf3419bccfb11f344d3ce2c7fa00aa9544483a64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - NanoCaller
  - bedtools
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# parent-of-origin-epihaplotype-classification

## Summary

Classifies parent-of-origin methylation states (epihaplotypes) at imprinting control regions by combining haplotype-phased reads with phased per-site methylation calls from DeepMod2 on ONT data. Enables trio-based assignment of methylated vs unmethylated parental alleles at known ICRs.

## When to use

Use when you have trio (e.g. Ashkenazim HG002/HG003/HG004) Oxford Nanopore POD5/BAM data and need to detect imprinted regions or assign parent-of-origin to haplotype-specific (epihaplotype) methylation patterns.

## When NOT to use

- Samples lacking trio/parent-offspring structure or phased variant calls — haplotype phasing and parent-of-origin assignment cannot be determined
- Imprinted regions falling in long runs of homozygosity, where read phasing may not be possible and phased methylation counts alone are unreliable
- Modifications other than 5mCpG — DeepMod2 currently only provides models for 5mCpG due to lack of training data for other methylation types

## Inputs

- Ashkenazim trio (HG002/HG003/HG004) ONT POD5 raw signal files
- GRCh38 reference FASTA
- Curated imprinting control region coordinates (Jima et al. Supplementary Table S1, humanicr.org)
- Dorado basecalled BAM with move tables (--emit-moves)

## Outputs

- phased.bam (reads assigned to parental haplotypes)
- DeepMod2 output.per_site and output.per_site.aggregated files with mod_percentage_phase1/phase2 columns
- Per-ICR epihaplotype classification table with parent-of-origin labels per trio member
- Concordance statistics of predicted vs curated ICRs across HG002, HG003, HG004

## How to apply

Basecall the trio POD5 signal files with Dorado (model dna_r10.4.1_e8.2_400bps_hac@v4.3.0, --emit-moves) and align to GRCh38 with minimap2 (-ax map-ont -y), then sort/index with samtools. Phase reads into parental haplotypes with NanoCaller (--mode snps --phase --wgs_contigs chr1-22XY --cpu 8) and merge per-chromosome phased BAMs with samtools cat into phased.bam. Run DeepMod2 detect (bilstm_r10.4.1_5khz_v4.3 model, reference-anchored mode, --threads 8) on the phased BAM plus POD5 and GRCh38 reference to produce phased per-site output containing mod_percentage_phase1 and mod_percentage_phase2 columns. Intersect the phased per-site BED files with curated imprint control region coordinates (Jima et al. Supplementary Table S1 / humanicr.org) using bedtools intersect, then classify each ICR per phase: near-0% methylation in one phase and near-100% in the other indicates the unmethylated vs methylated parental allele, allowing parent-of-origin labels to be assigned across trio members. Finally compute concordance between predicted imprinted methylation patterns and curated ICR expectations; the reference result detects 19, 20, and 19 of 21 known ICRs in HG002, HG003, and HG004 respectively.

## Related tools

- **DeepMod2** (Reference-anchored phased per-site methylation detection on phased BAM with POD5 signal input (bilstm_r10.4.1_5khz_v4.3 model)) — https://github.com/WGLab/DeepMod2
- **Dorado** (Basecalls POD5 signal files with --emit-moves to produce BAM with move tables for signal alignment) — https://github.com/nanoporetech/dorado
- **minimap2** (Aligns basecalled ONT reads to GRCh38 (-ax map-ont -y) while retaining move tables)
- **NanoCaller** (Phases reads into parental haplotypes (--mode snps --phase --wgs_contigs chr1-22XY))
- **samtools** (Sorts/indexes BAMs and merges per-chromosome phased BAMs (samtools cat) into phased.bam)
- **bedtools** (Intersects phased per-site methylation BED files with curated ICR coordinates)

## Examples

```
dorado basecaller dna_r10.4.1_e8.2_400bps_hac@v4.3.0 INPUT_DIR --emit-moves > basecall.bam
```

## Evaluation signals

- A correctly classified ICR shows one phase completely unmodified (mod_percentage_phase1 = 0) and the other nearly completely modified (mod_percentage_phase2 approaches 1), e.g. chr11:2699000-2702000
- Concordance against curated ICRs: ~19-20 of 21 known imprinting control regions detected per trio member (HG002: 19, HG003: 20, HG004: 19)
- Parent-of-origin labels are mutually consistent across the trio: HG003/HG004 parental alleles complement HG002's biparental pattern
- Output.per_site.aggregated contains mod_percentage_phase1 and mod_percentage_phase2 columns for phased samples; absence indicates phasing failed upstream
- Per-read methylation F1 on R10.4.1 trio data should be in the range DeepMod2 reports (~95% F1 per-read, ~99% per-site)

## Limitations

- Read phasing may not be possible in regions with runs of homozygosity, a drawback when depending solely on phased methylation counts for imprinted region analysis
- Move tables provide only approximate signal-to-basecall alignment, which can lead to uninformative signal summary statistics
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- No models available for modifications other than 5mCpG; no 5hmC detected in GIAB cell lines HG001-HG007

## Evidence

- [intro] phased methylation calls can accurately predict imprinted regions: "we demonstrate that phased methylation calls from DeepMod2 can be used to accurately predict imprinted regions in human cell-lines"
- [results] runs of homozygosity limit phasing: "read phasing may not be possible in regions with runs of homozygosity"
- [results] ICR list source: "obtained the list of all candidate ICRs from Supplementary Table S1 of Jima et al.36, which can also be found at https://humanicr.org/"
- [readme] Dorado basecalling with move tables: [evidence span withheld — anchor status: external_doc]
