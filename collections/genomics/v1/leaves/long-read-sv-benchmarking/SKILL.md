---
name: long-read-sv-benchmarking
description: Use when evaluating or reproducing SV caller performance on long-read
  (PacBio HiFi/CLR, ONT) datasets where a truth or high-confidence callset exists
  and recall/precision/F1/GT_F1 comparisons are the deliverable.
when_to_use_negative:
- Dipcall cannot be included in complex SV (translocation/inversion/duplication) benchmarking
  since it was not designed to detect complex SVs
- The CHM13 diploid assembly cannot be processed with the VolcanoSV-asm phasing component
  because the sample is homozygous and unsuitable for haplotype phasing
- Do not expect reliable small-indel assembly-based calling on ONT data, where precision
  was low for all three evaluated tools
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0257
tools:
- name: Truvari
  role: Benchmarks SV calls of each tool against the GIAB SV gold standard and HCC1395
    callsets with parameters p=0.5, P=0.5, r=500, O=0.01
- name: SURVIVOR
  role: Calls somatic variants from paired normal-tumor VCFs (v1.0.6) for the HCC1395
    benchmark
- name: VISOR
  role: Inserts SNPs and complex SVs (translocations, duplications, inversions) into
    the hg19 reference (v1.1.2) to create simulation truth sets
- name: PBSIM
  role: Simulates long reads for each data type (v3.0.0) from the SV-inserted reference
- name: VolcanoSV
  role: SV caller under evaluation; run per chromosome (volcanosv-asm.py, volcanosv-vc-large-indel.py,
    volcanosv-vc-complex-sv.py) in WGS mode with -cl chrom_list.txt
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: SVIM-asm
  role: Assembly-based SV comparator on simulated and real cancer complex SV data
  repo: https://github.com/eldariont/svim-asm
- name: Dipcall
  role: Assembly-based comparator restricted to indel/SNP benchmarking; excluded from
    complex SV analysis
  repo: https://github.com/lh3/dipcall
- name: PAV
  role: Assembly-based comparator in the CHM13 false discovery rate analysis
  repo: https://github.com/EichlerLab/pav
- name: rasusa
  role: Subsamples datasets to 40x, 30x, 20x, 10x, and 5x coverage (v0.6.0) for low-coverage
    robustness benchmarking
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:9b9022036fedf7c1614e773fafc87169e5cea8031adc206e1ca9d9337bfec72c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - DeBreak
  - Dipcall
  - MAMnet
  - NanoVar
  - PAV
  - PBSIM
  - SURVIVOR
  - SVDSS
  - SVIM-asm
  - Sniffles2
  - Truvari
  - VISOR
  - VolcanoSV
  - cuteSV
  - pbsv
  - rasusa
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0257
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# long-read-sv-benchmarking

## Summary

Benchmark structural variant calls from long-read SV callers (VolcanoSV, Sniffles2, cuteSV, pbsv, NanoVar, MAMnet, SVDSS, DeBreak, and assembly-based SVIM-asm/PAV/Dipcall) against truth sets such as the GIAB SV gold standard or HCC1395 high-confidence callsets using Truvari (v4.0.0), computing recall, precision, F1 and genotyping accuracy.

## When to use

Use when evaluating or reproducing SV caller performance on long-read (PacBio HiFi/CLR, ONT) datasets where a truth or high-confidence callset exists and recall/precision/F1/GT_F1 comparisons are the deliverable.

## When NOT to use

- Dipcall cannot be included in complex SV (translocation/inversion/duplication) benchmarking since it was not designed to detect complex SVs
- The CHM13 diploid assembly cannot be processed with the VolcanoSV-asm phasing component because the sample is homozygous and unsuitable for haplotype phasing
- Do not expect reliable small-indel assembly-based calling on ONT data, where precision was low for all three evaluated tools

## Inputs

- Long-read alignment BAMs (PacBio HiFi, PacBio CLR, ONT; e.g., HG002 Hifi_L1–L5 at 28–56x, HCC1395 tumor-normal CLR/ONT libraries)
- Truth or high-confidence SV callsets (GIAB HG002 SV gold standard, HCC1395 high-confidence callset)
- SV caller output VCFs (VolcanoSV, NanoVar, cuteSV, pbsv, Sniffles2, MAMnet, SVDSS, DeBreak, SVIM-asm, PAV, Dipcall)
- Simulated data inputs: VISOR-annotated hg19 reference and PBSIM-simulated reads
- T2T-CHM13 annotation BED tracks (chm13v2.0_censat_v2.0.bed, chm13v2.0_telomere.bed)

## Outputs

- Per-tool benchmark table of recall, precision, F1, and GT_F1 (e.g., complex_sv_performance.csv)
- False discovery rate comparison table with centromere/telomere overlap counts (e.g., false_discovery_rate_table.tsv)
- Truvari evaluation reports against the gold standard
- Somatic VCFs merged from paired normal-tumor calls via SURVIVOR

## How to apply

