---
name: clipped-read-assembly-error-detection
description: Use when a draft de novo assembly (e.g. Falcon contigs) may contain misjoined/chimeric
  contigs and the original Illumina short-read and PacBio long-read data are available
  to remap for clipped-alignment error detection before Hi-C or optical-map scaffolding.
when_to_use_negative:
- The genome is polyploid or of higher ploidy — the article states evaluation of higher-ploidy
  assemblies 'may not be as accurate as the benchmarked cases presented here'
- No original NGS/SMS sequencing reads are available for the assembly — CRAQ is reference-free
  but requires remapping raw reads to the draft contigs
- The input is already a finished/reference-quality assembly (AQI > 90) with no suspected
  structural misjoins, or the target is only small-scale local error repair (CREs)
  rather than split-then-rescaffold structural correction
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: CRAQ
  role: Extracts clipped (CIGAR S/H) alignments from filtered BAMs, classifies CREs/CSEs
    vs heterozygous loci using clipping-ratio cutoffs f and h, and splits chimeric
    contigs at CSE breakpoints with --break|-b T, reporting R-AQI/S-AQI
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps raw Illumina reads with '-ax sr' and PacBio long reads with 'map-pb'/'map-hifi'
    onto the draft contigs to generate the clipping signal CRAQ consumes
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters read alignments before CRAQ using 'samtools view -F 1796 -q 20' (MAPQ
    <20, unmapped, secondary, QC-failed, PCR-duplicate reads removed)
- name: Integrative Genomics Viewer (IGV)
  role: Visually inspects filtered alignments around candidate CSE/CRE breakpoints
    (e.g. the contig8 misjoin)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:40819f29b7a7bc62c7dca144abb57c7267974f9d4692555cb8702d7440a174e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Integrative Genomics Viewer (IGV)
  - Minimap2
  - SAMtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# clipped-read-assembly-error-detection

## Summary

Detect large-scale misjoins (CSEs) in a draft genome assembly at single-nucleotide resolution by mining clipped (CIGAR S/H) read alignments from NGS and SMS read remapping with CRAQ, then split chimeric contigs at CSE breakpoints so optical-map or Hi-C scaffolding can fix the errors. Demonstrated on A. oxysepala Falcon contigs, where CRAQ-corrected scaffolds gained S-AQI 58.1 vs 39.0 and N50 28 Mb vs 20 Mb.

## When to use

Use when a draft de novo assembly (e.g. Falcon contigs) may contain misjoined/chimeric contigs and the original Illumina short-read and PacBio long-read data are available to remap for clipped-alignment error detection before Hi-C or optical-map scaffolding.

## When NOT to use

- The genome is polyploid or of higher ploidy — the article states evaluation of higher-ploidy assemblies 'may not be as accurate as the benchmarked cases presented here'
- No original NGS/SMS sequencing reads are available for the assembly — CRAQ is reference-free but requires remapping raw reads to the draft contigs
- The input is already a finished/reference-quality assembly (AQI > 90) with no suspected structural misjoins, or the target is only small-scale local error repair (CREs) rather than split-then-rescaffold structural correction

## Inputs

- Draft de novo contig assembly in FASTA (e.g. A.oxysepala_contig_draft.fa)
- Illumina short-read sequencing data (paired-end FASTQ) from the same sample
- Single-molecule sequencing (SMS) long reads, e.g. PacBio CLR or HiFi FASTQ/BAM
- Bionano DLE-1 optical consensus maps (CMAPs) for validation (optional)
- Hi-C FASTQ reads (MboI) for post-correction rescaffolding (optional)

## Outputs

- CRAQ-corrected contig FASTA (out_correct.fa / A.oxysepala_contig_CRAQcorrect.fa) with contigs split at CSE breakpoints
- Structural error breakpoint BED file (strER_out/out_final.CSE.bed), e.g. contig8:1,874,290
- CRAQ quality report (out_final.Report) with normalized CRE/CSE counts and R-AQI/S-AQI scores
- Rescaffolded Hi-C assemblies and comparison metrics table (e.g. craq_correction_metrics.tsv) with N50, R-AQI, S-AQI per version
- Optical-map validation record (ctg8_1 → CMAP-1, ctg8_2 → CMAP-10) and conflict counts before/after correction

## How to apply

