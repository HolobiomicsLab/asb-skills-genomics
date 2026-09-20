---
name: genomic-coordinate-signature-tabulation
description: Use when you have contig-to-reference alignments (BAM) from a diploid
  assembly (e.g., VolcanoSV-asm output on HiFi/CLR/ONT reads) and need to tabulate
  large INS/DEL SV signatures for downstream variant calling.
when_to_use_negative:
- The target variants are small indels or SNPs — VolcanoSV-vc provides separate Small_INDEL
  and SNP modules rather than this large-indel signature approach
- No diploid assembly contigs are available and only read-to-reference alignments
  exist — assembly-based signature extraction is inapplicable
- The sample is homozygous (e.g., CHM13), since VolcanoSV-asm phasing-based assembly
  is unsuitable for such samples
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: VolcanoSV
  role: Provides the VolcanoSV-vc Large_INDEL module (volcanosv-vc-large-indel.py)
    that computes Diff_dis/Diff_olp signatures from assembly alignments and outputs
    a phased VCF
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: pysam
  role: Used to parse the contig-to-reference BAM and extract primary and supplementary
    (split) alignment blocks per contig
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:7fc8440a578c94a1c1e3b70d068815de8a3285b02737240b1bb54ca317c22893
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  - pysam
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Genomic-coordinate signature tabulation for large indel SVs

## Summary

Extracts large insertion/deletion SV signatures (chromosome, start, svlen) from contig-to-reference alignments of the VolcanoSV diploid assembly, using consecutive-block reference-distance (Diff_dis, TH_dis=30 bp) and overlap (Diff_olp, TH_olp=3000 bp) metrics, and writes them to a signatures.tsv consumed by VolcanoSV-vc to emit a phased VCF.

## When to use

Use when you have contig-to-reference alignments (BAM) from a diploid assembly (e.g., VolcanoSV-asm output on HiFi/CLR/ONT reads) and need to tabulate large INS/DEL SV signatures for downstream variant calling.

## When NOT to use

- The target variants are small indels or SNPs — VolcanoSV-vc provides separate Small_INDEL and SNP modules rather than this large-indel signature approach
- No diploid assembly contigs are available and only read-to-reference alignments exist — assembly-based signature extraction is inapplicable
- The sample is homozygous (e.g., CHM13), since VolcanoSV-asm phasing-based assembly is unsuitable for such samples

## Inputs

- Contig-to-reference alignment BAM from diploid assembly (e.g., Hifi_L2_hg19_minimap2_chr10.bam)
- Reference FASTA with index (genome.fa / genome.fa.fai)
- Thresholds TH_dis=30 bp (deletion-distance) and TH_olp=3000 bp (overlap)

## Outputs

- signatures.tsv with columns chromosome, start, svlen (svlen<0 for DEL, svlen>0 for INS)
- Downstream: phased VCF from the VolcanoSV-vc large indel module

## How to apply

Parse the contig-to-reference BAM (e.g., Hifi_L2_hg19_minimap2_chr10.bam from the Zenodo test data) with pysam, extracting all primary and supplementary (split) alignment blocks per contig. For each pair of consecutive alignment blocks on the same contig, compute the reference-distance metric Diff_dis and the overlap metric Diff_olp. Classify the signature as a large INS when Diff_dis exceeds the deletion-distance threshold (TH_dis=30 bp) with non-overlapping blocks, and as a large DEL when Diff_olp exceeds the overlap threshold (TH_olp=3000 bp); derive svlen from the block geometry (svlen<0 for DEL, svlen>0 for INS). Write all classified signatures to signatures.tsv with columns chromosome, start, svlen; this table feeds the VolcanoSV-vc large indel module, whose output is ultimately a phased VCF. Validation hinges on svlen sign consistency with the INS/DEL classification under the 30 bp/3000 bp thresholds.

## Related tools

- **VolcanoSV** (Provides the VolcanoSV-vc Large_INDEL module (volcanosv-vc-large-indel.py) that computes Diff_dis/Diff_olp signatures from assembly alignments and outputs a phased VCF) — https://github.com/maiziezhoulab/VolcanoSV
- **pysam** (Used to parse the contig-to-reference BAM and extract primary and supplementary (split) alignment blocks per contig)

## Examples

```
import pysam; bam = pysam.AlignmentFile('Hifi_L2_hg19_minimap2_chr10.bam','rb'); blocks = [a for a in bam.fetch(until_eof=True) if not a.is_secondary]
```

## Evaluation signals

- signatures.tsv exists and every row contains the chromosome, start, and svlen fields
- signatures.tsv has a non-zero row count on test data such as Hifi_L2_hg19_minimap2_chr10.bam
- Every svlen sign is consistent with its INS/DEL classification (svlen>0 for INS via Diff_dis>30 bp with non-overlapping blocks; svlen<0 for DEL via Diff_olp>3000 bp)
- Final phased VCF from VolcanoSV-vc large indel detection reproduces the expected result on the Zenodo test dataset before applying to user data

## Limitations

- VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- As assembly algorithms follow simplest paths through overlapping reads, complex SVs may be missed by assembly-based components
- Chromosome/contig naming must be consistent across reference and BAM; a --chrom_list file is required in WGS mode listing names exactly as they appear in the reference

## Evidence

- [methods] workflow card: block-pair metrics: [evidence span withheld — anchor status: not_found]
- [methods] workflow card: classification thresholds: [evidence span withheld — anchor status: not_found]
- [readme] README: VolcanoSV pipeline overview: "VolcanoSV is an innovative hybrid SV detection pipeline that utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly."
- [readme] README: large indel module location: "Single chromosome mode Large Indel detection"
- [intro] article intro: low FDR of VolcanoSV-vc: "VolcanoSV-vc, the assembly-based SV calling component, has a low false discovery rate"
