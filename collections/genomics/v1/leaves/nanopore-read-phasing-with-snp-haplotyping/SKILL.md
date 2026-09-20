---
name: nanopore-read-phasing-with-snp-haplotyping
description: Use when ONT reads need to be assigned to parental haplotypes to resolve
  haplotype-specific methylation — e.g. detecting imprinted control regions, parent-of-origin
  labeling in a trio, or differential methylation between phases.
when_to_use_negative:
- The target region lies in long runs of homozygosity — read phasing may not be possible
  there, so haplotype-resolved methylation counts will be unreliable
- Only bulk per-site methylation frequencies are needed with no haplotype context
  — run DeepMod2 detect on the unphased aligned BAM instead
- The modification of interest is not 5mCpG — DeepMod2 currently only provides models
  for 5mCpG due to lack of training data for other modifications
edam_operation: http://edamontology.org/operation_3236
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0622
tools:
- name: NanoCaller
  role: Phases reads into parental haplotypes by SNP haplotyping (--mode snps --phase)
    across chr1-22XY
- name: Dorado
  role: Basecalls POD5 signal with move table emission (--emit-moves) so signals can
    later be anchored during methylation detection
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Aligns basecalled reads to GRCh38 (-ax map-ont -y) while retaining move tables
    for reference-anchored methylation calls
- name: samtools
  role: Sorts/indexes the aligned BAM and concatenates per-chromosome phased BAMs
    into phased.bam
- name: DeepMod2
  role: Calls haplotype-resolved 5mC methylation on the phased BAM (bilstm_r10.4.1_5khz_v4.3,
    reference-anchored) producing per-site phase1/phase2 methylation percentages
  repo: https://github.com/WGLab/DeepMod2
- name: bedtools
  role: Intersects phased per-site methylation BEDs with curated ICR coordinates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:6b59f8d3d8957fdb11a783aa7c12ff9034471e3a5e1491ababdfc0b3dff2a4ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - DeepMod2
  - Dorado
  - NanoCaller
  - bedtools
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3236
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# nanopore-read-phasing-with-snp-haplotyping

## Summary

Phases ONT reads into parental haplotypes via SNP haplotyping (NanoCaller) and then calls haplotype-resolved 5mC methylation with DeepMod2, enabling detection of allele-specific (epihaplotype) methylation such as imprinted control regions in the Ashkenazim trio (HG002/HG003/HG004).

## When to use

Use when ONT reads need to be assigned to parental haplotypes to resolve haplotype-specific methylation — e.g. detecting imprinted control regions, parent-of-origin labeling in a trio, or differential methylation between phases.

## When NOT to use

- The target region lies in long runs of homozygosity — read phasing may not be possible there, so haplotype-resolved methylation counts will be unreliable
- Only bulk per-site methylation frequencies are needed with no haplotype context — run DeepMod2 detect on the unphased aligned BAM instead
- The modification of interest is not 5mCpG — DeepMod2 currently only provides models for 5mCpG due to lack of training data for other modifications

## Inputs

- POD5 (or FAST5) raw signal files from ONT R10.4.1 sequencing
- Basecalled/aligned BAM with move tables (Dorado --emit-moves, minimap2 -ax map-ont -y)
- GRCh38 reference FASTA
- Curated imprinting control region coordinates (Jima et al. Supplementary Table S1 / humanicr.org)
- Trio samples for parent-of-origin assignment (HG002, HG003, HG004)

## Outputs

- Phased BAM (per-chromosome NanoCaller outputs merged with samtools cat into phased.bam)
- DeepMod2 phased per-site methylation output (output.per_site and output.per_site.aggregated with mod_percentage_phase1/phase2 columns)
- Epihaplotype classification of ICRs per phase (near-0% vs near-100% methylation) with parent-of-origin labels
- Concordance table of predicted imprinted regions across trio members

## How to apply