Map raw Illumina reads to the draft contigs with Minimap2 2.17+ '-ax sr' and PacBio long reads with 'map-hifi' (or 'map-pb'), then filter alignments with 'samtools view -F 1796 -q 20' to drop MAPQ<20, unmapped, secondary, QC-failed, and PCR-duplicate reads. Run CRAQ on the assembly (-g) with the SMS and NGS BAMs, enabling '--break|-b T' so CSEs are called at clipping-ratio cutoff f=0.75 (loci with clipping ratio inside the heterozygous band h=0.4-0.6 are classified as CRHs/CSHs, not errors; candidates below m=2 clipped reads or above M=5x average coverage are excluded) and chimeric fragments are split at CSE breakpoints, writing out_correct.fa and strER_out/out_final.CSE.bed. For A. oxysepala expect ~117 CSEs, including the breakpoint at contig8:1,874,290 that splits ctg8 into ctg8_1 and ctg8_2. Validate the split by aligning corrected contigs to Bionano DLE-1 optical consensus maps with RefAligner (ctg8_1 should map to CMAP-1 and ctg8_2 to CMAP-10) and count optical-map conflicts before versus after correction. Scaffold both original and corrected contigs with Hi-C via Juicer ('-s MboI') and 3D-DNA ('-m haploid -r 0', no internal error correction), then compare N50, R-AQI, and S-AQI from out_final.Report: corrected scaffolds should reach approximately N50 28 Mb, R-AQI 78.7, S-AQI 58.1 versus original N50 20 Mb, R-AQI 79.1, S-AQI 39.0 — the structural gain with near-unchanged regional quality is the signature of a successful misjoin correction.

## Related tools

- **CRAQ** (Extracts clipped (CIGAR S/H) alignments from filtered BAMs, classifies CREs/CSEs vs heterozygous loci using clipping-ratio cutoffs f and h, and splits chimeric contigs at CSE breakpoints with --break|-b T, reporting R-AQI/S-AQI) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps raw Illumina reads with '-ax sr' and PacBio long reads with 'map-pb'/'map-hifi' onto the draft contigs to generate the clipping signal CRAQ consumes) — https://github.com/lh3/minimap2
- **SAMtools** (Filters read alignments before CRAQ using 'samtools view -F 1796 -q 20' (MAPQ <20, unmapped, secondary, QC-failed, PCR-duplicate reads removed))
- **Integrative Genomics Viewer (IGV)** (Visually inspects filtered alignments around candidate CSE/CRE breakpoints (e.g. the contig8 misjoin))

## Examples

```
minimap2 -ax map-pb A.oxysepala_contig_draft.fa pacbio.fq.gz | samtools view -F 1796 -q 20 -b -o sms.filtered.bam
```

## Evaluation signals

- CSE count is in the vicinity of the reported 117 for A. oxysepala draft contigs, including breakpoint contig8:1,874,290
- RefAligner/Bionano Solve validation confirms ctg8_1 maps to CMAP-1 and ctg8_2 to CMAP-10, and optical-map conflicts decrease after correction
- Corrected-scaffold metrics reproduce approximately N50 = 28 Mb, R-AQI = 78.7, S-AQI = 58.1 versus original N50 = 20 Mb, R-AQI = 79.1, S-AQI = 39.0 (structural gain, regional quality nearly unchanged)
- CRAQ output files out_correct.fa and strER_out/out_final.CSE.bed exist and split positions correspond to high clipping-ratio (≥ f = 0.75) loci, while loci with clipping ratio within h = 0.4–0.6 were not split
- Filtered BAMs were produced with '-F 1796 -q 20'; unfiltered mappings should be rejected as CRAQ input

## Limitations

- CRAQ accuracy for polyploid genomes is not established — 'evaluation of genome assemblies for species with higher ploidy levels may not be as accurate as the benchmarked cases presented here'
- Regions with coverage above the upper level of h but below f are reported as ambiguous heterozygous-or-error regions without definitive classification
- The CSE/CRE coverage-discrepancy cutoff (-d) is user-tunable with no stated universal value, and observed error-breakpoint ratios can fall below 100% due to sequencing errors or inaccurate read mapping
- Optical-map validation only infers approximate misjoin locations, not exact breakpoints
- Read mapping is the most resource-intensive step; users must manually split query sequences to parallelize

## Evidence

- [abstract] CRAQ detects potential chimeric contigs and break them at conflict breakpoints for assembly correction: "CRAQ detects potential chimeric contigs and break them at conflict breakpoints for assembly correction"
- [intro] CRAQ offers the option to correct conflicting contigs by breaking them at relevant error breakpoints; optical maps or Hi-C can then be integrated to fix such errors and improve the assembly.: "CRAQ offers the option to correct conflicting contigs by breaking them at relevant error breakpoints; optical maps or Hi-C can then be integrated to fix such errors and improve the assembly."
- [discussion] clipping-ratio cutoff f: "A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75)"
- [discussion] filtered alignments: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’35"
- [readme] README CRAQcorrect contigs: [evidence span withheld — anchor status: external_doc]
