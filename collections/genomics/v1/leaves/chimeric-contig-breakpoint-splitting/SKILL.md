---
name: chimeric-contig-breakpoint-splitting
description: Use when a de novo draft assembly (e.g. Falcon/Canu contigs) may contain
  misjoins, and raw Illumina short reads plus PacBio/ONT long reads are available
  to detect CSE breakpoints before scaffolding with Hi-C or optical maps.
when_to_use_negative:
- Candidate breakpoints fall in heterozygous loci (clipping ratio within h = 0.4-0.6)
  — these are haplotype differences, not misjoins, and must not be split
- The assembly is already reference quality (AQI > 90) with no optical-map or read-mapping
  conflict signal
- The genome is polyploid — the paper states CRAQ evaluation accuracy 'may not be
  as accurate as the benchmarked cases' for higher ploidy levels
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: CRAQ
  role: Detects CSE breakpoints from clipped NGS/SMS alignments and splits chimeric
    contigs when run with --break|-b T
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps short ('-ax sr') and long reads ('map-pb'/'map-hifi'/'map-ont') back
    to the draft assembly to generate clipping signals
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: 'Filters alignments before CRAQ (''samtools view -F 1796 -q 20'': MAPQ >=
    20, no unmapped/secondary/QC-fail/duplicate reads)'
- name: Integrative Genomics Viewer (IGV)
  role: Visual inspection of clipped-read clusters at candidate CSE breakpoints
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:e99fc67ffb43464c0ecb236341fbcd518114365537f431785243b1a599057f2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Integrative Genomics Viewer (IGV)
  - Minimap2
  - SAMtools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# chimeric-contig-breakpoint-splitting

## Summary

Detects chimeric misjoined contigs by mapping NGS and SMS reads back to a draft assembly with CRAQ, then splits contigs at Clip-based Structural Error (CSE) breakpoints so downstream scaffolding (Bionano optical maps, Hi-C/3D-DNA) joins the correct pieces. Splitting before rescaffolding raised the A. oxysepala S-AQI from 39.0 to 58.1 and N50 from 20 Mb to 28 Mb.

## When to use

Use when a de novo draft assembly (e.g. Falcon/Canu contigs) may contain misjoins, and raw Illumina short reads plus PacBio/ONT long reads are available to detect CSE breakpoints before scaffolding with Hi-C or optical maps.

## When NOT to use

- Candidate breakpoints fall in heterozygous loci (clipping ratio within h = 0.4-0.6) — these are haplotype differences, not misjoins, and must not be split
- The assembly is already reference quality (AQI > 90) with no optical-map or read-mapping conflict signal
- The genome is polyploid — the paper states CRAQ evaluation accuracy 'may not be as accurate as the benchmarked cases' for higher ploidy levels

## Inputs

- Draft genome assembly FASTA (e.g. A.oxysepala_contig_draft.fa)
- Raw Illumina NGS paired-end reads (FASTQ)
- SMS long reads (PacBio CLR/HiFi or ONT, FASTQ)
- Optional: Bionano DLE-1 optical consensus maps (CMAPs) for validation
- Optional: Hi-C reads and enzyme site files for post-split rescaffolding

## Outputs

- CRAQ-corrected split contigs (out_correct.fa, e.g. A.oxysepala_contig_CRAQcorrect.fa)
- CSE breakpoint coordinates (strER_out/out_final.CSE.bed; ~117 CSEs for A. oxysepala including contig8:1,874,290)
- CRAQ report (out_final.Report) with CRE/CSE counts, R-AQI and S-AQI
- Rescaffolded Hi-C assemblies of original vs corrected contigs (e.g. A.oxysepala_corrected.scaffold.hic.fas)
- Comparison table of N50 / R-AQI / S-AQI (craq_correction_metrics.tsv) and optical-map conflict counts before vs after correction

## How to apply

Map Illumina short reads with minimap2 '-ax sr' and PacBio/ONT long reads with 'map-pb'/'map-ont' (or 'map-hifi') to the draft FASTA, then filter with 'samtools view -F 1796 -q 20' to drop MAPQ<20, unmapped, secondary, QC-failed, and duplicate alignments. Run CRAQ with the assembly (-g) and the two filtered BAMs, enabling '--break|-b T' so chimeric fragments are split at CSE breakpoints (clipping-ratio cutoff f = 0.75, heterozygous band h = 0.4-0.6, coverage window m = 2 to M = 5x average coverage), producing out_correct.fa and strER_out/out_final.CSE.bed. Distinguish true errors from heterozygous loci: loci with clipping ratios within h are heterozygous, only ratios exceeding f are breakpoints. Validate splits independently — for A. oxysepala the ctg8:1,874,290 breakpoint yields ctg8_1 aligning to Bionano DLE-1 CMAP-1 and ctg8_2 to CMAP-10 via RefAligner — then rescaffold corrected and original contigs identically (Juicer '-s MboI' + 3D-DNA '-m haploid -r 0') and compare N50, R-AQI, and S-AQI.

## Related tools

- **CRAQ** (Detects CSE breakpoints from clipped NGS/SMS alignments and splits chimeric contigs when run with --break|-b T) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps short ('-ax sr') and long reads ('map-pb'/'map-hifi'/'map-ont') back to the draft assembly to generate clipping signals) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments before CRAQ ('samtools view -F 1796 -q 20': MAPQ >= 20, no unmapped/secondary/QC-fail/duplicate reads))
- **Integrative Genomics Viewer (IGV)** (Visual inspection of clipped-read clusters at candidate CSE breakpoints)

## Examples

```
minimap2 -ax sr A.oxysepala_contig_draft.fa reads_R1.fq.gz reads_R2.fq.gz | samtools view -bF 1796 -q 20 -o ngs.bam -
```

## Evaluation signals

- CSE count lands near the reported 117 for A. oxysepala Falcon contigs, including the breakpoint at contig8:1,874,290
- RefAligner/Bionano Solve validation: ctg8_1 maps to CMAP-1 and ctg8_2 maps to CMAP-10, with fewer optical-map conflicts after correction than before
- Corrected Hi-C scaffolds reproduce ~N50 = 28 Mb, R-AQI = 78.7, S-AQI = 58.1 vs original N50 = 20 Mb, R-AQI = 79.1, S-AQI = 39.0
- R-AQI is nearly unchanged between original and corrected scaffolds (78.7 vs 79.1) — improvement should be structural (S-AQI), not regional
- Output files exist and are consistent: out_correct.fa, strER_out/out_final.CSE.bed, out_final.Report

## Limitations

- Error clipping ratios can be lower than 100% in practice due to sequencing errors or inaccurate read mapping, complicating classification
- CSE/CRE classification depends on the user-tunable coverage-discrepancy parameter '-d', for which no universal value is stated
- Regions with coverage above the upper h level and below f are reported only as ambiguous heterozygous-or-error regions
- Optical-map validation can only infer approximate misjoin locations, not exact breakpoints
- Read mapping is the most resource-intensive step; large assemblies must be split into fragments to parallelize

## Evidence

- [abstract] CRAQ detects potential chimeric contigs and break them at conflict breakpoints for assembly correction: "CRAQ detects potential chimeric contigs and break them at conflict breakpoints for assembly correction"
- [intro] split-before-scaffolding improves structural quality: "CRAQ offers the option to correct conflicting contigs by breaking them at relevant error breakpoints; optical maps or Hi-C can then be integrated"
- [discussion] breakpoint calling cutoff: "A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75)"
- [discussion] alignment filtering before CRAQ: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’35"
- [readme] corrected contigs are split at CSE positions: [evidence span withheld — anchor status: external_doc]
