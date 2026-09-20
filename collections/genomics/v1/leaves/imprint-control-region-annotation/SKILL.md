---
name: imprint-control-region-annotation
description: Use when you have phased per-site methylation output (per_site / per_site.aggregated
  BED with mod_percentage_phase1 and mod_percentage_phase2 columns) from Nanopore
  reads of a human sample or trio and need to detect imprinted regions and parent-of-origin
  methylation.
when_to_use_negative:
- The ICR lies in a long run of homozygosity where read phasing is not possible, so
  phased methylation counts alone are unreliable
- The target modification is not 5mCpG (DeepMod2 provides models only for 5mCpG; no
  5hmC detected in GIAB cell lines HG001-HG007)
- Input is unphased per-site methylation without mod_percentage_phase1/phase2 columns,
  so allele-specific epihaplotype classification cannot be performed
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0622
tools:
- name: DeepMod2
  role: Generates phased per-site methylation output (per_site and per_site.aggregated
    with mod_percentage_phase1/phase2) used for ICR epihaplotype classification
  repo: https://github.com/WGLab/DeepMod2
- name: NanoCaller
  role: Phases reads into parental haplotypes (--mode snps --phase) to enable phase-resolved
    methylation calls
- name: bedtools
  role: Intersects phased per-site methylation BED files with the curated ICR coordinate
    BED
- name: DSS
  role: Detects differentially methylated regions (≥100 bp, ≥15 CpGs, ≥50% significant
    CpGs, p<1e-5) supporting imprinted-region calls
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:9432eac8436229c99ad41b347fea607fac75b12b6607f8cfdd5189e0272d8b63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DSS
  - DeepMod2
  - NanoCaller
  - bedtools
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# imprint-control-region-annotation

## Summary

Annotates imprinting control regions (ICRs) with allele-specific methylation state by intersecting phased per-site methylation calls from DeepMod2 with the curated ICR catalog (Jima et al. Supplementary Table S1 / humanicr.org), assigning epihaplotype and parent-of-origin labels from mod_percentage_phase1/phase2 values.

## When to use

Use when you have phased per-site methylation output (per_site / per_site.aggregated BED with mod_percentage_phase1 and mod_percentage_phase2 columns) from Nanopore reads of a human sample or trio and need to detect imprinted regions and parent-of-origin methylation.

## When NOT to use

- The ICR lies in a long run of homozygosity where read phasing is not possible, so phased methylation counts alone are unreliable
- The target modification is not 5mCpG (DeepMod2 provides models only for 5mCpG; no 5hmC detected in GIAB cell lines HG001-HG007)
- Input is unphased per-site methylation without mod_percentage_phase1/phase2 columns, so allele-specific epihaplotype classification cannot be performed

## Inputs

- Phased per-site methylation BED files from DeepMod2 (output.per_site and output.per_site.aggregated with mod_percentage_phase1/phase2 columns)
- Curated imprinting control region coordinates (Jima et al. Supplementary Table S1 / humanicr.org) as BED against GRCh38
- Phased BAM (NanoCaller-phased, per-chromosome BAMs merged with samtools cat)
- GRCh38 reference FASTA
- Trio pedigree relationships (e.g., Ashkenazim trio HG002/HG003/HG004) for parent-of-origin assignment

## Outputs

- Epihaplotype state per ICR and per haplotype phase (unmethylated vs methylated parental allele)
- Parent-of-origin labels for ICRs in trio samples
- Concordance table of predicted imprinted regions vs curated ICR expectations across HG002, HG003, HG004
- Optionally, DSS differential methylation regions meeting the 100 bp / 15 CpG / 50% DMC / p<1e-5 criteria

## How to apply