Run each SV caller on the same long-read BAMs (e.g., HG002 Hifi_L1–L5, CLR_L1–L3, ONT_L1–L6), then benchmark all VCFs with Truvari (v4.0.0) against the truth set using p=0.5, P=0.5, r=500, O=0.01. For somatic SVs on paired tumor-normal data (HCC1395 PacBio CLR and ONT), run each tool per library and call somatic variants with SURVIVOR (v1.0.6) from the paired normal-tumor VCFs, then report recall only since the high-confidence callset is incomplete. For simulated complex SVs (inserted with VISOR v1.1.2, reads simulated with PBSIM v3.0.0), exclude Dipcall because it was not designed to detect complex SVs, and compute F1 and GT_F1 for inversions and duplications but only recall for translocations (genotyping reciprocal TRAs is ambiguous). For assembly-based false-discovery analysis on CHM13, compute FDR as the ratio of SVs detected against T2T-CHM13 to SVs against GRCh38, intersecting false discoveries with chm13v2.0_censat_v2.0.bed and chm13v2.0_telomere.bed. Aggregate per-tool metrics (recall, precision, F1, GT_F1) into a comparison table such as complex_sv_performance.csv or false_discovery_rate_table.tsv.

## Related tools

- **Truvari** (Benchmarks SV calls of each tool against the GIAB SV gold standard and HCC1395 callsets with parameters p=0.5, P=0.5, r=500, O=0.01)
- **SURVIVOR** (Calls somatic variants from paired normal-tumor VCFs (v1.0.6) for the HCC1395 benchmark)
- **VISOR** (Inserts SNPs and complex SVs (translocations, duplications, inversions) into the hg19 reference (v1.1.2) to create simulation truth sets)
- **PBSIM** (Simulates long reads for each data type (v3.0.0) from the SV-inserted reference)
- **VolcanoSV** (SV caller under evaluation; run per chromosome (volcanosv-asm.py, volcanosv-vc-large-indel.py, volcanosv-vc-complex-sv.py) in WGS mode with -cl chrom_list.txt) — https://github.com/maiziezhoulab/VolcanoSV
- **SVIM-asm** (Assembly-based SV comparator on simulated and real cancer complex SV data) — https://github.com/eldariont/svim-asm
- **Dipcall** (Assembly-based comparator restricted to indel/SNP benchmarking; excluded from complex SV analysis) — https://github.com/lh3/dipcall
- **PAV** (Assembly-based comparator in the CHM13 false discovery rate analysis) — https://github.com/EichlerLab/pav
- **rasusa** (Subsamples datasets to 40x, 30x, 20x, 10x, and 5x coverage (v0.6.0) for low-coverage robustness benchmarking)

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-vc/Large_INDEL/volcanosv-vc-large-indel.py -bam sample.bam -ref ref.fa -cl chrom_list.txt -o outdir  # then Truvari bench: truvari bench -b truth.vcf.gz -c caller.vcf.gz -p 0.5 -P 0.5 -r 500 -O 0.01
```

## Evaluation signals

- Reproduced metrics match the reported finding that VolcanoSV outperformed SVIM-asm in recall across all libraries and complex SV types, with the largest advantage on duplications (e.g., 12% higher on PacBio CLR, 17% higher on ONT)
- VolcanoSV achieves the lowest FDR among PAV, SVIM-asm, and Dipcall in the CHM13 false-discovery analysis
- Expected performance ranges: VolcanoSV average F1 of 91.03% (insertions) and 94.19% (deletions) with genotyping accuracy of 98.32% and 99.01% across Hifi datasets
- Confirm the Truvari run completed with the specified parameter set (p=0.5, P=0.5, r=500, S=30, O=0.01)
- Only recall is reported for HCC1395 somatic SVs and only F1/GT_F1 for INVs and DUPs, matching the ambiguity of genotyping reciprocal TRAs

## Limitations

- The GIAB gold standard SV callset is not a complete set and could contain false positives
- The HCC1395 high-confidence benchmark callset included SV calls only from alignment-based tools, potentially biasing evaluation against assembly-based callers
- VolcanoSV's recall for real complex somatic SVs was relatively lower than on simulated data or indel SVs, and it failed to detect a sufficient number of unique somatic complex SVs in the tumor sample beyond germline SVs

## Evidence

- [results] Truvari parameters and metric scope: "An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard"
- [results] Truvari parameter setting: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Somatic variant calling with SURVIVOR: "then used SURVIVOR (v1.0.6)36 to call somatic variants based on the paired normal-tumor VCFs"
- [results] Dipcall exclusion for complex SVs: "Dipcall was not involved in this analysis since it was not designed to detect complex SVs"
- [results] FDR definition: "The false discovery rate was computed as the ratio of the number of detected SVs against CHM13 to the number of SVs against GRCh38"
- [results] Incomplete truth set caveat: "the GIAB gold standard SV callset is not a complete set and could also contain false positives"
- [results] Subsampling coverage levels: "All three datasets were subsampled to 40x, 30x, 20x, 10x, and 5x coverage using rasusa (v0.6.0)37"
