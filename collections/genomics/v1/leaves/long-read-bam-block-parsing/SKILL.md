---
name: long-read-bam-block-parsing
description: Use when you have a BAM of assembled contigs aligned to a reference (e.g.
  minimap2 alignments of VolcanoSV diploid assembly output) and need to extract large
  INS/DEL SV signatures from split-alignment geometry.
when_to_use_negative:
- When the goal is calling SVs directly from reads rather than from a diploid assembly
  — VolcanoSV-vc processes assembly output, and single-chromosome mode cannot support
  translocation detection since that requires a WGS BAM
- For small indel or SNP detection, which are handled by separate VolcanoSV-vc modules
  (Small_INDEL and SNP detection) with different signatures
- For sex chromosomes — VolcanoSV is designed only for autosome chromosomes because
  the phasing procedure for sex chromosomes is very different
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0102
tools:
- name: VolcanoSV
  role: source pipeline whose VolcanoSV-vc Large_INDEL module consumes the extracted
    block signatures to call large indels and produce a phased VCF
  repo: https://github.com/maiziezhoulab/VolcanoSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:a40158edb0beef54588220e303503cb009e608332d35ebeaadc44b8dcb5b1c96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# long-read-bam-block-parsing

## Summary

Parses a contig-to-reference long-read alignment BAM into primary and supplementary (split) alignment blocks with pysam, and computes the reference-distance (Diff_dis) and overlap (Diff_olp) metrics between consecutive blocks on each contig to surface large indel SV signatures for VolcanoSV-vc.

## When to use

Use when you have a BAM of assembled contigs aligned to a reference (e.g. minimap2 alignments of VolcanoSV diploid assembly output) and need to extract large INS/DEL SV signatures from split-alignment geometry.

## When NOT to use

- When the goal is calling SVs directly from reads rather than from a diploid assembly — VolcanoSV-vc processes assembly output, and single-chromosome mode cannot support translocation detection since that requires a WGS BAM
- For small indel or SNP detection, which are handled by separate VolcanoSV-vc modules (Small_INDEL and SNP detection) with different signatures
- For sex chromosomes — VolcanoSV is designed only for autosome chromosomes because the phasing procedure for sex chromosomes is very different

## Inputs

- contig-to-reference alignment BAM (e.g. contigs aligned with minimap2 to hg19, such as Hifi_L2_hg19_minimap2_chr10.bam)
- reference FASTA index (genome.fa / genome.fa.fai) for contig/chromosome naming consistency

## Outputs

- signatures.tsv with columns chromosome, start, svlen (svlen<0 for DEL, svlen>0 for INS)

## How to apply

Parse the contig-to-reference BAM (e.g. Hifi_L2_hg19_minimap2_chr10.bam from the Zenodo test data) with pysam, extracting all primary and supplementary (split) alignment blocks per contig. For each pair of consecutive alignment blocks on the same contig, compute the reference-distance metric Diff_dis and the overlap metric Diff_olp. Classify the pair as a large INS when Diff_dis exceeds the deletion-distance threshold (TH_dis=30 bp) with non-overlapping blocks, and as a large DEL when Diff_olp exceeds the overlap threshold (TH_olp=3000 bp); record the reference chromosome and start of each signature and derive svlen from the block geometry (svlen<0 for DEL, svlen>0 for INS). Write all classified signatures to signatures.tsv with the columns chromosome, start, svlen for downstream VolcanoSV-vc large-indel variant calling, which ultimately contributes to a phased VCF.

## Related tools

- **VolcanoSV** (source pipeline whose VolcanoSV-vc Large_INDEL module consumes the extracted block signatures to call large indels and produce a phased VCF) — https://github.com/maiziezhoulab/VolcanoSV

## Examples

```
import pysam; bam = pysam.AlignmentFile('Hifi_L2_hg19_minimap2_chr10.bam','rb'); blocks=[a for a in bam.fetch() if not a.is_secondary and not a.is_supplementary or a.is_supplementary]
```

## Evaluation signals

- signatures.tsv exists and contains the chromosome, start, and svlen fields on every row with a non-zero row count
- Every svlen sign is consistent with its INS/DEL classification under the TH_dis=30 bp and TH_olp=3000 bp thresholds
- All primary and supplementary alignment blocks per contig were extracted (no dropped split alignments) before Diff_dis/Diff_olp computation
- Downstream VolcanoSV-vc large-indel run on the signatures completes and emits a phased VCF
- Chromosome names in signatures.tsv exactly match reference/BAM contig naming (e.g. chr10)

## Limitations

- VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- Chromosome/contig naming must be used consistently across the reference and BAM files; WGS mode requires an explicit --chrom_list text file listing contig names exactly as they appear in the reference
- The large-indel signature scan alone is insufficient for complex SVs (TRA/INV/DUP) — those require WGS BAM support and separate detection modules

## Evidence

- [other] Parse the contig-to-reference BAM (e.g. Hifi_L2_hg19_minimap2_chr10.bam from the Zenodo test data) with pysam, extracting all primary and supplementary (split) alignment blocks per contig.: [evidence span withheld — anchor status: not_found]
- [other] Classification uses Diff_dis > TH_dis=30 bp for INS and Diff_olp > TH_olp=3000 bp for DEL.: [evidence span withheld — anchor status: not_found]
- [other] VolcanoSV-vc contains a dedicated large indel SV detection module whose output is a phased VCF file.: [evidence span withheld — anchor status: not_found]
- [readme] README documents the Large_INDEL module path and Zenodo test data for single-chromosome mode.: "For the single chromosome mode, we provided the chr10 BAM file, contigs file and VCF file for Hifi, CLR and ONT data. You can download them from [zenodo]"
- [readme] The script lives at bin/VolcanoSV-vc/Large_INDEL/volcanosv-vc-large-indel.py per the installation notes.: "bin/VolcanoSV-vc/Large_INDEL/volcanosv-vc-large-indel.py"
- [abstract] The assembly-based component processes diploid assembly output for variant calling and produces phased SV calls.: "an innovative hybrid SV detection pipeline that utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly"