Obtain the candidate ICR list from Supplementary Table S1 of Jima et al. (also at humanicr.org) and convert the 21 curated ICR coordinates to a BED file against GRCh38. Run phased methylation detection (e.g., NanoCaller --mode snps --phase phasing into phased.bam, then DeepMod2 detect with the bilstm_r10.4.1_5khz_v4.3 model in reference-anchored mode) so each CpG carries phase-resolved methylation percentages. Intersect the phased per-site BED files with the ICR BED using bedtools intersect, then classify each ICR's epihaplotype per phase: near-0% mod_percentage in one phase and near-100% in the other indicates an unmethylated vs methylated parental allele (e.g., chr11:2699000-2702000 with mod_percentage_phase1=0 and mod_percentage_phase2 approaching 1). For trios, compare phase-specific methylation across HG002/HG003/HG004 to assign parent-of-origin labels; optionally support calls with DSS differential methylation requiring DMRs ≥100 bp, ≥15 CpGs, ≥50% significantly differentially methylated CpGs, mergeable within 100 bp, at p<1e-5. Judge success by concordance statistics of predicted imprinted regions against the curated ICR expectations across the trio members.

## Related tools

- **DeepMod2** (Generates phased per-site methylation output (per_site and per_site.aggregated with mod_percentage_phase1/phase2) used for ICR epihaplotype classification) — https://github.com/WGLab/DeepMod2
- **NanoCaller** (Phases reads into parental haplotypes (--mode snps --phase) to enable phase-resolved methylation calls)
- **bedtools** (Intersects phased per-site methylation BED files with the curated ICR coordinate BED)
- **DSS** (Detects differentially methylated regions (≥100 bp, ≥15 CpGs, ≥50% significant CpGs, p<1e-5) supporting imprinted-region calls)

## Examples

```
bedtools intersect -a humanicr_icrs.bed -b deepmod2.output.per_site.bed -wa -u
```

## Evaluation signals

- Expected ICR recovery: 19, 20, and 19 of 21 known imprinting control regions detected in HG002, HG003, and HG004, respectively
- A correctly classified ICR shows one phase completely unmodified and the other nearly completely modified (e.g., chr11:2699000-2702000 with mod_percentage_phase1=0 and mod_percentage_phase2 approaching 1)
- Parent-of-origin labels are consistent across the trio members (child's phase assignment matches the phased methylation pattern in the corresponding parent)
- Supporting DSS DMRs satisfy all three criteria: ≥100 bp length, ≥15 CpG sites, and ≥50% of CpGs differentially methylated at p<1e-5
- The analyzed ICR set matches the curated catalog from Jima et al. Supplementary Table S1 (humanicr.org)

## Limitations

- Read phasing may not be possible in regions with runs of homozygosity, a drawback of depending solely upon phased methylation counts for analyzing imprinted regions
- DeepMod2 provides models only for 5mCpG methylation due to lack of training data for other modifications; 5hmC is not detected
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Current differential methylation tools use binary methylated/unmethylated counts and ignore per-read methylation probabilities and long-read co-occurrence

## Evidence

- [intro] we demonstrate that phased methylation calls from DeepMod2 can be used to accurately predict imprinted regions in human cell-lines: "we demonstrate that phased methylation calls from DeepMod2 can be used to accurately predict imprinted regions in human cell-lines"
- [results] obtained the list of all candidate ICRs from Supplementary Table S1 of Jima et al.36, which can also be found at https://humanicr.org/: "obtained the list of all candidate ICRs from Supplementary Table S1 of Jima et al.36, which can also be found at https://humanicr.org/"
- [results] (1) to be at least 100 bp long, (2) to contain at least 15 CpG sites, (3) at least 50% of CpGs in the region should be differentially methylated in statistically significant manner: "(1) to be at least 100 bp long, (2) to contain at least 15 CpG sites, (3) at least 50% of CpGs in the region should be differentially methylated in statistically significant manner"
- [results] read phasing may not be possible in regions with runs of homozygosity: "This illustrates a drawback in depending solely upon phased methylation counts for analyzing imprinted regions, because read phasing may not be possible in regions with runs of homozygosity"
- [readme] DeepMod2 can call methylation from POD5 and FAST5 files basecalled with Guppy or Dorado and provides models for R10.4.1 and R9.4.1 flowcells.: "DeepMod2 can call methylation from POD5 and FAST5 files basecalled with Guppy or Dorado and provides models for R10.4.1 and R9.4.1 flowcells."