Basecall POD5 signal with Dorado (model dna_r10.4.1_e8.2_400bps_hac@v4.3.0, --emit-moves) and align to GRCh38 with minimap2 (-ax map-ont -y), keeping the move tables for later signal anchoring. Phase reads with NanoCaller in SNP mode (--mode snps --phase --wgs_contigs chr1-22XY --cpu 8) and merge per-chromosome phased BAMs with samtools cat into a single phased.bam carrying haplotype tags. Run DeepMod2 detect (bilstm_r10.4.1_5khz_v4.3, reference-anchored mode with --ref GRCh38 and --threads 8) on the phased BAM plus POD5 input to produce output.per_site files with mod_percentage_phase1/mod_percentage_phase2 columns. Intersect these per-site BEDs with curated ICR coordinates (Jima et al. Supplementary Table S1 / humanicr.org) using bedtools intersect, then classify each region's epihaplotype: one phase near 0% methylation and the other near 100% indicates a fully unmethylated vs fully methylated parental allele, from which trio parent-of-origin labels are assigned. Downstream differential methylation can use DSS with DMR criteria of ≥100 bp length, ≥15 CpGs, ≥50% significantly differentially methylated CpGs, and merging within 100 bp at p<1e-5.

## Related tools

- **NanoCaller** (Phases reads into parental haplotypes by SNP haplotyping (--mode snps --phase) across chr1-22XY)
- **Dorado** (Basecalls POD5 signal with move table emission (--emit-moves) so signals can later be anchored during methylation detection) — https://github.com/nanoporetech/dorado
- **minimap2** (Aligns basecalled reads to GRCh38 (-ax map-ont -y) while retaining move tables for reference-anchored methylation calls)
- **samtools** (Sorts/indexes the aligned BAM and concatenates per-chromosome phased BAMs into phased.bam)
- **DeepMod2** (Calls haplotype-resolved 5mC methylation on the phased BAM (bilstm_r10.4.1_5khz_v4.3, reference-anchored) producing per-site phase1/phase2 methylation percentages) — https://github.com/WGLab/DeepMod2
- **bedtools** (Intersects phased per-site methylation BEDs with curated ICR coordinates)

## Examples

```
NanoCaller --mode snps --phase --wgs_contigs chr1-22XY --cpu 8 -b aligned.bam -r GRCh38.fa -o phased_out/ ; samtools cat phased_out/chr*.phased.bam > phased.bam
```

## Evaluation signals

- A correctly classified ICR shows one phase with methylation near 0% and the other near 100% — e.g. chr11:2699000-2702000 with mod_percentage_phase1 = 0 and mod_percentage_phase2 approaching 1
- 19, 20, and 19 of 21 known imprinting control regions are detected in HG002, HG003, and HG004, respectively
- Parent-of-origin labels are concordant across all three trio members (HG002/HG003/HG004)
- The merged phased.bam is sorted/indexed and DeepMod2 emits separate mod_percentage_phase1/phase2 columns rather than a single blended frequency

## Limitations

- Read phasing is not possible in long runs of homozygosity, which limits reliance on phased methylation counts for analyzing imprinted regions
- Phasing was run only on chr1-22XY, so haplotype-resolved calls for unplaced/alt contigs are not produced
- Differential methylation tools like DSS use only binary methylated/unmethylated counts, ignoring per-read methylation probabilities and long-read co-occurrence
- DeepMod2 only supports 5mCpG models; other methylation types (e.g. 5hmC) are not detected in GIAB cell lines HG001-HG007

## Evidence

- [methods] phase reads into parental haplotypes with NanoCaller (--mode snps --phase --wgs_contigs chr1-22XY --cpu 8), merging per-chromosome phased BAMs with samtools cat into phased.bam: [evidence span withheld — anchor status: not_found]
- [intro] we demonstrate that phased methylation calls from DeepMod2 can be used to accurately predict imprinted regions in human cell-lines: "we demonstrate that phased methylation calls from DeepMod2 can be used to accurately predict imprinted regions in human cell-lines"
- [results] obtained the list of all candidate ICRs from Supplementary Table S1 of Jima et al.36, which can also be found at https://humanicr.org/: "obtained the list of all candidate ICRs from Supplementary Table S1 of Jima et al.36, which can also be found at https://humanicr.org/"
- [results] This illustrates a drawback in depending solely upon phased methylation counts for analyzing imprinted regions, because read phasing may not be possible in regions with runs of homozygosity: "read phasing may not be possible in regions with runs of homozygosity"
- [readme] Dorado basecalling with move tables is a prerequisite for downstream signal-anchored methylation detection on the phased BAM: [evidence span withheld — anchor status: external_doc]
