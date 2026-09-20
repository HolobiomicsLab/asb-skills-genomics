---
name: split-alignment-signature-extraction
description: Use when you have contig-to-reference BAM alignments (e.g. Hifi_L2_hg19_minimap2_chr10.bam
  from the VolcanoSV Zenodo test data) and need to harvest large indel SV signatures
  into a signatures.tsv before VCF generation.
when_to_use_negative:
- The goal is complex SV calling (translocations, duplications, inversions) — those
  require the VolcanoSV-vc Complex_SV module with WGS read BAM support, not split-alignment
  signatures from a single chromosome
- The input is read-to-reference alignments rather than assembly contig-to-reference
  alignments — this signature extraction is defined on the diploid assembly output
- The target is sex chromosomes — VolcanoSV is designed only for autosome chromosomes
  because the phasing procedure for sex chromosomes differs
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0622
tools:
- name: VolcanoSV
  role: Hosts the Large INDEL detection module (VolcanoSV-vc) whose large-indel script
    consumes the extracted signatures to emit a phased VCF
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: pysam
  role: Python library used to parse primary and supplementary alignment blocks from
    the contig-to-reference BAM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:c4fbcf405a8fbcdad9baa99b4ab370219babd5077c56be40adcf4142590496c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  - pysam
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# split-alignment-signature-extraction

## Summary

Extracts large insertion/deletion (50 bp–several kb) structural variant signatures from contig-to-reference alignments of a diploid assembly, as implemented in the VolcanoSV-vc Large INDEL module, by parsing primary and supplementary (split) alignment blocks and classifying consecutive-block discrepancies as INS or DEL signatures.

## When to use

Use when you have contig-to-reference BAM alignments (e.g. Hifi_L2_hg19_minimap2_chr10.bam from the VolcanoSV Zenodo test data) and need to harvest large indel SV signatures into a signatures.tsv before VCF generation.

## When NOT to use

- The goal is complex SV calling (translocations, duplications, inversions) — those require the VolcanoSV-vc Complex_SV module with WGS read BAM support, not split-alignment signatures from a single chromosome
- The input is read-to-reference alignments rather than assembly contig-to-reference alignments — this signature extraction is defined on the diploid assembly output
- The target is sex chromosomes — VolcanoSV is designed only for autosome chromosomes because the phasing procedure for sex chromosomes differs

## Inputs

- contig-to-reference BAM file (e.g. Hifi_L2_hg19_minimap2_chr10.bam, alignments of diploid assembly contigs against hg19)
- reference genome FASTA and .fai index (genome.fa / genome.fa.fai from Zenodo record 10520476)
- chromosome/contig name (e.g. chr10) matching the reference and BAM naming

## Outputs

- signatures.tsv with columns chromosome, start, svlen (svlen<0 for DEL, svlen>0 for INS)
- downstream: phased VCF of large indel SVs produced by the VolcanoSV-vc large indel detection module

## How to apply

Parse the contig-to-reference BAM (produced by aligning the phased diploid assembly contigs, e.g. with minimap2 against hg19) with pysam, keeping all primary and supplementary (split) alignment blocks per contig while discarding secondary alignments. For each pair of consecutive alignment blocks on the same contig, compute the reference-distance metric Diff_dis and the overlap metric Diff_olp. Classify the pair as a large INS signature when Diff_dis exceeds TH_dis=30 bp with non-overlapping blocks, and as a large DEL signature when Diff_olp exceeds TH_olp=3000 bp; derive svlen from the block geometry (svlen<0 for DEL, svlen>0 for INS) and record the reference chromosome and start position. Write the classified signatures to a signatures.tsv with columns chromosome, start, svlen; this file is the input the large indel detection module (bin/VolcanoSV-vc/Large_INDEL/volcanosv-vc-large-indel.py) consumes downstream on its way to a phased VCF. The rationale is that the article's filtering stage requires at least one supporting signature per raw call, so robust, complete signature extraction is the foundation of low-false-discovery SV calling in VolcanoSV-vc.

## Related tools

- **VolcanoSV** (Hosts the Large INDEL detection module (VolcanoSV-vc) whose large-indel script consumes the extracted signatures to emit a phased VCF) — https://github.com/maiziezhoulab/VolcanoSV
- **pysam** (Python library used to parse primary and supplementary alignment blocks from the contig-to-reference BAM)

## Examples

```
import pysam; bam=pysam.AlignmentFile('Hifi_L2_hg19_minimap2_chr10.bam','rb'); blocks=[a for a in bam.fetch('chr10') if not a.is_secondary]
```

## Evaluation signals

- signatures.tsv exists and contains chromosome, start, and svlen fields on every row with a non-zero row count
- every svlen sign is consistent with its INS/DEL classification: svlen<0 for DEL (Diff_olp > 3000 bp) and svlen>0 for INS (Diff_dis > 30 bp with non-overlapping blocks)
- signatures fall in the large indel range (~50 bp to several kb) — nothing below the 50 bp boundary that belongs to the small indel module
- every signature's reference chromosome and start coordinate lie on the contig's aligned region in the input BAM
- downstream, the phased VCF from the Large_INDEL module should contain calls corresponding to signatures, since a raw SV call with no supporting signature is filtered out as a false call

## Limitations

- Translocation detection requires a WGS BAM file as support, so single-chromosome signature extraction cannot support TRA calls
- Thresholds TH_dis=30 bp and TH_olp=3000 bp were calibrated in the VolcanoSV-vc pipeline; reusing them on other aligners/assemblies may shift sensitivity
- Assembly-based small indel detection proved ineffective for ONT data (low precision for all tools), so large-indel signatures alone cannot cover the small indel regime

## Evidence

- [readme] VolcanoSV is an innovative hybrid SV detection pipeline that utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly.: "VolcanoSV is an innovative hybrid SV detection pipeline that utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly."
- [readme] Large INDEL module path in the VolcanoSV repository: "bin/VolcanoSV-vc/Large_INDEL/volcanosv-vc-large-indel.py"
- [discussion] Signature support requirement for raw SV calls: "A raw SV call needs at least one supporting signature, otherwise it will be filtered out as a false call"
- [intro] Low false discovery rate of the assembly-based calling component: "VolcanoSV-vc, the assembly-based SV calling component, has a low false discovery rate"
